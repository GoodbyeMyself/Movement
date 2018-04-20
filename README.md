# 完美运动框架的实现思路

------

> 运动，其实就是在一段时间内改变left、right、width、height、opactiy的值，到达目的地之后停止。

 * 现在按照以下步骤来进行我们的运动框架的封装:

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

        -计时器setInterval(函数,交互时间(毫秒))：在执行时,从载入页面后每隔指定的时间执行代码。

        -取消计时器clearInterval(函数) 方法可取消由 setInterval() 设置的交互时间。
        
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

你没看错，就是那么简单。但是等等， what？ 怎么不会停？WTF？

那是因为我们没有运动终止条件。好再还是比较简单。直接在定时器内部，判断到达目标值，清除定时器就行拉！

``` javascript
/**
 * 运动框架-2-运动终止
 * @param {HTMLElement} element 进行运动的节点
 * @param {number}      iTarget 运动终止条件。
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

####运动中的BUG

      1.速度取到某些值会无法停止

      2.到达位置后再点击还会运动

      3.重复点击速度加快

      4.速度无法更改

####解决BUG

    1.速度取到某些值会无法停止（这个Bug稍后解决，在进化过程中自然解决）

    2.把运动和停止隔开(if/else)

    3.在开始运动时,关闭已有定时器

    4.把速度用变量保存