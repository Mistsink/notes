# 😭踩呀嘛踩呀嘛踩坑鸭~~~

心血来潮就写点吧，不然。。。emmm，就太监了吧。。。











# 😬`Vue` 篇

1. 组件生命周期：
   ![Vue 实例生命周期](https://cn.vuejs.org/images/lifecycle.png)

2. 🙏生命周期相关--小tips：

   - 组件创建完成时(created)，data 已经被 observed

   

3. `v-if` VS `v-show`：

   - `v-if` 是“真正”的条件渲染，因为它会**确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。**

     `v-if` 也是**惰性的**：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

   - `v-show` 就简单得多——不管初始条件是什么，**元素总是会被渲染，并且只是简单地基于 `CSS` 进行切换。**

   一般来说，**`v-if` 有更高的切换开销，而 `v-show` 有更高的初始渲染开销**。因此，如果需要非常频繁地切换，则使用 `v-show` 较好；如果在运行时条件很少改变，则使用 `v-if` 较好。
   
4. [基础组件的自动化全局注册](https://cn.vuejs.org/v2/guide/components-registration.html#基础组件的自动化全局注册)

   如果你恰好使用了 `webpack` (或在内部使用了 `webpack` 的 [Vue CLI 3+](https://github.com/vuejs/vue-cli))，那么就可以使用 `require.context` 只全局注册这些非常通用的基础组件。这里有一份可以让你在应用入口文件 (比如 `src/main.js`) 中全局导入基础组件的示例代码：

   ```js
   import Vue from 'vue'
   import upperFirst from 'lodash/upperFirst'
   import camelCase from 'lodash/camelCase'
   
   const requireComponent = require.context(
     // 其组件目录的相对路径
     './components',
     // 是否查询其子目录
     false,
     // 匹配基础组件文件名的正则表达式
     /Base[A-Z]\w+\.(vue|js)$/
   )
   
   requireComponent.keys().forEach(fileName => {
     // 获取组件配置
     const componentConfig = requireComponent(fileName)
   
     // 获取组件的 PascalCase 命名
     const componentName = upperFirst(
       camelCase(
         // 获取和目录深度无关的文件名
         fileName
           .split('/')
           .pop()
           .replace(/\.\w+$/, '')
       )
     )
   
     // 全局注册组件
     Vue.component(
       componentName,
       // 如果这个组件选项是通过 `export default` 导出的，
       // 那么就会优先使用 `.default`，
       // 否则回退到使用模块的根。
       componentConfig.default || componentConfig
     )
   })
   ```

5. ###### [Prop 的大小写 (camelCase vs kebab-case)](https://cn.vuejs.org/v2/guide/components-props.html#Prop-的大小写-camelCase-vs-kebab-case)

   HTML 中的 attribute 名是大小写不敏感的，所以浏览器会把所有大写字符解释为小写字符。这意味着当你使用 DOM 中的模板时，`camelCase `(驼峰命名法) 的 prop 名需要使用其等价的 kebab-case (短横线分隔命名) 命名：

   ```js
   Vue.component('blog-post', {
     // 在 JavaScript 中是 camelCase 的
     props: ['postTitle'],
     template: '<h3>{{ postTitle }}</h3>'
   })
   <!-- 在 HTML 中是 kebab-case 的 -->
   <blog-post post-title="hello!"></blog-post>
   ```

   重申一次，如果你使用字符串模板，那么这个限制就不存在了。

6. ###### [替换/合并已有的 Attribute](https://cn.vuejs.org/v2/guide/components-props.html#替换-合并已有的-Attribute)

   对于绝大多数 attribute 来说，从外部提供给组件的值会替换掉组件内部设置好的值。所以如果传入 `type="text"` 就会替换掉 `type="date"` 并把它破坏！庆幸的是，`class` 和 `style` attribute 会稍微智能一些，即两边的值会被合并起来，从而得到最终的值：`form-control date-picker-theme-dark`。















# 😥`Vue-Cli` 篇



1. `axios`访问不到本地的文件：
   **A:** 在`vue-cli`升级至@3+后，静态文件放在public文件夹下，故请求的根路径变成了：  http://localhost:8080/public/api
   记得将静态文件放在public下，而不再是static
   
2. vue/cli创建项目时报错：Failed to check for updates，且导致项目创建失败

   **A：**找到`.vuerc`文件，查看详细信息：

   - 淘宝镜像源是否启用
   - 包管理工具是npm还是yarn
   - vue/cli内置的包管理是yarn，yarn又是hadoop集群的资源管理系统，故查看是否是系统变量中hadoop的变量路径导致vue/cli使用了hadoop的yarn























































# 😁`Vuex` 篇



1. `getter` 在通过**属性访问**时是作为` Vue` 的**响应式系统的一部分缓存其中**的。

   `getter` 在通过**方法访问**时，每次都会去**进行调用，而不会缓存结果**。































# 🙏`Vue-router`篇

1. history模式下 `Vue`组件内守卫`beforeRouteUpdate`不起作用

   **A:**   history模式下目前确实不行，改为hash模式下，当设置的该相关路由改变时会触发
   
   
   
2. 重定向与别名：

   ```js
   const router = new VueRouter({
     routes: [
       { path: '/a', redirect: '/b'}, // 重定向
       { path: '/a', component: A, alias: '/b' }  // 别名
     ]
   })
   ```

   重定向： 访问/a时，跳到/b(虽然不严谨，反正这么个表现), URL上也会改为真正的/b
   别名： 访问/a时， 就是/a这个页面，访问/b时，实则为访问/a的页面，但URL上仍保持着/b的名字
   
   
   
3. **完整的导航解析流程：**

   1. 导航被触发。
   2. 在失活的组件里调用 `beforeRouteLeave` 守卫。
   3. 调用全局的 `beforeEach` 守卫。
   4. 在重用的组件里调用 `beforeRouteUpdate` 守卫 (2.2+)。
   5. 在路由配置里调用 `beforeEnter`。
   6. 解析异步路由组件。
   7. 在被激活的组件里调用 `beforeRouteEnter`。
   8. 调用全局的 `beforeResolve` 守卫 (2.5+)。
   9. 导航被确认。
   10. 调用全局的 `afterEach` 钩子。
   11. 触发 DOM 更新。
   12. 用创建好的实例调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数。
   
4. 在自定义路由跳转事件时，触发事件后浏览器会报错：
   `**Uncaught (in promise) NavigationDuplicated：。。。。**`
   解决方法：
        据翻看大佬的解释，`vue-router `≥3.0版本回调形式以及改成`promise api`的形式了，返回的是一个promise，如果没有捕获到错误，控制台始终会出现如图的警告，针对于路由跳转相同的地址，目前的解决方案: `this.$router.push('/location').catch(err => { console.log(err) })`

   ```js
   
   import Router from 'vue-router'
    
   const originalPush = Router.prototype.push
   Router.prototype.push = function push(location) {
     return originalPush.call(this, location).catch(err => err)
   }
   ```

   















































# 💎`Better-Scroll`篇

1. 真机测试时无法触发滚动事件：
   **A:**    初始化时将click设为true。

   ```js
   this.scroll =  new BScroll(this.$refs.wrapper, {
       click: true
   })
   ```

   - ![image-20200730201101202](C:\Users\Samsara\AppData\Roaming\Typora\typora-user-images\image-20200730201101202.png)

































# 🔑`Git` 篇

1. 远程关联 :
   获取到`github`仓库地址后，将本地仓库与`github`仓库关联

   输入指令： 

   ```
   git remote add your-warehouse-address
   ```

2. git或gitlab修改密码之后，报错remote: HTTP Basic: Access denied，fatal: Authentication failed for ‘git或gitlab地址’：
   ![image-20200817112455952](C:\Users\Samsara\AppData\Roaming\Typora\typora-user-images\image-20200817112455952.png)
   方案一：

   - ![image-20200817112517887](C:\Users\Samsara\AppData\Roaming\Typora\typora-user-images\image-20200817112517887.png)
   - ![image-20200817112538775](C:\Users\Samsara\AppData\Roaming\Typora\typora-user-images\image-20200817112538775.png)

   方案二：

   - ```
     git config --system --unset credential.helper
     git config --global credential.helper store
     ```

     再push就会提示输入用户名和新密码。