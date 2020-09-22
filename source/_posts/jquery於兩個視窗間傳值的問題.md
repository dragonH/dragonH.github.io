---
title: jquery於兩個視窗間傳值的問題
date: 2020-09-22 14:33:00
categories:
    - web
tags:
    - ithelp
    - js
    - javascript
    - browser
    - postMessage
---

# 情境問題


請教各位前輩，ajax有没有可能做到以下的傳值情況：

（一）先於瀏覽器開啟兩個視窗，一個執行A.php，一個執行B.php

（二）於A.php click一個按鈕後，利用ajax post一個json值給B.php

（三）B.php在不重新執行的情況下，能接收此JSON值，並於網頁上呈現


# 解決方法

單純看兩個視窗互傳值的話

可以用 [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage#Syntax)

不過有一些限制就是

不是隨便兩個視窗都能互傳

給個範例

`a.html`
```html
<!DOCTYPE html>
<html>
  <head>
    <title>A</title>
  </head>
  <body>
    <div class = "box">
      <button type = "button" id = "newWindow">開新視窗</button>
      <textarea id = "toSay"></textarea>
      <button id = "sendText">送出文字</button>
      <button id = "sendJson">送出json</button>
    </div>
  </body>
  <script>
    const newWindowBtn = document.getElementById('newWindow');
    const sendTextBtn = document.getElementById('sendText');
    const sendJsonBtn = document.getElementById('sendJson');
    let targetWindow = null;
    newWindowBtn.addEventListener('click', () => {
      targetWindow = open('./b.html');
    });
    sendTextBtn.addEventListener('click', () => {
      const toSayText = document.getElementById('toSay').value;
      if (targetWindow) {
        targetWindow.postMessage(toSayText, '*');
      } else {
        alert('請先開新視窗');
      }
    });
    sendJsonBtn.addEventListener('click', () => {
      if (targetWindow) {
        targetWindow.postMessage({ name1: 'apple', name2: 'banana' }, '*'); 
        // '*' 代表允許所有targetOrigin
      } else {
        alert('請先開新視窗');
      }
    });    
  </script>
</html>
<style>
  .box{
    display: flex;
    justify-content: center;
    justify-items: center;
  }
</style>
```

`b.html`
```html
<!DOCTYPE html>
<html>
  <head>
    <title>B</title>
  </head>
  <script>
    window.addEventListener('message', (msg) => {
      let node = document.createElement('p');
      node.textContent = `${new Date()} : ${JSON.stringify(msg.data)}`;
      document.getElementById('receiveDiv').append(node)
    });
  </script>
  <body>
    <div id = "receiveDiv"></div>
  </body>
</html>
```

`result`

{% asset_img jquery於兩個視窗間傳值的問題-result.gif %}

透過 a.html 開啟 b.html

就能從 a 傳到 b

也能 b 傳到 a (用 window.opener.postMessage())

如果b.php 是想要取得a.php執行更新後的資料的話

大部分的情況應該還是用websocket等那些方法