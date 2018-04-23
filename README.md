# 完美运动框架的实现思路

------

> 运动，其实就是在一段时间内改变left、right、width、height、opactiy的值，到达目的地之后停止。

#### 现在按照以下步骤来进行我们的运动框架的封装:

    1、匀速运动。

    2、缓冲运动。

    3、多物体运动。

    4、任意值变化。

    5、链式运动。

    6、同时运动。

## （一） 匀速运动

### 运动基础：如何让div动起来,如下：

1. 设置元素为绝对定位，只有绝对定位后，left,top等值才生效。
2. 定时器的使用（动态改变值），这里使用setInterval()每隔指定的时间执行代码：

        计时器 setInterval(函数,交互时间(毫秒))：在执行时,从载入页面后每隔指定的时间执行代码。

        取消计时器 clearInterval(函数) 方法可取消由 setInterval() 设置的交互时间。
        
3. 获取当前的位置，大小等等。offsetLeft（当前元素相对父元素位置）。
4. 速度--物体运动的快慢因素【定时器间隔时间、改变值的大小】。
 
根据上面的信息我们就可以开始封装运动框架创建一个变化的div了。

``` javascript
/**
 * 运动框架-1-动起来
 * @param {HTMLElement} element 进行运动的节点
 */
var timer = null;
function startMove(element) {
    timer = setInterval(function () {//定时器
        element.style.left = element.offsetLeft + 5 + "px";
    }, 30);
}
```

你没看错，就是那么简单。

但是等等， what？ 怎么不会停？WTF？

那是因为我们没有运动终止条件。好再还是比较简单。直接在定时器内部，判断到达目标值，清除定时器就行拉！

``` javascript
/**
 * 运动框架-2-运动终止
 * @param {HTMLElement} element  进行运动的节点
 * @param {number}      iTarget  运动终止条件。
 */
var timer = null;
function startMove(element, iTarget) {
    timer = setInterval(function () {
        element.style.left = element.offsetLeft + 5 + "px";
        if (element.offsetLeft === iTarget) {//停止条件
            clearInterval(timer);
        }
    }, 30);
}
```

就这样是不是就完成了呢？已经ok了呢？ no~no~no~ 还有一些Bug需要处理。

#### 运动中的BUG

    1.速度取到某些值会无法停止

    2.到达位置后再点击还会运动

    3.重复点击速度加快

    4.速度无法更改

#### 解决BUG

    1.速度取到某些值会无法停止（这个Bug稍后解决，在进化过程中自然解决）

    2.把运动和停止隔开(if/else)

    3.在开始运动时,关闭已有定时器

    4.把速度用变量保存
    
``` javascript
/**
 * 运动框架-3-解决Bug
 */
var timer = null;
function startMove(element, iTarget) {
    clearInterval(timer);//在开始运动时,关闭已有定时器
    timer = setInterval(function () {
        var iSpeed = 5;//把速度用变量保存
        //把运动和停止隔开(if/else)
        if (element.offsetLeft === iTarget) {//结束运动
            clearInterval(timer);
        } else {
            element.style.left = element.offsetLeft + iSpeed + "px";
        }
    }, 30);
}
```

这样一个简单的运动框架就完成了。但是，再等等。只能向右走？别急，我们不是定义了把速度变成为了变量吗？只需要对它进行一些处理就行啦！

> var iSpeed = 5;

``` javascript
//判断距离目标位置，达到自动变化速度正负
var iSpeed = 0;
if (element.offsetLeft < iTarget) {
    iSpeed = 5;
} else {
    iSpeed = -5;
}
```

### 透明度动画

    1.用变量alpha储存当前透明度。

    2.把上面的element.offsetLeft改成变量alpha。

    3.运动和停止条件部分进行更改。如下：
    
``` javascript
//透明度浏览器兼容实现
if (alpha === iTarget) {
    clearInterval(time);
} else {
    alpha += speed;
    element.style.filter = 'alpha(opacity:' + alpha + ')'; //兼容IE
    element.style.opacity = alpha / 100;//标准
}
```

## （二） 缓冲动画

### 思考：怎么样才是缓冲动画？

    应该有以下几点：

        1.逐渐变慢,最后停止

        2.距离越远速度越大

            速度由距离决定

            速度=(目标值-当前值)/缩放系数

        3.Bug :速度取整(使用Math方法)，不然会闪

            向上取整。Math.ceil(iSpeed)

            向下取整。Math.floor(iSpeed)

还是对速度作文章：

``` javascript
/**
 * 运动框架-4-缓冲动画
 */
function startMove(element, iTarget) {
    clearInterval(timer);
    timer = setInterval(function () {
    //因为速度要动态改变，所以必须放在定时器中
        var iSpeed = (iTarget - element.offsetLeft) / 10; //(目标值-当前值)/缩放系数=速度
        iSpeed = iSpeed > 0 ? Math.ceil(iSpeed) : Math.floor(iSpeed); //速度取整
        if (element.offsetLeft === iTarget) {//结束运动
            clearInterval(timer);
        } else {
            element.style.left = element.offsetLeft + iSpeed + "px";
        }
    }, 30);
}
```

