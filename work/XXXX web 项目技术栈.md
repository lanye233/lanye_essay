# 一、运用框架、API
* Vue  
* Vuex
* 百度地图接口
* element
# 二、项目结构
* 布局
  * 左侧百度地图接口
  * 右侧垂直导航栏
* 功能
  * 实时数据
  * 视频数据
  * 地图坐标查询
```
├─components
│ Aircondition.vue
│ BaiduMap.vue
│ Car.vue
│ CurrentFault.vue
│ DashBoard.vue //速度仪表盘
│ Device.vue
│ Door.vue
│ Electric.vue
│ FaultPage.vue
│ History.vue
│ HistoryFault.vue
│ Interface.vue //弹出窗口主界面
│ main.vue //入口文件
│ Modal.vue
│ Net.vue
│ ProgressBar.vue //自定义双端进度条
│ Video.vue
│ WindowFrame.vue //弹出窗口
│
├─css
│ demo.css
│
├─entity
│ singleCar.js
│ singleFault.js
│
├─router
│ index.js
│
├─service
│ AnalysisMsg.js //实时数据websocket连接所用函数
│ WebSocket.js //实时数据websocket连接
│ wfs.js //wfs连接视频端口，解码246流
│
└─store
│ store.js //vuex管理状态
│
└─modules
airStore.js
carStore.js
deviceStore.js
doorStore.js
electricStore.js
faultStore.js
headInfoStore.js
interfaceStore.js
netStore.js
stateInfoStore.js
```
# 三、整体功能
## （1）、点击线路，出现线路所属车辆
### 1.车辆列表（总表和显示表）：
var car_list = new Array(); *//车辆列表*

var show_list = new Array()

car_list[0] = new singleCar(1,8888)

car_list[1] = new singleCar(1,7777)

car_list[2] = new singleCar(2,6666)

car_list[3] = new singleCar(3,5555)

car_list[4] = new singleCar(3,4444)

car_list[5] = new singleCar(4,3333)

