## JavaScript实现[网易云音乐Web站登录窗口]拖拽功能

## 说明

> 你可能发现有很多网站他们的登录窗口或者说是登录框是可以拖动的, 更有甚者他们的站点提示框都可以拖动, 你也许可能会对这个功能的实现感兴趣, 那么这篇文章可能会对你有所帮助！具体的网站示例以 `网易云音乐` Web站点为例，具体效果如下图所示:

![网易云音乐登录窗口](http://oss6u9q3t.bkt.clouddn.com//image/javascript-drafting/163.music.login.dialog.gif)

## JavaScript实现登录窗口的拖拽原理解析

> 预先假设要实现的登录框允许点击鼠标获取拖拽事件的具体位置就是登录框的标题区块也就是下图所示登录区块黑色部分在文章中以 `允许点击鼠标获取拖拽事件区域` 说明问题, 并且假定 `红色十字` 图标就是鼠标状态:

![允许点击鼠标获取拖拽事件的具体位置](http://oss6u9q3t.bkt.clouddn.com//image/javascript-drafting/163.music.login.dialog.02.jpg)

- 当鼠标在 `允许点击鼠标获取拖拽事件区域` 点击鼠标时触发 `onmousedown` 事件

- 获取鼠标在 `允许点击鼠标获取拖拽事件区域` 点击时的具体位置

![点击时的具体位置](http://oss6u9q3t.bkt.clouddn.com//image/javascript-drafting/163.music.login.dialog.03.png)

- 当鼠标移动时改变 `登录窗口` 左上角位置(也是就是坐标点位置)距离页面可视区左上角位置, 那么这个 `登录窗口` 也就移动了, 也就实现了 `登录窗口` 的拖拽功能

	- 当鼠标移动时触发 `onmousemove` 事件

- 当鼠标抬起时, 触发 `onmouseup` 事件
	
	- 释放 `onmousemove` 事件
	- 释放 `onmouseup` 事件自身

> 以上过程就是一个完整的 `登录窗口` 拖拽的过程, 不过要注意以下几点:

1. 拖拽移动 `登录窗口` 时为 `document` 绑定 `onmousemove` 事件, 而不是 `允许点击鼠标获取拖拽事件区域` 
	- 为什么这样做呢? 如果只是为 `允许点击鼠标获取拖拽事件区域` 绑定 `onmousemove` 件事, 当鼠标移动的快了就会导致事件绑定丢失, 不过你可以去验证
	- 下图是将 `onmousemove` 绑定到 `允许点击鼠标获取拖拽事件区域` 这样其是不对的！
	- ![示例](http://oss6u9q3t.bkt.clouddn.com//image/javascript-drafting/163.music.login.dialog.04.gif)
	- 下图是将 `onmousemove` 绑定到 `document` 上的事件, 这样才是最完美的, 因为你不管怎么拖它都不会丢失事件绑定
	- ![示例](http://oss6u9q3t.bkt.clouddn.com//image/javascript-drafting/163.music.login.dialog.07.gif)

2. 抬起鼠标时同样也是为 `document` 绑定 `onmouseup` 事件, 而不是 `允许点击鼠标获取拖拽事件区域`

	-	为什么呢？如果只是为 `允许点击鼠标获取拖拽事件区域` 绑定 `onmouseup` 事件, 你会发现当鼠标移动到脱离文档可视区域时，抬起点击的鼠标按键你会发现当再一次移动鼠标时它依然可以移动这就不符合常理了不是嘛！那就给 `document` 绑定 `onmouseup` 事件时，它就可以很好的解决这个怪异的问题！
	-	下图是将 `onmouseup` 绑定到 `允许点击鼠标获取拖拽事件区域` 这样其是不对的！
	- ![示例](http://oss6u9q3t.bkt.clouddn.com//image/javascript-drafting/163.music.login.dialog.05.gif)
	- 下图是将 `onmouseup` 绑定到 `document` 上的事件, 这样才是最完美的
	- ![示例](http://oss6u9q3t.bkt.clouddn.com//image/javascript-drafting/163.music.login.dialog.06.gif)
	- 不过你会发现拖不回来了, 这是问题也是后面要说的优化问题, 先留一坑，一会填坑时再说！


## JavaScript实现登录窗口的拖拽效果

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>JS拖拽</title>
    <style>
        /* public style start */
        * { margin: 0; padding: 0; }
        body, textarea, select, input, button {font-size: 12px;color: white;font-family: Arial, Helvetica, sans-serif;-webkit-text-size-adjust: none;}
        .fl-l { float: left }
        .fl-r { float: right }
        .clearfix:after {visibility:hidden;display:block;font-size:0;content:'.';clear:both;height:0;}
        .clearfix {*zoom:1;}
        /* public style end */

        /* dialog window style start */

        /* dialog window main */
        #dialog {
            height: 200px;
            width: 400px;
            background-color: #EAF6DB;
            border: 1px solid lightslategray;
            position: absolute;
            top: 200px;
            left: 400px;
        }

        /* dialog window title section */
        .dialog-title {
            height: 40px;
            line-height: 40px;
            background-color: #3a3333;
            cursor: move;
        }

        /* dialog window title inner detail */
        .login, .close {
            display: inline-block;
            margin: 0 15px;
        }
    </style>
    <script>
        window.onload = function () {
            // 获取DOM元素
            var oDialog_title = document.getElementById( 'dialog-title' );  // 允许点击鼠标获取拖拽事件区域
            var oDialog = oDialog_title.parentNode; // 登录窗口

            // 初始化鼠标默认位置
	        var iDisX = 0;
	        var iDisY = 0;

            // 为获取到的DOM元素添加鼠标按下事件 `onmousedown`
	        oDialog_title.onmousedown = function ( ent ) {
                // 保存鼠标事件对象
            	var oEvent = ent || event;

            	// 距离计算鼠标位于弹出框内的位置
	            iDisX = oEvent.clientX - oDialog.offsetLeft;    // 鼠标X轴位置 - 弹出框X外左边距
	            iDisY = oEvent.clientY - oDialog.offsetTop;     // 鼠标Y轴位置 - 弹出框Y外上边距

                // 点击弹出框后拖动鼠标, 移动弹出框
	            document.onmousemove = function ( ent ) {
		            // 保存鼠标事件对象
		            var oEvent = ent || event;

		            // 当鼠标移动时改变弹出框的位置
		            oDialog.style.left = oEvent.clientX - iDisX + 'px';
		            oDialog.style.top = oEvent.clientY - iDisY + 'px';
	            };

	            // 当点击鼠标拖动弹出框, 抬起鼠标时
	            document.onmouseup = function () {
	            	// 清除弹出框的移动事件及本身
		            document.onmousemove = null;
		            document.onmouseup = null;
	            };

	            // 阻止默认事件, 如果不加这个阻止默认事件, 在firefox下会有一个获取焦点的光标一直在闪动, 在3.0及以下会出现拖动出现重影的情况
                return false;
            };


        };
    </script>
</head>
<body>
    <!-- 假设这个DIV就是一个登录窗口 -->
    <div id="dialog">
        <div id="dialog-title" class="dialog-title clearfix">
            <span class="fl-l login">登录</span>
            <span class="fl-r close">X</span>
        </div>
    </div>
</body>
</html>
```

> 以上就是以一个简单的DIV模拟 `登录窗口` 实现的一个简单拖拽过程

## JavaScript实现登录窗口的拖拽具体效果

![JavaScript实现登录窗口的拖拽具体效果](http://oss6u9q3t.bkt.clouddn.com//image/javascript-drafting/163.music.login.dialog.08.gif)

> 你可能已经发现了这个 `登录窗口` 与 `网易云音乐` 最大的区别在于它可以拖拽出文档可视区域, 它甚至可以拖拽到文档不可见区域, 那就永远拖不回了, 就像那已分手并结婚了的前女友永远也回不来一样

![就像那已分手并结婚了的前女友永远也回不来一样](http://oss6u9q3t.bkt.clouddn.com//image/javascript-drafting/163.music.login.dialog.09.modify.gif)

> (好了, 找一没人的地哭晕在厕所好了 :sob: 是不是同时又想起来那几句话[得不到的永远在骚动, 失去的永远在怀念, 身边的永远成为风景, ......]), 以上是逗逼时刻就当没发生一样好了 :joy:

> 其实这也是上面留的一个坑, 现在来优化填坑, 就是实现和 `网易云音乐` 网站 `登录窗口` 一样的效果, 禁止 `登录窗口` 拖拽出文档可视区以外! 下面是填坑时刻, 非战斗人员请火速离开现场  :joy:

## JavaScript实现登录窗口的拖拽优化填坑

> 其实思路很简单就是当 `登录窗口` 的四个边和文档窗口的其一边界重合时, 就让 `登录窗口` 的那一个边的外边距值与重合的文档那一个边的值相等, 那这个事情就妥妥的搞定了!

- 只需要修改 `documnet.onmousemove` 事件方法 `登录窗口` 当前位置即可!

```console
// 当鼠标移动时改变弹出框的位置
oDialog.style.left = oEvent.clientX - iDisX + 'px';
oDialog.style.top = oEvent.clientY - iDisY + 'px';
```

- 改写如下:

```console
// 优化填坑 - 禁止 `登录窗口` 拖拽出文档可视区域, 保存 `登录窗口` 在文档中具体位置
var iCurrentDialogDisLift = oEvent.clientX - iDisX; // `登录窗口` 当前位置于X轴具体值
var iCurrentDialogDisTop = oEvent.clientY - iDisY;  // `登录窗口` 当前位置于Y轴具体值

// 检测当前 `登录窗口` X轴是否位于文档可视区域最左侧或最右侧
if ( iCurrentDialogDisLift < 0 ) {
    iCurrentDialogDisLift = 0;
} else if ( iCurrentDialogDisLift > document.documentElement.clientWidth - oDialog.offsetWidth  ) {
    // 当前文档X轴可视区域大小包括左右边框线宽度 - `登录窗口` X轴区域大小包括左右边框线宽度
    iCurrentDialogDisLift = document.documentElement.clientWidth - oDialog.offsetWidth;
}

// 检测当前 `登录窗口` Y轴是否位于文档可视区域最上端或最下端
if ( iCurrentDialogDisTop < 0 ) {
    iCurrentDialogDisTop = 0;
} else if ( iCurrentDialogDisTop > document.documentElement.clientHeight - oDialog.offsetHeight ) {
    // 当前文档Y轴可视区域大小包括上下边框线宽度 - `登录窗口` Y轴区域大小包括上下边框线宽度
    iCurrentDialogDisTop = document.documentElement.clientHeight - oDialog.offsetHeight;
}

// 当鼠标移动时改变弹出框的位置
oDialog.style.left = iCurrentDialogDisLift + 'px';
oDialog.style.top = iCurrentDialogDisTop + 'px';
```
## JavaScript实现登录窗口的拖拽优化填坑全文档

```console
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>JS拖拽</title>
    <style>
        /* public style start */
        * { margin: 0; padding: 0; }
        body, textarea, select, input, button {font-size: 12px;color: white;font-family: Arial, Helvetica, sans-serif;-webkit-text-size-adjust: none;}
        .fl-l { float: left }
        .fl-r { float: right }
        .clearfix:after {visibility:hidden;display:block;font-size:0;content:'.';clear:both;height:0;}
        .clearfix {*zoom:1;}
        /* public style end */

        /* dialog window style start */

        /* dialog window main */
        #dialog {
            height: 200px;
            width: 400px;
            background-color: #EAF6DB;
            border: 1px solid lightslategray;
            position: absolute;
            top: 200px;
            left: 400px;
        }

        /* dialog window title section */
        .dialog-title {
            height: 40px;
            line-height: 40px;
            background-color: #3a3333;
            cursor: move;
        }

        /* dialog window title inner detail */
        .login, .close {
            display: inline-block;
            margin: 0 15px;
        }
    </style>
    <script>
        window.onload = function () {
            // 获取DOM元素
            var oDialog_title = document.getElementById( 'dialog-title' );  // 允许点击鼠标获取拖拽事件区域
            var oDialog = oDialog_title.parentNode; // 登录窗口

            // 初始化鼠标默认位置
	        var iDisX = 0;
	        var iDisY = 0;

            // 为获取到的DOM元素添加鼠标按下事件 `onmousedown`
	        oDialog_title.onmousedown = function ( ent ) {
                // 保存鼠标事件对象
            	var oEvent = ent || event;

            	// 距离计算鼠标位于弹出框内的位置
	            iDisX = oEvent.clientX - oDialog.offsetLeft;    // 鼠标X轴位置 - 弹出框X外左边距
	            iDisY = oEvent.clientY - oDialog.offsetTop;     // 鼠标Y轴位置 - 弹出框Y外上边距

                // 点击弹出框后拖动鼠标, 移动弹出框
	            document.onmousemove = function ( ent ) {
		            // 保存鼠标事件对象
		            var oEvent = ent || event;

                    // 优化填坑 - 禁止 `登录窗口` 拖拽出文档可视区域, 保存 `登录窗口` 在文档中具体位置
                    var iCurrentDialogDisLift = oEvent.clientX - iDisX; // `登录窗口` 当前位置于X轴具体值
                    var iCurrentDialogDisTop = oEvent.clientY - iDisY;  // `登录窗口` 当前位置于Y轴具体值

                    // 检测当前 `登录窗口` X轴是否位于文档可视区域最左侧或最右侧
                    if ( iCurrentDialogDisLift < 0 ) {
                        iCurrentDialogDisLift = 0;
                    } else if ( iCurrentDialogDisLift > document.documentElement.clientWidth - oDialog.offsetWidth  ) {
                        // 当前文档X轴可视区域大小包括左右边框线宽度 - `登录窗口` X轴区域大小包括左右边框线宽度
                        iCurrentDialogDisLift = document.documentElement.clientWidth - oDialog.offsetWidth;
                    }

                    // 检测当前 `登录窗口` Y轴是否位于文档可视区域最上端或最下端
                    if ( iCurrentDialogDisTop < 0 ) {
                        iCurrentDialogDisTop = 0;
                    } else if ( iCurrentDialogDisTop > document.documentElement.clientHeight - oDialog.offsetHeight ) {
                        // 当前文档Y轴可视区域大小包括上下边框线宽度 - `登录窗口` Y轴区域大小包括上下边框线宽度
                        iCurrentDialogDisTop = document.documentElement.clientHeight - oDialog.offsetHeight;
                    }

                    // 当鼠标移动时改变弹出框的位置
                    oDialog.style.left = iCurrentDialogDisLift + 'px';
                    oDialog.style.top = iCurrentDialogDisTop + 'px';
	            };

	            // 当点击鼠标拖动弹出框, 抬起鼠标时
	            document.onmouseup = function () {
	            	// 清除弹出框的移动事件及本身
		            document.onmousemove = null;
		            document.onmouseup = null;
	            };

	            // 阻止默认事件, 如果不加这个阻止默认事件, 在firefox下会有一个获取焦点的光标一直在闪动, 在3.0及以下会出现拖动出现重影的情况
                return false;
            };
        };
    </script>
</head>
<body>
    <!-- 假设这个DIV就是一个登录窗口 -->
    <div id="dialog">
        <div id="dialog-title" class="dialog-title clearfix">
            <span class="fl-l login">登录</span>
            <span class="fl-r close">X</span>
        </div>
    </div>
</body>
</html>
```

## JavaScript实现登录窗口的拖拽优化填坑后具体效果

![JavaScript实现登录窗口的拖拽优化填坑后具体效果](http://oss6u9q3t.bkt.clouddn.com//image/javascript-drafting/163.music.login.dialog.10.gif)

> 以上就是实现与 `网易云音乐` Web站 `登录窗口`  拖拽效果一致的具体过程

> 希望本文对你的工作和学习有所帮助

> 如果觉得还不错怎么感谢我呢？ 妈呀! 点赞啊!

> Good Luck! from warnerwu at 2017.08.19 AM