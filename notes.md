# 😭踩呀嘛踩呀嘛踩坑啊~

心血来潮就写点吧，不然。。。emmm，就太监了吧。。。







# `Vue-router`篇

1. history模式下 `Vue`组件内守卫`beforeRouteUpdate`不起作用

   **A:**   history模式下目前确实不行，改为hash模式下，当设置的该相关路由改变时会触发











# `Better-Scroll`篇

1. 真机测试时无法触发滚动事件：
   **A:**    初始化时将click设为true。

   ```js
   this.scroll =  new BScroll(this.$refs.wrapper, {
       click: true
   })
   ```

   - ![image-20200730201101202](C:\Users\Samsara\AppData\Roaming\Typora\typora-user-images\image-20200730201101202.png)









# `Git` 篇

1. 远程关联 :
   获取到`github`仓库地址后，将本地仓库与`github`仓库关联

   输入指令： 

   ```
   git remote add your-warehouse-address
   ```

   