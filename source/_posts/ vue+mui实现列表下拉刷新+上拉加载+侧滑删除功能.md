---

title: vue+mui实现列表下拉刷新+上拉加载+侧滑删除功能

date: 2017-05-31 15:56:00

tags:

- web

---

# 注意事项：
1）点击事件需使用@tap($event)
2）上拉刷新没有数据的话，可以通过如下代码参数传入true禁止上拉加载，并显示相应提示
```
mui('#refreshContainer').pullRefresh().endPullupToRefresh(true);  
```

3）侧滑删除后关闭对应侧滑按钮方法，其中elem为@tap($event)事件中传入的参数
```
mui.swipeoutClose(elem.target.parentNode.parentNode); 
```

# 以下为全部功能代码
```
<div id="wdxxVue">
<div id="refreshContainer" class="mui-content mui-scroll-wrapper">
<div class="mui-scroll">
<ul id="OA_task_1" class="mui-table-view">
<li class="mui-table-view-cell" v-for="wdxx in wdxxs">
<div class="mui-slider-right mui-disabled">
<a class="mui-btn mui-btn-yellow" @tap="iKnow(wdxx.cBH, $event)">我知道了</a>
</div>
<div class="mui-slider-handle">
<div class="wdxx_nr"><span></span>{{wdxx.cConent}}</div>
</div>
</li>
</ul>
</div>
</div>
</div>
```

```
//mui初始化，配置下拉刷新
mui.init({
pullRefresh: {
container: '#refreshContainer',
down: {
style: 'circle',
offset: '0px',
auto: true,
callback: pulldownRefresh
},
up: {
contentrefresh: '正在加载...',
callback: pullupRefresh
}
}
});
(function($) {
$(".mui-scroll-wrapper").scroll({
bounce: false, //滚动条是否有弹力默认是true
indicators: true, //是否显示滚动条,默认是true
});
})(mui);
/**
* 下拉刷新获取最新列表 
*/
function pulldownRefresh() {
if (window.plus && plus.networkinfo.getCurrentType() === plus.networkinfo.CONNECTION_NONE) {
plus.nativeUI.toast('似乎已断开与互联网的连接', {
verticalAlign: 'top'
});
return;
}
wdxxVue.pageNum = 1;
var url = BASE_URL_MSG + "list/" + localStorage.userId + "/" + wdxxVue.pageNum + "/" + wdxxVue.pageSize;
axios.get(url)
.then(function(response) {
mui('#refreshContainer').pullRefresh().endPulldownToRefresh(); //停止下拉刷新
mui('#refreshContainer').pullRefresh().enablePullupToRefresh(); //允许上拉加载
var code = response.data.code;
if (code == 200) {
var datas = response.data.data;
if (datas.length < wdxxVue.pageSize) {
wdxxVue.noMoreData = true;
}
wdxxVue.wdxxs = datas;
} else {
console.log("获取消息失败");
}
})
.catch(function(error) {
console.log(error);
});
}
/**
* 上拉加载拉取历史列表 
*/
function pullupRefresh() {
wdxxVue.pageNum++;
var url = BASE_URL_MSG + "list/" + localStorage.userId + "/" + wdxxVue.pageNum + "/" + wdxxVue.pageSize;
axios.get(url)
.then(function(response) {
mui('#refreshContainer').pullRefresh().endPullupToRefresh(wdxxVue.noMoreData); //没有更多数据显示相应提示
var code = response.data.code;
if (code == 200) {
var datas = response.data.data;
if (datas.length < wdxxVue.pageSize) {
wdxxVue.noMoreData = true;
}
wdxxVue.wdxxs = wdxxVue.wdxxs.concat(datas);
} else {
console.log("获取消息失败");
}
})
.catch(function(error) {
console.log(error);
});
}
var wdxxVue = new Vue({
el: "#wdxxVue",
data: {
wdxxs: [],
pageNum: 1,
pageSize: 10,
noMoreData: false,
},
created() {},
methods: {
iKnow: function(cBH, elem) {
var url = BASE_URL_MSG + "update";
var param = {
"cUserId": localStorage.userId,
'cBHList': [cBH],
};
axios.post(url, param)
.then(function(response) {
mui.swipeoutClose(elem.target.parentNode.parentNode);
if (response.data.code == 200) {
successToast("标记消息已读成功");
pulldownRefresh();
} else if (response.data.code == 500) {
errorToast(response.data.msg);
} else {
errorToast("标记消息已读失败");
}
})
.catch(function(err) {
mui.swipeoutClose(elem.target.parentNode.parentNode);
errorToast("标记消息已读失败");
});
},
},
});
```

