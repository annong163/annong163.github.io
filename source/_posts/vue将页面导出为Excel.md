---
title: Vue将页面导出为Excel
author: AnNong
cover: true
categories: VUE
summary: 整合jquery插件table2excel.js，支持自定义文件名、自定义工作表名称、表格自动添加边框。
tags:
  - VUE组件
  - Ant Design Vue
abbrlink: a345773a
date: 2022-04-27 09:54:45
---

## 思路
整合jquery插件table2excel.js ，并且修改部分源码支持自定义文件名、自定义sheet工作表名称、表格自动添加边框。
## 场景
项目中有许多报表之类的统计 涉及表格跨行跨列较多故无法使用xlsx导出（因为需要一行一行处理跨行跨列），所以在Vue中整合了 jquery插件table2excel.js ，并且修改了部分源码支持自定义文件名、自定义sheet工作表名称、表格自动添加边框。

## 效果图

![](/img/导出为excel1.png)
![](/img/导出为excel2.png)


## 详细步骤
### 1.首先需要在Vue中引入jquery
```javascript
 npm install jquery
```
### 2.将下载好的table2excel放到项目中 比如我们用的antd脚手架直接放到util下(下载链接在文末)
![](/img/导出为excel3.png)


### 3.修改table2excel源码，在第一行引入Vue中的jquery （最新文件已设置完毕，可忽略）
```javascript
import jQuery from 'jquery'
```

### 4.在项目入口main.js中引入
```javascript
import './utils/table2excel' // global table2excel
```

### 5.在demo.vue中使用table2excel
#### 5.1 引入jquery
```javascript
import $ from 'jquery'
```
#### 5.2 table中加入id
![](/img/导出为excel4.png)

#### 5.3 在methods方法中使用
```javascript

methods: {
    exportExcel() {
      $("#aTable").table2excel({
         exclude: ".noExl",
         sheetName: "八项数据管控",
         filename: "八项数据管控",
         exclude_img: true,
         exclude_links: true,
         exclude_inputs: true
      });
    }
}
```
## 下载链接
[https://annong.lanzous.com/iDU8jnarhah](https://annong.lanzous.com/iDU8jnarhah)
