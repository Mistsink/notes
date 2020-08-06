# axios实例生成

在lib文件夹下，utils负责公用函数，可以说是小工具；defauls是导出一个对象，这里是一堆默认配置。主要集成在axios中：

```js
// Create the default instance to be exported
var axios = createInstance(defaults);

// createInstance 
function createInstance(defaultConfig) {
  var context = new Axios(defaultConfig);
    // 创建一个基本实例--上下文
  var instance = bind(Axios.prototype.request, context);
    // 这里的bind（）是axios自己实现的一个bind方法
    // 返回的是wrap函数，当执行axios时，会执行Axios.prototype.request，仅仅改变this

    
    
  // Copy axios.prototype to instance
  utils.extend(instance, Axios.prototype, context);

  // Copy context to instance
  utils.extend(instance, context);

  return instance;
}
```

这里axios不仅是wrap函数，同时实例上也通过extend挂载了很多方法，成了一个对象。



# Axios核心对象

## 构造函数

```js
/**
 * Create a new instance of Axios
 *
 * @param {Object} instanceConfig The default config for the instance
 */
function Axios(instanceConfig) {
  this.defaults = instanceConfig;
  this.interceptors = {
    request: new InterceptorManager(),
    response: new InterceptorManager()
  };
}


Axios.prototype.request = function request(config) {...}
Axios.prototype.getUri = function getUri(config) {...}

utils.forEach(['delete', 'get', 'head', 'options'], function forEachMethodNoData(method) {...}
utils.forEach(['post', 'put', 'patch'], function forEachMethodWithData(method) {...}
```

主要的操作是

- 给实例设置配置
- 添加拦截器
- 实现request原型方法
- 实现getUri原型方法
- 在原型上挂载各个请求方法

## 合并merge函数

```js
function merge(/* obj1, obj2, obj3, ... */) {
  var result = {};
    // 先创建一个空对象
  function assignValue(val, key) {
    if (typeof result[key] === 'object' && typeof val === 'object') {
        // 如果已存在一个值为对象的键，而后来者也有同为对象的键，需要进行深度合并，递归实现就可以了
      result[key] = merge(result[key], val);
    } else {
      result[key] = val;
    }
  }

  for (var i = 0, l = arguments.length; i < l; i++) {
    forEach(arguments[i], assignValue);
  }
  return result;
}
```



## 配置结构

调用请求时：

```js
utils.forEach(['delete', 'get', 'head', 'options'], function forEachMethodNoData(method) {
  /*eslint func-names:0*/
  Axios.prototype[method] = function(url, config) {
    return this.request(utils.merge(config || {}, {
      method: method,
      url: url
    }));
  };
});

// 他都会去调用request方法
```

在调用request方法时传入配置参数

```js
Axios.prototype.request = function request(config) {
  /*eslint no-param-reassign:0*/
  // Allow for axios('example/url'[, config]) a la fetch API
  if (typeof config === 'string') {
    config = arguments[1] || {};
    config.url = arguments[0];
  } else {
    config = config || {};
  }

  config = mergeConfig(this.defaults, config);

  // Set config.method
  if (config.method) {
    config.method = config.method.toLowerCase();
  } else if (this.defaults.method) {
    config.method = this.defaults.method.toLowerCase();
  } else {
    config.method = 'get';
  }
    。。。
    。。。
}
```

开发对请求的配置参数会不一样，故在request中做一个处理，将其整合为一个对象。

过程如下：

