---
title: '[JS]利用echart.js繪圖'
date: 2021-02-15 20:42:12
tags: [Javacscript, echart]
categories: 資料視覺化
---

## echart

### 基本運用
echart是一個用js繪圖的框架，[官方文檔](https://echarts.apache.org/zh/feature.html)。

引入echart後，首先必須準備一個畫圖的區塊。

<!-- more -->

```html
<div class="row">
    <div id="month_stat" style="width:50%; height:400px;" class="col"></div>
    <div id="month_stat_2" style="width:50%; height:400px;" class="col"></div>
</div>
```
我會繪製兩個圖表，一個負責AVG、OBP、SLG，另一個負責PA、AB、H、RBI、HR，並且利用Bootstrap將兩個圖表排在一列，會使用長條圖來表示。

echart官網有即時顯示的[編輯器](https://echarts.apache.org/examples/zh/editor.html?c=bar-simple)可用。

`官方範例`
```javascript
option = {
    xAxis: {
        type: 'category',
        data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    },
    yAxis: {
        type: 'value'
    },
    series: [{
        data: [120, 200, 150, 80, 70, 110, 130],
        type: 'bar'
    }]
};
```
xAxis為x軸的設置，其中type為x軸的屬性，可以選擇`value`、`category`、`time`、`log`四種，x軸我預定顯示數值種類，所以選擇`category`，而y軸為數值。

如果要繪製橫向的長條圖，只要把xAxis和yAxis的內容對調就好，series中的data會自動對應。

基本的長條圖介紹玩了，我想要繪製的長條圖x軸為月份，y軸為數值，並且每個月分都有每種數據，需要用到[官方範例](https://echarts.apache.org/examples/zh/editor.html?c=bar-stack)。

跟上一個不同的是，這個範例有著標籤可以控制顯示，需要更改series裡面的內容，並且加入legend屬性。

```
series: [{
        name: 'AVG',
        type: 'bar',
        data: result['AVG']
    },{
        name: 'OBP',
        type: 'bar',
        data: result['OBP']
    },{
        name: 'SLG',
        type: 'bar',
        data: result['SLG']
    }]
```
> result為ajax跟Falsk取的資料
> 

三種資料有三種name，則我必須在legend裡面宣告這三種name。

```
legend: {
        data: ['AVG', 'SLG', 'OBP']
    }
```

### 增加圖表功能

#### toolbox
echart有提供許多API來讓我們使用，在[官方文檔](https://echarts.apache.org/zh/api.html#echarts)中可以查到許多有用的API。

今天我主要回會用toolbox這個配置項的功能，toolbox可以為圖表增添工具欄，有輸出圖片、數據圖、縮放等。

首先要啟用toolbox要先設置`show`為true，並且可以在`feature`中設置各選項的內容。

```javascript
 toolbox: {
            show: true,
            orient: 'vertical',
            left: 'right',
            top: 'center',
            feature: {
                magicType: { show: true, type: ['line', 'bar', 'stack', 'tiled'] },
                restore: { show: true },
                saveAsImage: { show: true }
            }
        }
```
> magicType可以讓長條圖轉成折線圖、堆疊圖等
> 

#### dataZoom

接著我想要增加縮放月份的功能，這邊用到的是`dataZoom`，`dataZoom`的區域縮放功能可以選取想要的區域數值。

`dataZoom`有inside和slider兩種，在type屬性中宣告(預設是slider)，inside是內置的，可以使用滑鼠的滾輪操作，手機可以用兩指放大，而slider則是外面有縮放條可以選取。

`slider`
![](https://i.imgur.com/zyRNbh4.png)


```javascript
dataZoom: [ 
            {
                show: true,
                realtime: true,
                start: 0,
                end: 100
            }
        ],
```


#### tooltip

現在滑鼠移到圖表上會有顯示數據，但是都是每一條獨立觸發，我希望當我的滑鼠移到某個月分時，可以將三個數據都顯示出來，這時要用到`tooltip`的功能

還沒有tooltip的提示
![](https://i.imgur.com/VnNTtBa.png)

tooltip有兩種觸發條件：item和axis，item主要用在散點圖，透過點觸發，而axis用在長條圖，透過軸觸發。

```javascript
tooltip: {
            trigger: 'axis'
        },
```

![](https://i.imgur.com/KQRrNU3.png)


完整程式碼
```javascript
function get_month(){
    let p = $("#player").val();
    let chart = echarts.init(document.getElementById('month_stat'), 'white', { renderer: 'canvas' });
    let chart2 = echarts.init(document.getElementById('month_stat_2'), 'white', { renderer: 'canvas' });

    if(p!=='選擇球員'){
        $.ajax({
        type: 'POST',
        url: "/getchartinfo",
        data: {"p":p},
        success: function (result) {
            let option = {
                title: {
                    text: result['name']
                },
                legend: {
                    data: ['AVG', 'SLG', 'OBP']
                },
                toolbox: {
                    show: true,
                    orient: 'vertical',
                    left: 'right',
                    top: 'center',
                    feature: {
                        magicType: { show: true, type: ['line', 'bar', 'stack', 'tiled'] },
                        restore: { show: true },
                        saveAsImage: { show: true }
                    }
                },
                dataZoom: [ 
                    {
                        show: true,
                        realtime: true,
                        start: 0,
                        end: 100
                    }
                ],
                tooltip: {
                    trigger: 'axis'
                },
                xAxis: {
                    data: result['attr']
                },
                yAxis: {},
                series: [{
                    name: 'AVG',
                    type: 'bar',
                    data: result['AVG']
                },{
                    name: 'OBP',
                    type: 'bar',
                    data: result['OBP']
                },{
                    name: 'SLG',
                    type: 'bar',
                    data: result['SLG']
                }]
            };
            chart.setOption(option);

            let option_2 = {
                title: {
                    text: '其他數據'
                },
                tooltip: {},
                legend: {
                    data: ['PA', 'AB', 'RBI', 'H', 'HR']
                },
                toolbox: {
                    show: true,
                    orient: 'vertical',
                    left: 'right',
                    top: 'center',
                    feature: {
                        mark: { show: true },
                        dataView: { show: true, readOnly: false },
                        magicType: { show: true, type: ['line', 'bar', 'stack', 'tiled'] },
                        restore: { show: true },
                        saveAsImage: { show: true }
                    }
                },
                dataZoom: [
                    {
                        show: true,
                        realtime: true,
                        start: 0,
                        end: 100
                    }
                ],
                tooltip: {
                    trigger: 'axis'
                },
                xAxis: {
                    data: result['attr']
                },
                yAxis: {},
                series: [{
                    name: 'PA',
                    type: 'bar',
                    data: result['PA']
                }, {
                    name: 'AB',
                    type: 'bar',
                    data: result['AB']
                }, {
                    name: 'H',
                    type: 'bar',
                    data: result['H']
                }, {
                    name: 'RBI',
                    type: 'bar',
                    data: result['RBI']
                }, {
                    name: 'HR',
                    type: 'bar',
                    data: result['HR']
                }]
            };
            chart2.setOption(option_2);
        }
    });
    }
}
```