做到这里，（速度取到某些值会无法停止)这个Bug就自动解决啦！

#### 例子:缓冲菜单【跟随页面滚动的缓冲侧边栏】     [演示地址](https://codepen.io/guowen921/pen/rxLOMd%20)


> 潜在问题：目标值不是整数的时候

## （三） 多物体运动

### 思考：如何实现多物体运动？

    单定时器，存在问题。每个div一个定时器

    定时器作为对象的属性

        直接使用element.timer把定时器变成对象上的一个属性。

    参数的传递:物体/目标值

        比较简单把上面框架的进行如下更改：timer-->element.timer


就这样就行啦！

## （四） 任意值变化

咳咳。我们来给div加个1px的边框。boder :1px solid #000

然后来试试下面的代码


``` javascript
/**
 * 运动框架-4-缓冲动画
 */
setInterval(function () {
    oDiv.style.width = oDiv.offsetWidth - 1 + "px";
}, 30) 
```

嗯，神奇的事情发生了！what？设置的不是宽度在减吗？怎么尼玛增加了！ 不对啊，大兄弟。

究竟哪里出了问题呢？

一起找找资料，看看文档，原来offset这一系列的属性都会存在，被其他属性干扰的问题。

好吧，既然不能用，那么就顺便把任意值变化给做了吧。

### 第一步：获取实际样式

> 使用offsetLeft..等获取样式时, 若设置了边框, padding, 等可以改变元素宽度高度的属性时会出现BUG..

通过查找发现element.currentStyle(attr)可以获取计算过之后的属性。

但是因为兼容性的问题，需封装getStyle函数。（万恶的IE）

当然配合CSS的box-sizing属性设为border-box可以达到一样的效果 ? (自认为，未验证)。

``` javascript
/**
 * 获取实际样式函数
 * @param   {HTMLElement}   element  需要寻找的样式的html节点
 * @param   {String]}       attr     在对象中寻找的样式属性
 * @returns {String}                 获取到的属性
 */
function getStyle(element, attr) {
    //IE写法
    if (element.currentStyle) {
        return element.currentStyle[attr];
    //标准
    } else {
        return getComputedStyle(element, false)[attr];
    }
}
```
### 第二步：改造原函数

1.添加参数，attr表示需要改变的属性值。

2.更改element.offsetLeft为getStyle(element, attr)。

>需要注意的是：getStyle(element, attr)不能直接使用，因为它获取到的字符串,例：10px。
> 变量iCurrent使用parseInt(),将样式转成数字。
    
3.element.style.left为element.style[attr]。

``` javascript
/**
 * 运动框架-4-任意值变化
 * @param {HTMLElement} element 运动对象
 * @param {string}      attr    需要改变的属性。
 * @param {number}      iTarget 目标值
 */
function startMove(element, attr, iTarget) {
    clearInterval(element.timer);
    element.timer = setInterval(function () {
    //因为速度要动态改变，所以必须放在定时器中
    var iCurrent=0;
    iCurrent = parseInt(getStyle(element, attr));       //实际样式大小
        var iSpeed = (iTarget - iCurrent) / 10;         //(目标值-当前值)/缩放系数=速度
        iSpeed = iSpeed > 0 ? Math.ceil(iSpeed) : Math.floor(iSpeed); //速度取整
        if (iCurrent === iTarget) {//结束运动
            clearInterval(element.timer);
        } else {
            element.style[attr] = iCurrent + iSpeed + "px";
        }
    }, 30);
}
```

试一试，这样是不是就可以了呢？

> 还记得上面我们写的透明度变化吗? 再试试

果然还是不行， （废话，你见过透明度有"px"单位的么? - -白眼 ）

### 第三步：透明度兼容处理

#### 思考：需要对那些属性进行修改？

    1.判断attr是不是透明度属性opacity 。

    2.对于速度进行处理。

        · 为透明度时，由于获取到的透明度会是小数，所以需要 * 100

        · 并且由于计算机储存浮点数的问题，还需要将小数，进行四舍五入为整数。使用：Math.round(parseFloat(getStyle(element, attr)) * 100)。
        
        · 否则，继续使用默认的速度。

    3.对结果输出部分进行更改。

        · 判断是透明度属性，使用透明度方法

        · 否则，使用使用默认的输出格式。