```js
// 大部分的配置规划在这里
config = mergeConfig(this.defaults, config);


//mergeConfig中
// 主要内容分为三大部分
  var valueFromConfig2Keys = ['url', 'method', 'params', 'data'];
  var mergeDeepPropertiesKeys = ['headers', 'auth', 'proxy']; 
  var defaultToConfig2Keys = [
    'baseURL', 'url', 'transformRequest', 'transformResponse', 'paramsSerializer',
    'timeout', 'withCredentials', 'adapter', 'responseType', 'xsrfCookieName',
    'xsrfHeaderName', 'onUploadProgress', 'onDownloadProgress',
    'maxContentLength', 'validateStatus', 'maxRedirects', 'httpAgent',
    'httpsAgent', 'cancelToken', 'socketPath'
  ];

// 第一部分
  utils.forEach(valueFromConfig2Keys, function valueFromConfig2(prop) {
    if (typeof config2[prop] !== 'undefined') {
      config[prop] = config2[prop];
    }
  });
// 采取后者配置优先原则

// 第二部分
  utils.forEach(mergeDeepPropertiesKeys, function mergeDeepProperties(prop) {
    if (utils.isObject(config2[prop])) {
      config[prop] = utils.deepMerge(config1[prop], config2[prop]);
    } else if (typeof config2[prop] !== 'undefined') {
      config[prop] = config2[prop];
    } else if (utils.isObject(config1[prop])) {
      config[prop] = utils.deepMerge(config1[prop]);
        // 这里对一个对象进行深合并，与之前的merge的区别在于，不会去保留原始的对象，而是创建一个新的对象
    } else if (typeof config1[prop] !== 'undefined') {
      config[prop] = config1[prop];
    }
  });

// 第三部分
  utils.forEach(defaultToConfig2Keys, function defaultToConfig2(prop) {
    if (typeof config2[prop] !== 'undefined') {
      config[prop] = config2[prop];
    } else if (typeof config1[prop] !== 'undefined') {
      config[prop] = config1[prop];
    }
  });
// 后者配置优先原则

// 最后进行所以主要部分的合并
  var axiosKeys = valueFromConfig2Keys
    .concat(mergeDeepPropertiesKeys)
    .concat(defaultToConfig2Keys);

// 再最后🙃
// 把config2中有的，但是前面主要属性数组中没有的属性配置传进去
  var otherKeys = Object
    .keys(config2)
    .filter(function filterAxiosKeys(key) {
      return axiosKeys.indexOf(key) === -1;
    });

```



## 拦截器实现

```js
// 先定义一个数组 -- 链
// 和一个完成状态的promise
var chain = [dispatchRequest, undefined];
var promise = Promise.resolve(config);

// 调用拦截器上的request（InterceptorManager实例）中的forEach()
this.interceptors.request.forEach(function (){})

// 那就进去瞅瞅这个InterceptorManager
function InterceptorManager() {
    // 定义一个拦截器数组
  this.handlers = [];
}
InterceptorManager.prototype.use = function use(){}
InterceptorManager.prototype.eject = function eject(){}
InterceptorManager.prototype.forEach = function forEach(){}

// 先就看看这个forEach干了啥
InterceptorManager.prototype.forEach = function forEach(fn) {
  utils.forEach(this.handlers, function forEachHandler(h) {
    if (h !== null) {
      fn(h);
    }
  });
};
// 依次将handlers中的值传入fn中调用
```



### 拦截器回调

forEach是对handlers中每一项进行的操作，而handlers怎么生成的呢，我们用axios定义拦截器的时候是：

```js
axios.intercepters.request.use(succeed, failed)
axios.intercepters.responce.use(succeed, failed)
```

那就看一下定义拦截器的use方法：

```js
InterceptorManager.prototype.use = function use(fulfilled, rejected) {
    // 他将每次定义传入的两个回调函数作为一整个对象添加至handles中，而这一整个对象也就是我们定义的一个拦截器
  this.handlers.push({
    fulfilled: fulfilled,
    rejected: rejected
  });
  return this.handlers.length - 1;
};
```

回到Axios类中，

```js
  // Hook up interceptors middleware
  var chain = [dispatchRequest, undefined];
// 链式数组这里面初始定义的两个东西自有说头，后面再讲
  var promise = Promise.resolve(config);

  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    chain.unshift(interceptor.fulfilled, interceptor.rejected);
      // 将每次的请求拦截器添加至链式数组的头部
  });

  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    chain.push(interceptor.fulfilled, interceptor.rejected);
      // 将每次的响应拦截器添加至链式数组的尾部
  });

// 在这里开始了链式拦截器的执行规划
  while (chain.length) {
      // 每一次拦截作为一个promise任务，一次性传入两个chain中的两个方法，分别对应着我们定义的succeed和failed
      // 一次拦截之后进入下一次拦截，就利用promise，每次将原先的promise覆盖，形成链
    promise = promise.then(chain.shift(), chain.shift());
  }

// 链的基本结构就是 请求拦截-----（dispatchRequest， undefined） -----响应拦截

  return promise;
```

### dispatchRequest

他接受到了之前处理之后的config后

首先进行对config 的规范化处理

```js
// Ensure headers exist
  config.headers = config.headers || {};

  // Transform request data
  config.data = transformData(
    config.data,
    config.headers,
    config.transformRequest
  );

  // Flatten headers
  config.headers = utils.merge(
    config.headers.common || {},
    config.headers[config.method] || {},
    config.headers
  );

```

具体怎么规范的就一笔带过了

```js
// 然后就可以把原先设置的七个方法的对象删了就可以了
utils.forEach(
    ['delete', 'get', 'head', 'post', 'put', 'patch', 'common'],
    function cleanHeaderConfig(method) {
      delete config.headers[method];
    }
  );

```

