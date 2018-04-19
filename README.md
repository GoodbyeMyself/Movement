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
 4. 速度--物体运动的快慢因素【定时器间隔时间、改变值的大小】
 
        
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