``` javascript
/**
 * 运动框架-5-兼容透明度
 * @param {HTMLElement} element 运动对象
 * @param {string}      attr    需要改变的属性。
 * @param {number}      iTarget 目标值
 */
function startMove(element, attr, iTarget) {
    clearInterval(element.timer);
    element.timer = setInterval(function () {
        //因为速度要动态改变，所以必须放在定时器中
        var iCurrent = 0;
        if (attr === "opacity") { //为透明度时执行。
            iCurrent = Math.round(parseFloat(getStyle(element, attr)) * 100);
        } else { //默认情况
            iCurrent = parseInt(getStyle(element, attr)); //实际样式大小
        }
        var iSpeed = (iTarget - iCurrent) / 10; //(目标值-当前值)/缩放系数=速度
        iSpeed = iSpeed > 0 ? Math.ceil(iSpeed) : Math.floor(iSpeed); //速度取整
        if (iCurrent === iTarget) {//结束运动
            clearInterval(element.timer);
        } else {
            if (attr === "opacity") { //为透明度时，执行
                element.style.filter = "alpha(opacity:" + (iCurrent + iSpeed) + ")"; //IE
                element.style.opacity = (iCurrent + iSpeed) / 100; //标准
            } else { //默认
                element.style[attr] = iCurrent + iSpeed + "px";
            }
        }
    }, 30);
}
```

到这里，这个运动框架就基本上完成了。但是，我们是追求完美的不是吗？

继续进化！

## （五）链式动画

> 链式动画：顾名思义，就是在该次运动停止时，开始下一次运动。

### 如何实现呢？

    · 使用回调函数：运动停止时,执行函数

    · 添加func形参（回调函数）。

    · 在当前属性到达目的地时iCurrent === iTarget，判断是否有回调函数存在，有则执行。 
  
``` javascript
if (iCurrent === iTarget) {//结束运动
    clearInterval(element.timer);
    if (func) {
        func();//回调函数
    }
}
```
good，链式动画完成！距离完美还差一步！

## （六）同时运动

### 思考：如何实现同时运动？

    应该有以下几点：

        1.使用JSON传递多个值。
        
        2.使用for in循环，遍历属性，与值。
        
        3.定时器问题!(运动提前停止)
        
            在循环外设置变量,假设所有的值都到达了目的值为true
            
            在循环中检测是否到达目标值,若没有值未到则为false
            
            在循环结束后,检测是否全部达到目标值.是则清除定时器
            
     实现：
     
        1.删除attr与iTarget两个形参，改为json

        2.在函数开始时，设置一个标记var flag = true; //假设所有运动到达终点.

        3.在定时器内使用for in，遍历属性与目标，改写原来的attr与iTarget，为json的属性与值

        4.修改运动终止条件，只有每一项的实际属性值iCurrent，等于目标值json[attr]时，flag才为true。清除定时器，判断是否回调。

        5.否则，继续执行代码，直到所有属性值等于目标值。


## 完美运动框架

``` javascript
/**
 * 完美运动框架
 * @param {HTMLElement} element 运动对象
 * @param {JSON}        json    属性：目标值      
 *   @property {String} attr    属性值
 *   @config   {Number} target  目标值
 * @param {function}    func    可选，回调函数，链式动画。
 */
function startMove(element, json, func) {
    var flag = true; //假设所有运动到达终点.
    clearInterval(element.timer);
    element.timer = setInterval(function () {
        for (var attr in json) {
            //1.取当前的属性值。
            var iCurrent = 0;
            if (attr === "opacity") { //为透明度时执行。
                iCurrent = Math.round(parseFloat(getStyle(element, attr)) * 100);
            } else { //默认情况
                iCurrent = parseInt(getStyle(element, attr)); //实际样式大小
            }
            //2.算运动速度,动画缓冲效果
            var iSpeed = (json[attr] - iCurrent) / 10; //(目标值-当前值)/缩放系数=速度
            iSpeed = iSpeed > 0 ? Math.ceil(iSpeed) : Math.floor(iSpeed); //速度取整

            //3.未到达目标值时，执行代码 
            if (iCurrent != json[attr]) {
                flag = false; //终止条件
                if (attr === "opacity") { //为透明度时，执行
                    element.style.filter = "alpha(opacity:" + (iCurrent + iSpeed) + ")"; //IE
                    element.style.opacity = (iCurrent + iSpeed) / 100; //标准
                } else { //默认
                    element.style[attr] = iCurrent + iSpeed + "px";
                }
            } else {
                flag = true;
            }
            //4. 运动终止，是否回调
            if (flag) {
                clearInterval(element.timer);
                if (func) {
                    func();
                }
            }
        }
    }, 30);
}

/**
 * 获取实际样式函数
 * @param   {HTMLElement}   element  需要寻找的样式的html节点
 * @param   {String]} attr 在对象中寻找的样式属性
 * @returns {String} 获取到的属性
 */
function getStyle(element, attr) {
    //IE写法
    if (element.currentStyle) {
        return element.currentStyle[attr];
        //标准
    } else {
        return getComputedStyle(element, false)[attr];
    }
}

```

## 运动框架总结

|    框架                                   | 变化   |
|   --------                                | -----:  |
| startMove(element)                        | 运动 | 
| startMove(element,iTarget)                | 匀速-->缓冲-->多物体 | 
| startMove(element,attr,iTargrt)           | 任意值 | 
| startMove(element,attr,iTargrt,func)      | 链式运动 | 
| startMove(element,json,func)              | 多值(同时)-->完美运动框架 | 








