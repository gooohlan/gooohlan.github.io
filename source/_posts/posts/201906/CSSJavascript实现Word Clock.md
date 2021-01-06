title: CSS/Javascript实现Word Clock
date: '2019-06-02 20:08:10'
updated: '2019-06-07 23:45:08'
tags: [HTML, CSS, JS]
permalink: /articles/2019/06/02/1559477290334.html
---
![无标题.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/无标题-f88e3fb4.png)
### 无意中看到了一款锁屏效果--world clock，感觉有点意思，就想着用前端来实现他。

### 实现这个效果主要需要实现两个部分

* 时间元素旋转显示
* 时间自动旋转

> 实现元素旋转显示

```
// li 通用样式
li {
        color: #777;
        list-style: none;
        width: 150px;
        text-transform: lowercase;
        transform-origin: 0 center;
        position: absolute;
        left: 0;
        top: 0;
    }
// li 行内样式
li:nth-child(1){ transform: rotate(0deg); }
``````
### 每个li根据实际位置，旋转度数不同，总的加起来360度就可以了
> 时间初始化与自动旋转

* 每一个li去做循环，等于当前日期值的li旋转度数为0，小于的当前日期值为负，大于为正。
```
<!-- 当前月份为6月，所以6月对应的旋转角度为0，小于6月的为负，大于6月为正-->
<!-- 整个旋转度数为360度，-30°与330°效果一致-->
<ul class="month" id="month">
        <li style="transform: rotate(-150deg); color: rgb(119, 119, 119);">一月</li>
        <li style="transform: rotate(-120deg); color: rgb(119, 119, 119);">二月</li>
        <li style="transform: rotate(-90deg); color: rgb(119, 119, 119);">三月</li>
        <li style="transform: rotate(-60deg); color: rgb(119, 119, 119);">四月</li>
        <li style="transform: rotate(-30deg); color: rgb(119, 119, 119);">五月</li>
        <li style="transform: rotate(0deg); color: rgb(255, 255, 255);">六月</li>
        <li style="transform: rotate(30deg); color: rgb(119, 119, 119);">七月</li>
        <li style="transform: rotate(60deg); color: rgb(119, 119, 119);">八月</li>
        <li style="transform: rotate(90deg); color: rgb(119, 119, 119);">九月</li>
        <li style="transform: rotate(120deg); color: rgb(119, 119, 119);">十月</li>
        <li style="transform: rotate(150deg); color: rgb(119, 119, 119);">十一月</li>
        <li style="transform: rotate(180deg); color: rgb(119, 119, 119);">十二月</li>
    </ul>
