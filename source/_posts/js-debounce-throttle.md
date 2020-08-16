---
title: js防抖节流
date: 2020-08-13 02:27:08
tags: Javascript
categories: Javascript
cover: /img/debounce.jpg
---

# 防抖

#### 触发事件后必须等待一个delay才能执行，频繁触发只会重置等待时间。



![](debounce.gif)

#### html

```html
<div>
    <input id="input" type="text">
    <p id="content"></p>
</div>
```

#### js

```javascript
    let input = document.getElementById('input');
    let content = document.getElementById('content');

	//将this绑定到input上，并传递参数
    input.onkeyup= debounce(show, 1000).bind(input,',hello',',world');

    function show(arg1,arg2) {
        content.innerHTML+=this.value+arg1+arg2+'<br/>';
    }

    function debounce(func,delay) {
        let timer;
        return function () {
            if(timer)
                clearTimeout(timer);
            timer= setTimeout(()=>{
                func.call(this,...arguments);
            },delay);
        }
    }
```



# 节流

#### 允许你频繁触发事件，但只会按设定的节奏（delay周期）来执行事件。



![](throttle.gif)

```javascript
    let input = document.getElementById('input');
    let content = document.getElementById('content');

    input.onkeypress= throttle(show,1000).bind(input,'world');  //注意，这里 keypress 事件才支持一直按

    function show(arg) {
        content.innerHTML+="hello,"+arg+'<br/>';
    }

    function throttle(func,delay) {
        let timer = null;
        return function () {
            if(!timer){            //必须确保上一次定时器执行完毕
                timer = setTimeout(()=>{
                    func.call(this,...arguments);
                    timer=null;    //及时清理，表示执行完毕，clearTimeout后timer仍有值！！！画重点！！！
                },delay)
            }
        }
    }
```

