# Vue-cropp

一款简单易用的vue图片裁剪插件，适合用于头像裁剪等功能，兼顾有vue2和vue3，但浏览器兼容做的不是很好，后续会逐步改进，欢迎使用。

## 特色

- 自定义画布大小、裁剪框大小
- 自定义背景色
- 支持缩放、拖动、裁剪框拖动大小
- 支持移动端，可适配大小，可拖动、缩放图片

## 项目地址

Github: https://github.com/sun0317tao/vue-crpp

gitee: https://gitee.com/sun0317tao/vue-cropp#type-support-for-vue-imports-in-ts

(觉得还不错的，请赏一个star吧。)

## 使用方法

1、安装

```
npm install vue-cropp --save-dev # vue2
npm install vue-cropp --save-dev # vue3
```

2、将 vue-cropp 引入项目中

```
import Cropp from "vue-cropp";
export default {
        components:{
            Cropp
        },
...
}
<Cropp
      :croppwidth="410"
      :croppheight="600"
      :fileOrUrl="fileval"
      :backGroundColor="false"
      @moveupCropp="moveupCropp"
      @confirmCropp="confirmCropp"
    />
```



## 参数说明

| 属性名           | 作用                                     | 类型           | 必填 | 默认值 |
| ---------------- | ---------------------------------------- | -------------- | ---- | :----: |
| croppwidth       | 画布宽                                   | number         | 否   |  800   |
| croppheight      | 画布高                                   | number         | 否   |  400   |
| croppBoxWidth    | 裁剪盒子宽                               | number         | 否   |  200   |
| croppBoxHeight   | 裁剪盒子高                               | number         | 否   |  200   |
| scalenum         | 放大缩小的速度（值越大放大或缩小的越快） | number         | 否   |  0.01  |
| fileOrUrl        | 图片file文件对象                         | object         | 否   |   “”   |
| backGroundColor  | 画布背景色，支持颜色值和布尔值           | Boolean/string | 否   |  #000  |
| croppfourthColor | 自定义裁剪框四个角颜色                   | string         | 否   |  #fff  |

## 钩子函数

| 属性名       | 作用                             | 类型     | 必填 | 返回值       |
| ------------ | -------------------------------- | -------- | ---- | ------------ |
| moveupCropp  | 裁剪框或图片移动后触发的钩子函数 | function | 否   | Base64       |
| confirmCropp | 点击画布中的确认按钮触发的事件   | function | 否   | file文件对象 |



## 更新日志

**1.0.0**

- 第一版开发完成。

**1.1.2**

- 去除项目中的log，优化移动端第一次在裁剪框中拖动和缩放不了图片的问题

1.1.3

- 优化图片清晰度问题，但是裁剪出来的图片为了保持清晰度图片的默认像素会放大当前设备屏幕的像素比
- 增加可自定义裁剪框四个角颜色
