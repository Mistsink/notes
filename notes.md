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

















# 😥`Vue-Cli` 篇



1. `axios`访问不到本地的文件：
   **A:** 在`vue-cli`升级至@3+后，静态文件放在public文件夹下，故请求的根路径变成了：  http://localhost:8080/public/api
   记得将静态文件放在public下，而不再是static

























































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

   