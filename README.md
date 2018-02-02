总结一下日常开发问题，涵盖适配性，兼容性，性能，新技术的使用等方面

问题收集的形式是在此项目中提`issues`，定期收集整理到`README.md`中，鼓励提交`Merge Request`，提交 demo 页面说明问题以及解决问题的办法

##### hybrid
1. iOS webview使用的是UIwebview组件，该webview有以下问题
    1. 用户滚动页面时，页面处于阻塞状态，所有的渲染和JS逻辑都不执行，因此无法实现非常流畅的无限滚动加载功能。
    2. 输入框focus呼出键盘时，`position: fixed`定位元素会乱飞，因此遇到“输入框紧贴键盘弹出”这样的需求需要格外小心
    3. iOS11中，`position: fixed; position: sticky`元素在页面过程中会闪动，消失，直到页面滚动停止时出现
2. iOS/Android 不支持webview内长按图片，弹出系统组件保存功能
3. UIwebview阻塞promise，中断页面逻辑。[参考](https://juejin.im/post/59c37b7ff265da066e173751)

##### canvas
1. html2canvas图片跨域问题
2. toDataUrl图片模糊。canvas在导出图片时按照canvas的width&height作为图片的分辨率，在移动端一般需要2倍分辨率以上的图，这时就容易出现导出图片模糊的情况。

```
<meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no">
```
通常情况下我们约定移动端页面的缩放比是1.0，此时的比例关系是 `canvas.width = window.devicePixelRatio * canvas.style.width;`

##### video/audio
1. 在任何情况下，小心那些页面打开加载完成后自动播放的各种需求，iOS/Android不同系统不同设备的表现不一致，基本都无法自动播放（系统限制）。必须要用户手动触发才能播放。

```
$(function(){
    $(video)[0].play()  //fail
})

$('btn').on('click', function(){
    $(video)[0].play()  //success
})
```

2. Android X5内核开启同层播放器。默认情况下，使用了X5内核的Android微信webview会统一给页面的视频加播放器，此播放器优先级最高，无法覆盖，播放中无法隐藏，需要对video进行特殊处理

```
<video src="http://xxx.mp4" x5-video-player-type="h5"></video>
```
[详细请看 -  H5同层播放器接入规范](https://x5.tencent.com/tbs/guide/video.html)

3. iOS开启内嵌播放，低于iOS10机型，在webview开启了相关功能后，可以通过增加`-webkit-playsinline`实现，高于iOS10机型，增加`playsinline`
```
<video src="http://xxx.mp4" -webkit-playsinline playsinline></video>
```
4. Android/iOS old version 不支持在webview中同时播放多个音频，后播放的音频会把前一个顶掉，有背景音乐+交互音效的需求需要注意

##### webGL
1、图片面积与内存占用情况成正比
   所有WebGL中使用的图片，会以二进制数组的形式上传到GPU内存中。因为数组中保存的是每个像素的rgb(a)信息，因此数组所占内存与图片面积（像素数量）成正比，与图片本身文件大小无关。(图片文件本身都是压缩格式，体积与像素数量不成正比)
   IOS设备中纹理大小最大不超过 4096 * 4096，较老的设备中最大纹理大小为 2048 * 2048。考虑到内存占用情况，一般使用的纹理大小宜限制在2048 * 2048以下

2、IOS中，APP切到后台时，系统默认禁止WebGL继续渲染。但后台APP中WebView页面的代码仍在执行，此时调用WebGL会导致APP报错崩溃。
   在IOS APP 3.5.0版本中添加了 onAppState 接口。APP在切到后台前、回到前台后会分别通知WebView相关状态码：
   
```
window.onAppState = function(e){
   if (e == 0) {
   	//回到前台后
   }
   if (e == 1) {
   	//切到后台前
   }
}
```

3、APP是否会崩溃不仅与页面逻辑、资源占用有关，不同设备下的表现也不一致。
   例如上述第2点，某些IOS设备满足条件时100%几率崩溃，但在一些设备中完全没有崩溃现象。
   内存原因导致崩溃也可能与设备本身系统、硬件有关，实际开发中只能尽量减少图片素材的像素数，降低崩溃风险。

4、WebGL帧率在30帧时，动画表现效果良好，同时对CPU/GPU的性能消耗降低
   黑五活动H5页面中，商品数量较多，同时页面头部WebGL动画未做暂停的状态下，页面性能未见大幅度降低。


