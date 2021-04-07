微信小程序中的 `image` 组件在使用时，有一个 `mode` 属性，该属性决定了图片的裁剪、缩放的模式。

> image组件默认宽度320px（官方说是300px）、高度240px

**mode 的合法值**

| 值           | 说明                                                         |
| :----------- | :----------------------------------------------------------- |
| scaleToFill  | 缩放模式，不保持纵横比缩放图片，使图片的宽高完全==**拉伸**==至填满 image 元素 |
| aspectFit    | 缩放模式，保持纵横比缩放图片，使图片的长边能完全显示出来。也就是说，可以完整地将图片显示出来。 |
| aspectFill   | 缩放模式，保持纵横比缩放图片，只保证图片的短边能完全显示出来。也就是说，图片通常只在水平或垂直方向是完整的，另一个方向将会发生截取。 |
| widthFix     | 缩放模式，宽度不变，高度自动变化，保持原图宽高比不变         |
| top          | 裁剪模式，不缩放图片，只显示图片的顶部区域                   |
| bottom       | 裁剪模式，不缩放图片，只显示图片的底部区域                   |
| center       | 裁剪模式，不缩放图片，只显示图片的中间区域                   |
| left         | 裁剪模式，不缩放图片，只显示图片的左边区域                   |
| right        | 裁剪模式，不缩放图片，只显示图片的右边区域                   |
| top left     | 裁剪模式，不缩放图片，只显示图片的左上边区域                 |
| top right    | 裁剪模式，不缩放图片，只显示图片的右上边区域                 |
| bottom left  | 裁剪模式，不缩放图片，只显示图片的左下边区域                 |
| bottom right | 裁剪模式，不缩放图片，只显示图片的右下边区域                 |



```html
<!--image.wxml-->
<view class="page">
  <view class="page__bd">
    <view class="section section_gap" wx:for="{{array}}" wx:for-item="item">
      <view class="section__title">{{index + 1}}、{{item.text}}</view>
      <view class="section__ctn">
        <image style="width:100%; background-color: #eeeeee;" mode="{{item.mode}}" src="{{src}}"></image>
      </view>
    </view>
  </view>
</view>
```

```js
// image.js
Page({
  data: {
    array: [
      {
        mode: 'scaleToFill',
        text: 'scaleToFill：不保持纵横比缩放图片，使图片完全适应'
      }, 
      {
        mode: 'aspectFit',
        text: 'aspectFit：保持纵横比缩放图片，使图片的长边能完全显示出来'
      }, 
      {
        mode: 'aspectFill',
        text: 'aspectFill：保持纵横比缩放图片，只保证图片的短边能完全显示出来'
      }, 
      {
        mode: 'widthFix',
        text: 'widthFix：宽度不变，高度自动变化，保持原图宽高比不变'
      }, 
      {
        mode: 'top',
        text: 'top：不缩放图片，只显示图片的顶部区域'
      }, 
      {
        mode: 'bottom',
        text: 'bottom：不缩放图片，只显示图片的底部区域'
      }, 
      {
        mode: 'center',
        text: 'center：不缩放图片，只显示图片的中间区域'
      }, 
      {
        mode: 'left',
        text: 'left：不缩放图片，只显示图片的左边区域'
      }, 
      {
        mode: 'right',
        text: 'right：不缩放图片，只显示图片的右边边区域'
      }, 
      {
        mode: 'top left',
        text: 'top left：不缩放图片，只显示图片的左上边区域'
      }, 
      {
        mode: 'top right',
        text: 'top right：不缩放图片，只显示图片的右上边区域'
      }, 
      {
        mode: 'bottom left',
        text: 'bottom left：不缩放图片，只显示图片的左下边区域'
      }, 
      {
        mode: 'bottom right',
        text: 'bottom right：不缩放图片，只显示图片的右下边区域'
      }
    ],
    src: 'https://res.wx.qq.com/wxdoc/dist/assets/img/0.4cb08bb4.jpg'
  },
})
```