### 2.车辆实体：
```
//singleCar.js
function singleCar(carLine, carNumber, mode) {
  this.carLine = carLine
  this.carNumber = carNumber
  this.mode = "离线"
}
export {
  singleCar,
}
```
### 3.点击线路，将该线路车辆加入到显示列表：
```
getCars(i){
  this.carLineIndex = i;
  show_list.length = 0
  this.setState()
  console.log("--------------------------------------------------------------------选择线路:"+this.carLineIndex)
  for(var j=0;j<car_list.length;j++){
    if(car_list[j].carLine == i){
      show_list.push(car_list[j])
    }
  }
}
```
### 4.点击具体车辆，检查车辆 mode，为运行时发送请求实时数据，并将车号 carNumber 传到新建小窗口页面：
```
<el-submenu index="1">
<template slot="title"><i class="el-icon-truck"></i>机车查询结果列表</template>
<el-menu-item
v-for="(item) in show_list"
v-bind:key="item.carNumber"
@click=" (item.mode == '运行') ? false:true && sendMessage(JSON.stringify({type:3,carNum:item.carNumber}))"
@click.native="showOne(item.mode,item.carNumber)"
>{{item.carNumber}}&nbsp;&nbsp;&nbsp;&nbsp;{{item.mode}}</el-menu-item>
</el-submenu>
```
## （2）、Vuex 状态管理
```
import Vue from 'vue'
const state = {
  test:[]
  //这里填写参数信息
}
const getters = {
  test: (state) => {
    return state.test
  }
}
const actions = {
}
const mutations = {
  //这里写你的函数
  createInfo(state) {
    Vue.set(state.test, 0, "测试")
    //由于 Vue 不能检测到对象属性的添加或删除，所以不能用 test[0]="测试"这种方式为参数添加新属性或新值。
  }
}
export default {
  state,
  getters,
  actions,
  mutations,
}
```
## （3）、H.264 裸流视频播放
用到了 github 上的 [ChihChengYang](https://github.com/ChihChengYang) 的 [wfs.js](https://github.com/ChihChengYang/wfs.js) 和 [index.html](https://github.com/ChihChengYang/wfs.js/blob/master/demo/index.html)

### 1.在 Vue 中可以这样写视频页：
```
<template>
  <div id="Video">
    <div id="videoBorder">
      <video id="video1" width="640" height="480" autoplay></video>
    </div>
  </div>
</template>
<script>  
import wfs from '../service/wfs.js'
import {split,initWebpack,websocketSend, websocket} from '../service/WebSocket'
window.onload = function() {
            if (Wfs.isSupported()) {
                var video1 = document.getElementById("video1"),
                    wfs = new Wfs();
                wfs.attachMedia(video1, 'ch1');
            }
        };
...
```
### 2.然后在你添加到项目中的 wfs.js 文件中更改你的 websocket 连接路径，搜索 onMediaAttached 找到创建 websocket 的函数，按以下方式修改：
```
key: 'onMediaAttached',
value: function onMediaAttached(data) {
    if (data.websocketName != undefined) {
        var url = "你的连接地址"
        var client = new WebSocket(url)
        //clientSocket = client
        //(如果你要调用wfs.js中的websocket发送函数，你需要用到这行代码)
        this.wfs.attachWebsocket(client, data.channelName);
    } else {
        console.log('websocketName ERROE!!!');
    }
}
```
* 去掉缓存：注释 onSBUpdateEnd 方法里的 this.mediaSource.endOfStream() 和 that.media.play()这两行，然后在 video 标签里加上 autoplay，去掉 controls。
* 调整播放速度：将 this.H264_TIMEBASE = 3000 中的数值改为 4500（或其他你觉得合适的数值），有两个地方需要修改。
### 3.现在你就可以在浏览器上运行你的项目，看看是否有画面出现。在项目试验过程中可能出现以下问题：
1. 花屏：可能是服务端出了问题，可以在服务端写文件进行测试。
2. 可能会出现 mediaError 的打印日志，但并不影响画面播放。
### 4.如果你想在外部调用 websocket 发送你的项目所要求的请求视频数据，请接着看。
（参考网址： [JS如何在外部调用函数内部的函数](https://blog.csdn.net/weixin_43694639/article/details/88723280) ）

* 下面是接受 H.264 裸流并进行转化的函数：
```
key: 'receiveSocketMessage',
value: function receiveSocketMessage(event) {
    this.buf = new Uint8Array(event.data);
    var copy = new Uint8Array(this.buf);
    if (this.mediaType === 'FMp4') {
        this.wfs.trigger(_events2.default.WEBSOCKET_ATTACHED, { payload: copy });
    }
    if (this.mediaType === 'H264Raw') {
        this.wfs.trigger(_events2.default.H264_DATA_PARSING, { data: copy });
    }
}
```
* 在整个 wfs.js 文件函数的最外层定义变量：
```
var sendMsg;
var clientSocket;
```
* 在 send 函数中赋值：
```
key: 'onWebsocketMessageSending',
value: function onWebsocketMessageSending(i) {
    clientSocket.send(i)
    console.log('发送数据：' + i)
        //this.client.send(i)
        //this.client.send(JSON.stringify({ type: 2, carNum: 8888 }))
    sendMsg = onWebsocketMessageSending
}
```
* 在函数最后 export 出去：
```
export {
    sendMsg,
    clientSocket,
}
```
* 在你想调用的页面 import：
```
import {sendMsg,clientSocket} from '../service/wfs.js'
```
>使用过程中的提问和解答：[https://github.com/ChihChengYang/wfs.js/issues/31](https://github.com/ChihChengYang/wfs.js/issues/31)
## （4）、axios 进行 http 请求
1.在 main.js 中添加以下代码：

```
//main.js
import axios from 'axios'
Vue.prototype.$http = axios;
```
2.新建 http.js 文件：
```
//http.js
import axios from 'axios'
export default {
    install(Vue) {
        //install方法，向Vue实例中添加一个$http方法
        Vue.prototype.$http = axios
        Vue.$http = axios
    },
    $http: axios
}
export const Http = axios
```
3.在要进行 http 请求的页面写请求方法：
```
//http请求方法
requestData:function(){
      console.log('http请求,'+'time:'+this.duration);
      console.log('url:'+url+this.className)
      this.$http.get(
        url,
        {
          params:{
            date1:this.duration[0],
            date2:this.duration[1],
          }
        }    
      )
        .then(res => {
          var ans = res.data;
          console.log('res.data:'+res.data);
          console.log('value0:'+this.duration);
          if(res.data.code == '0'){
            console.log('yes');
          }else{
            alert('数据不存在');
          }
        })
        .catch(err => {
          alert('请求失败！');
        })
```
>P.s. 根据项目需求，具体的请求方法参数会不一样。
# 四、其他
## 1.学到的一些基础知识点：
* CSS 中的 position 定位：relative&absolute
## 2.不足与可改进之处：
* 主界面的车辆车门，用 flex 布局或网格布局会更好
* 网络界面，选定布局，由大到小来画会更有规律，更好修改