```
### 源码如下

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Word Clock</title>
    <script src="http://code.jquery.com/jquery-latest.js"></script>
</head>
<style>
    body {
        height: 100%;
        width: 100%;
        background-color: #000;
    }

    div {
        height: 1000px;
        width: 100%;
    }

    ul {
        position: absolute;
        left: 50%;
        top: 500px;
    }

    li {
        color: #777;
        list-style: none;
        width: 150px;
        text-transform: lowercase;
        transform-origin: 0 center;
        position: absolute;
        left: 0;
        top: 0;
    }

    .month > li {
        padding: 0 0 0 80px;
    }

    .day > li {
        padding: 0 0 0 130px;
    }

    .week > li {
        padding: 0 0 0 200px;
    }

    .hour > li {
        padding: 0 0 0 260px;
    }

    .minute > li {
        padding: 0 0 0 325px;
    }

    .second > li {
        padding: 0 0 0 410px;
    }
</style>
<body>
<div>
    <ul class="month" id="month">
        <li style="transform: rotate(-150deg); color: rgb(119, 119, 119);">一月</li>
        <li style="transform: rotate(-120deg); color: rgb(119, 119, 119);">二月</li>
        <li style="transform: rotate(-90deg); color: rgb(119, 119, 119);">三月</li>
        <li style="transform: rotate(-60deg); color: rgb(119, 119, 119);">四月</li>
        <li style="transform: rotate(-30deg); color: rgb(119, 119, 119);">五月</li>
        <li style="transform: rotate(0deg); color: rgb(255, 255, 255);">六月</li>
        <li style="transform: rotate(30deg); color: rgb(119, 119, 119);">七月</li>
        <li style="transform: rotate(60deg); color: rgb(119, 119, 119);">八月</li>
        <li style="transform: rotate(90deg); color: rgb(119, 119, 119);">九月</li>
        <li style="transform: rotate(120deg); color: rgb(119, 119, 119);">十月</li>
        <li style="transform: rotate(150deg); color: rgb(119, 119, 119);">十一月</li>
        <li style="transform: rotate(180deg); color: rgb(119, 119, 119);">十二月</li>
    </ul>
    <ul class="day" id="day">
    </ul>
    <ul class="week" id="week">
    </ul>
    <ul class="hour" id="hour">
    </ul>
    <ul class="minute" id="minute">
    </ul>
    <ul class="second" id="second">
    </ul>
</div>
</body>
<script>
    function mGetDate(year, month) {

        var d = new Date(year, month, 0);
        return d.getDate();
    }

    function numberToChinese(num) {
        var chineseNum = '零一二三四五六七八九十';
        var chinese = '';
        if (num <= 10) {
            return chineseNum.split('')[num]
        } else if (num < 20) {
            return '十' + chineseNum.split('')[num - 10]
        } else {
            if (num % 10 == 0) {
                return chineseNum.split('')[num / 10] + '十'
            } else {
                return chineseNum.split('')[parseInt(num / 10)] + '十' + chineseNum.split('')[parseInt(num % 10)]
            }
        }

    }

    var myDate = new Date();
    var month = document.getElementById('month');
    var day = document.getElementById('day');
    var week = document.getElementById('week');
    var hour = document.getElementById('hour');
    var minute = document.getElementById('minute');
    var second = document.getElementById('second');
    var html = ``;
    // 生成月html
    for (var i = 0; i < 12; i++) {
        html += `<li>` + numberToChinese(i + 1) + `月</li>`
    }
    month.innerHTML = html;

    //  生成日html
    var months = myDate.getMonth() + 1;   //月份从0开始获取，所以需要加1
    var year = myDate.getYear();
    var d = new Date(year, months, 0);
    var days = d.getDate();
    html = ``
    for (i = 0; i < days; i++) {
        html += `<li>` + numberToChinese(i + 1) + `日</li>`
    }
    day.innerHTML = html;

    // 星期html
    html = ``
    for (i = 0; i < 7; i++) {
        if (i == 6) {
            html += `<li>星期日</li>`
        } else {
            html += `<li>星期` + numberToChinese(i + 1) + `</li>`
        }
    }
    week.innerHTML = html;

    // 时html
    html = ``
    for (i = 0; i < 24; i++) {
        html += `<li>` + numberToChinese(i) + `时</li>`
    }
    hour.innerHTML = html;
    //分html
    html = ``
    for (i = 0; i < 60; i++) {
        html += `<li>` + numberToChinese(i) + `分</li>`
    }
    minute.innerHTML = html;
    //秒html
    html = ``
    for (i = 0; i < 60; i++) {
        html += `<li>` + numberToChinese(i) + `秒</li>`
    }
    second.innerHTML = html;


    function setTime() {
        // 获取当前时间
        var myDate = new Date();
        var numArray = [myDate.getMonth() + 1, myDate.getDate(), myDate.getDay(), myDate.getHours() + 1, myDate.getMinutes() + 1, myDate.getSeconds() + 1];// 月,日,星期,时,分,秒

        var l = 6;
        // 遍历ul实现时间赋值与更改
        $("ul").each(function () {
            let length = $(this).children().length
            for (let j = 0; j < length; j++) {
                let deg = "rotate(" + (j + 1 - numArray[6 - l]) * 360 / length + "deg)"
                $(this).find("li").eq(j).css("transform", deg);
                if (j + 1 - numArray[6 - l] == 0 || (l == 4 && j == length - 1 && numArray[6 - l] == 0)) {

                    $(this).find("li").eq(j).css("color", "#fff");
                } else {
                    $(this).find("li").eq(j).css("color", "#777");
                }
            }
            l--;
        })
    }

    setTime()
    window.setInterval("setTime()", 1000);
</script>
</html>
```
### 最后的效果
![image.png](https://cdn.jsdelivr.net/gh/inkdp/CDN@main/img/image-7492af1c.png)

### 写在最后，时间直接使用jq更改很僵硬，可以通过CSS动画实现。