# PHASER游戏框架与HTML DOM元素之间的通信交互

本想按照PHASER的HTML Dom元素官方实例：[http://labs.phaser.io/index.html?dir=game%20objects/dom%20element/&q=](http://labs.phaser.io/index.html?dir=game objects/dom element/&q=) Canvas来创建HTML DOM元素，但this.add.dom 一直提示错误，无奈直接用HTML5的语法来创建DOM元素，然后在Phaser内获取该DOM元素，也不用再使用第三方的Phaser Html Input插件Plugin，还是挺方便快捷的。

按照这个思路，还可以创建listView,ScrollView等一系列HTML Dom元素，然后再在Phaser内操作其对应的div ID就可以了，而至于BUTTON元素则用addEventListerner监听即可。

HTML代码

```html
<div id='gameBetZone' class="gameBetZone">
    <div class="row rowPadding" id="rowBet">
        <div class="col-xs-3">数量：</div>
        <div class="col-xs-6"><input id="amount" type="number" value="10"></div>
        <div class="col-xs-3"><button id="betButton">提交</button></div>
    </div>
    <div class="row" id="rowSearching">
        <div class="col-xs-4"></div>
        <div class="col-xs-4">
            <div class="pendingTxt">搜索中...</div>
        </div>
        <div class="col-xs-4"></div>
    </div>
</div>
```

PHASER 代码

```js
 //MARK:-- create element
    createBetElement: function () {
        document.getElementById('gameBetZone').style.display = 'none'; // block
        document.getElementById('rowBet').style.display = 'block';
        document.getElementById('rowSearching').style.display = 'none';
    }

// 取得输入框 amount的值
this.amountEle = document.getElementById('amount');
this.betButton = document.getElementById('betButton');
me.betButton.addEventListener('click', myClickEvent, false);
function myClickEvent() {
           
            // REMOVE EXISTING BUTTON EVENT;
            me.betButton.removeEventListener('click', myClickEvent, false);
            //TODO:sending this value to server;
            console.log('me.amountEle.value$', me.amountEle.value);
          
}
```