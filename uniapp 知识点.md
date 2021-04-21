UniApp  细节整理

1.pages.json 中全局设置  

 onReachBottomDistance  页面上拉触底事件触发时距页面底部距离，单位只支持px

2.组件封装使用

`$emit`，`$on`，`$off`常用于跨页面，跨组件通讯，这里为了方便演示放在同一个页面

### [uni。$ emit（eventName，OBJECT）](https://uniapp.dcloud.net.cn/api/window/communication?id=emit)

触发变量的自定义事件，附加参数都会传给监听器某些函数。

### [uni。$ on（eventName，callback）](https://uniapp.dcloud.net.cn/api/window/communication?id=on)

侦听监听的自定义事件，事件由`uni.$emit`触发，多个函数会接收事件触发函数的触发参数。

3.小程序上传打包注意事项

- 设置appid


- 配置服务器地址 域名


- 提交审核，上线版


4.

vue 页面宽度750upx 
push 数据后添加 unshift数组前添加 
view 换行；block一行

5.字太多显示省略号

​        white-space: nowrap;
​		overflow: hidden;
​		text-overflow: ellipsis;

6.vue特点

​    轻量级框架

  双向数据绑定

 指令就行交互

 组件化

客户端路由

状态管理

7.spring boot 核心就是自动配置

 springboot 类似 ssm框架

spring cloud 多个服务，多个springboot项目

可以不依赖tomcat等容器来运行，可以以jar包形势运行

8.uni.showActionSheet 长按显示操作按钮

9微信端 调用 

![image-20210302143402379](C:\Users\My\AppData\Roaming\Typora\typora-user-images\image-20210302143402379.png)

https://www.huzhan.com/code/goods377438.html



### 防抖 节流

https://segmentfault.com/a/1190000018428170

### wangeditor  轻量级富文本编辑框





# uni-app布局之display: flex

我们主要涉及到以下6个属性

##### flex-direction 容器内项目的排列方向(默认横向排列)

- row 横向布局
- column 竖直布局

##### justify-content 项目在主轴上的对齐方式

- flex-start 主轴从开始位置开始排列
- flex-end 主轴从结束位置开始排列
- center 主轴居中排列
- space-between 主轴两头对齐等间距排列

##### align-items 项目在交叉轴上如何对齐

- flex-start 交叉轴从开始位置开始排列
- flex-end 交叉轴从结束位置开始排列
- center 交叉轴居中排列
- stretch 交叉轴拉伸铺满

##### flex-wrap 容器内项目换行方式

- inherit 继承父试图的换行规则
- initial 初始的
- nowrap 不换行(默认)
- wrap 换行
- wrap-reverse 换行

##### flex-flow 是flex-direction和flex-wrap的简写方式

##### align-content 定义了多根轴线的对齐方式。如果项目只有一根轴线则不起作用

- flex-start 靠近开始位置排列
- flex-end 靠近结束位置排列
- center 居中排列
- space-around 等间距排列(默认)
- space-between 两头对齐等间距排列
- inherit 集成父试图的值
- initial 初始值
- stretch 拉伸铺满

 