之后调用适配器



#### adapter

他会根据不同的环境去选取不同的网络请求适配器

```js
function getDefaultAdapter() {
  var adapter;
  if (typeof XMLHttpRequest !== 'undefined') {
    // For browsers use XHR adapter
    adapter = require('./adapters/xhr');
  } else if (typeof process !== 'undefined' && Object.prototype.toString.call(process) === '[object process]') {
    // For node use HTTP adapter
    adapter = require('./adapters/http');
  }
  return adapter;
}
```



下面就只讲下浏览器的网络适配--> `xhrAdapter`

#### xhrAdapter

就是将xhr那一套用promise重新完整 的封装一次

// 首先是对请求的设置

```js
// 首先是对请求的设置
    var requestData = config.data;
    var requestHeaders = config.headers;

    if (utils.isFormData(requestData)) {
      delete requestHeaders['Content-Type']; // Let the browser set it
    }

    var request = new XMLHttpRequest();

// HTTP basic authentication
    if (config.auth) {
      var username = config.auth.username || '';
      var password = config.auth.password || '';
      requestHeaders.Authorization = 'Basic ' + btoa(username + ':' + password);
    }

    var fullPath = buildFullPath(config.baseURL, config.url);
    request.open(config.method.toUpperCase(), buildURL(fullPath, config.params, config.paramsSerializer), true);

    // Set the request timeout in MS
    request.timeout = config.timeout;
```

而后主要看下响应的处理

```js
// Listen for ready state
    request.onreadystatechange = function handleLoad() {
      if (!request || request.readyState !== 4) {
        return;
      }

      // The request errored out and we didn't get a response, this will be
      // handled by onerror instead
      // With one exception: request that using file: protocol, most browsers
      // will return status as 0 even though it's a successful request
      if (request.status === 0 && !(request.responseURL && request.responseURL.indexOf('file:') === 0)) {
        return;
      }

      // Prepare the response
      // 这里！这里！！！瞅过来🐷
      var responseHeaders = 'getAllResponseHeaders' in request ? parseHeaders(request.getAllResponseHeaders()) : null;
      var responseData = !config.responseType || config.responseType === 'text' ? request.responseText : request.response;
      var response = {
        data: responseData,
        status: request.status,
        statusText: request.statusText,
        headers: responseHeaders,
        config: config,
        request: request
      };
// 上面进行一堆对响应的规划，把他们整理好塞进一个responce对象中
        
        // 下面是对响应的具体处理以及这个promise 的执行        
      settle(resolve, reject, response);

      // Clean up request
      request = null;
    };
```

#### settle方法

```js
/**
 * Resolve or reject a Promise based on response status.
 *
 * @param {Function} resolve A function that resolves the promise.
 * @param {Function} reject A function that rejects the promise.
 * @param {object} response The response.
 */
module.exports = function settle(resolve, reject, response) {
  var validateStatus = response.config.validateStatus;
  if (!validateStatus || validateStatus(response.status)) {
    resolve(response);
  } else {
    reject(createError(
      'Request failed with status code ' + response.status,
      response.config,
      null,
      response.request,
      response
    ));
  }
};
```

就是对响应状态的判断，成功就resolve，否则reject。

好，现在回到dispatchRequest中

```js
return adapter(config).then(func1, func2)
```

这里面func1与func2就是分别对adapter返回的promise的两种状态的相应处理。

主要就看下这个成功resolve的处理：

```js
function onAdapterResolution(response) {
    throwIfCancellationRequested(config);

    // Transform response data
    response.data = transformData(
      response.data,
      response.headers,
      config.transformResponse
    );

    return response；
```

就是转换一下响应的数据。细节看代码😏

![image-20200806144138950](C:\Users\Samsara\AppData\Roaming\Typora\typora-user-images\image-20200806144138950.png)

# 再然后，

没了😭

好了，主干部分差不多就是这样了



# 补充API

## create

```js
axios.create = function create(instanceConfig) {
  return createInstance(mergeConfig(axios.defaults, instanceConfig));
};
```

就是生成一个Axios实例，不同的是传入 的参数是默认参数与开发者设定的参数的合并。



## all/spread

```js
axios.all = function all(promises) {
  return Promise.all(promises);
};

axios.spread = require('./helpers/spread');

// ./helpers/spread
function spread(callback) {
  return function wrap(arr) {
    return callback.apply(null, arr);
  };
};

```

用例：
常用于高并发的辅助函数

```js

axios.all([promise1, promise2])
  .then(axios.spread(function (...res) {
    // your code
  }))
```





## 别的嘛

。。。我还没用过呢🤥