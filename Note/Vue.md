# Vue

## 1,目录结构

![image-20191029225932785](E:\NotePadWorkSpace\Note\img\vue-001.png)



## 2，常用API

 https://cn.vuejs.org/v2/api/#v-model 



## 3，官方教程

 https://cn.vuejs.org/v2/guide/installation.html 



## 4，JS常用方法

 https://www.w3school.com.cn/jsref/index.asp 



## 5，常用模块

### 5.1,  组件库`element-ui`

安装命令，其中 --save用于将模块名保存在package.json中。

```javascript
npm install element-ui --save
```



然后在`main.js`中添加如下语句，将模块引入

```javascript
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';

Vue.use(ElementUI);
```



官方使用说明：

https://element.eleme.cn/#/zh-CN/component





