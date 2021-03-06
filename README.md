# canvas-demo

&emsp;&emsp;本示例是源于项目的一个需求做的一个demo。初衷只是为了实现通过拖拽绘制用户所需大小的矩形，后面就继续完善了一下功能。目前实现了通过拖拽绘制矩形、移动画布上现有的矩形、保存当前画布、撤销与反撤销

&emsp;&emsp;canvas是HTML5新增的元素，是HTML5 的一大亮点，canvas翻译过来其实就是画布的意思，它可以替代flash，制作网页的很多动画效果以及游戏。渲染效率非常高，不像flash要在游览器安装flash adobe插件，canvas不需要安装任何插件即可渲染这个动画。目前所有主流游览器都支持canvas。

## canvas常用api

canvas的基础用法大家可以去查看文档，相信会看文档的人都会用了。以下列出了常用的几个api。
``` js
// ctx为canvas上下文对象
ctx.beginPath() // 开始一个路径

ctx.moveTo(x,y) // 路径移到画布中的指定点 , 即起点

ctx.lineTo(x,y) //添加一个新点，画线

ctx.closePath() // 关闭绘制路径

ctx.fillStyle // 设置填充颜色

ctx.fill() // 填充区域

ctx.lineWidth // 设置线的宽度

ctx.strokeStyle // 设置描边颜色

ctx.stroke() // 填充描边

ctx.rect(x,y,w,h) // 绘制矩形 x、y为起始坐标，w、h为矩形的宽、高
ctx.fillRect(x,y,w,h) // 填充矩形
ctx.strokeRect(x,y,w,h) // 描边矩形

arc(x,y,r,sa,ea,true/false) // 绘制圆形 x、y为圆心坐标，r为半径，sa为起始角度，ea为结束角度，true是逆时针画圆，false是顺时针画圆

fillText(text,x,y,maxWidth) // 填充绘制 text表示文字，x、y为坐标，maxWidth可选，为文字最大宽度，防止文字溢出
strokeText(text,x,y,maxWidth) // 描边绘制 text表示文字，x、y为坐标，maxWidth可选，为文字最大宽度，防止文字溢出

 ctx.clearRect(x,y,width,height) // x为清除起点横坐标， y为清除起点纵坐标，width为清除长度，height为清除高度
```

## 功能1： 鼠标拖拽绘制矩形
&emsp;&emsp;在鼠标按下时记录当前按下位置的坐标值，为起始坐标，松开鼠标时的坐标为结束坐标，以这两个坐标可以得到四个点，绘制出一个矩形
``` js
// 为canvas注册事件
window.onload = function () {
  canvas = document.getElementById('canvas')
  context = canvas.getContext('2d')

  canvas.onmousedown = mouseDown
  canvas.onmouseup = mouseUp
}

// 鼠标按下事件，记录起始坐标值
function mouseDown(e) {
  startX = e.offsetX
  startY = e.offsetY
}

// 鼠标松开事件， 记录结束坐标
function mouseUp(e) {
  endX = e.offsetX
  endY = e.offsetY
  rectList.unshift(new Rect(startX, startY, endX, endY, color))
  // 绘制矩形
  context.beginPath()
  context.globalAlpha = 0.3 // 透明度
  context.moveTo(startX, startY)
  context.lineTo(endX, startY)
  context.lineTo(endX, endY)
  context.lineTo(startX, endY)
  context.lineTo(startX, startY)
  context.fillStyle = 'yellow'
  context.strokeStyle = 'black'
  context.fill()
  context.stroke()
}
```
&emsp;&emsp;以上代码就可以绘制出一个矩形，可能有人会问为什么不直接用rect()方法来绘制，如果用rect()方法的话就得计算宽高，并通过计算方向来判断鼠标按下时的点为起始坐标还是松开时的点为起始坐标，而使用路径的方法绘制矩形就不用那么麻烦了。

&emsp;&emsp;但是是有一个问题，就是鼠标点击后拖拽并没有达到我们想要的效果，从点击———>拖拽———>松开这3步期间应该有个过渡效果。

优化：在鼠标拖拽过程中不断的改变结束的坐标值，并绘制矩形  
&emsp;&emsp;&emsp;为canvas注册鼠标拖拽事件mouseMove
``` js
let isDrawing = false // 是否正在绘图
// 在鼠标按下时设置isDrawing=true， 松开时为false
function mouseMove(e) {
  if (isDrawing) { // 判断是否正在绘图
    endX = e.offsetX
    endY = e.offsetY
    context.globalAlpha = 0.3
    context.beginPath()
    context.moveTo(startX, startY)
    context.lineTo(endX, startY)
    context.lineTo(endX, endY)
    context.lineTo(startX, endY)
    context.lineTo(startX, startY)
    context.fillStyle = color
    context.strokeStyle = 'black'
    context.fill()
    context.stroke()
  }
}
```
运行后就会发现出现以下情况：  
![avatar][pic1]  
解决方案：在每次绘制时都清空画布  
&emsp;&emsp;为了能同时绘制出多个矩形，用一个数组存储每个矩形对象，每次松开鼠标后把当前的矩形对象存进数组的头部（因为每次绘制是按顺序从数组中取出矩形绘制，如果存进尾部后绘制的矩形就会被前面的矩形覆盖，所以这里用的是unshift而不是push），因为每次绘制都需要清空画布，所以每次在画布上绘制矩形时都要还原之前画布的状态
``` js
// 定义一个矩形的函数
function Rect(startX, startY, endX, endY, color) {
  this.startX = startX // 起始横坐标
  this.startY = startY // 起始纵坐标
  this.endX = endX // 结束横坐标
  this.endY = endY // 结束纵坐标
  this.color = color // 填充颜色（随机颜色）
  this.isSelected = false // 是否被选中
}
let rectList = [] // 矩形对象数组

function mouseUp(e) {
  rectList.unshift(new Rect(startX, startY, endX, endY, color))
  isDrawing = false
}

// 还原画布状态，在mouseMove中每次绘制前先调用该函数
function drawRects() {
  context.clearRect(0, 0, canvas.width, canvas.height)
  for (let i = 0; i < rectList.length; i++) {
    let rect = rectList[i]
    context.globalAlpha = 0.3
    context.beginPath()
    context.moveTo(rect.startX, rect.startY)
    context.lineTo(rect.endX, rect.startY)
    context.lineTo(rect.endX, rect.endY)
    context.lineTo(rect.startX, rect.endY)
    context.lineTo(rect.startX, rect.startY)
    context.fillStyle = rect.color
    context.fill()
  }
}
```

## 功能2：鼠标拖动矩形变更位置
&emsp;&emsp;在鼠标按下时要先判断该操作是绘制还是拖拽，这里以鼠标按下时的坐标是否在矩形内来判断，在矩形外则为绘制（isDrawing=true），在矩形内则为拖拽（isDragging=true）。若是为拖拽还要判断是哪一个矩形，选中的矩形设置isSelected=true。为了方便区分，为正在操作的矩形添加黑色边框。
```js
function mouseDown(e) {
  startX = e.offsetX
  startY = e.offsetY
  rectIndex = rectList.findIndex(item => { // 是否在矩形内，findIndex()查找数组内的对象，找到符合条件的则返回该对象的下标，没有则返回-1
    if (item.startX < item.endX) {
      if (item.startY < item.endY) {
        return startX > item.startX && startX < item.endX && startY > item.startY && startY < item.endY
      } else {
        return startX > item.startX && startX < item.endX && startY > item.endY && startY < item.startY
      }
    } else {
      if (item.startY < item.endY) {
        return startX > item.endY && startX < item.startY && startY > item.startY && startY < item.endY
      } else {
        return startX > item.startX && startX < item.endX && startY > item.endY && startY < item.startY
      }
    }
  })
  if (rectIndex !== -1) {
    currentRect = rectList[rectIndex]
    isDragging = true
    currentRect.isSelected = true
  } else {
    isDrawing = true
  }
  color = colors[randomFromTo(0, 8)] // colors为存储颜色的数组，randomFromTo为一个方法，返回随机整数
}
function mouseMove(e) {
  endX = e.offsetX
  endY = e.offsetY
  if (isDrawing) {
    drawRects()
    context.globalAlpha = 0.3
    context.beginPath()
    context.moveTo(startX, startY)
    context.lineTo(endX, startY)
    context.lineTo(endX, endY)
    context.lineTo(startX, endY)
    context.lineTo(startX, startY)
    context.fillStyle = color
    context.strokeStyle = 'black'
    context.fill()
    context.stroke()
  } else if (isDragging) {
    const w = Math.abs(startX - endX)
    const h = Math.abs(startY - endY)
    if (endX < startX) {
      startX -= w
      endX -= w
      currentRect.startX -= w
      currentRect.endX -= w
    }
    if (endX >= startX) {
      startX += w
      endX += w
      currentRect.startX += w
      currentRect.endX += w
    }
    if (endY < startY) {
      startY -= h
      endY -= h
      currentRect.startY -= h
      currentRect.endY -= h
    }
    if (endY >= startY) {
      startY += h
      endY += h
      currentRect.startY += h
      currentRect.endY += h
    }
    drawRects()
  }
}

function mouseUp(e) {
  if (isDrawing) {
    rectList.unshift(new Rect(startX, startY, endX, endY, color))
    isDrawing = false
  }
  if (isDragging) {
    rectList.forEach(item => {
      item.isSelected = false
    })
    isDragging = false
  }
}

function drawRects() {
  context.clearRect(0, 0, canvas.width, canvas.height)
  for (let i = 0; i < rectList.length; i++) {
    let rect = rectList[i]
    context.globalAlpha = 0.3
    context.beginPath()
    context.moveTo(rect.startX, rect.startY)
    context.lineTo(rect.endX, rect.startY)
    context.lineTo(rect.endX, rect.endY)
    context.lineTo(rect.startX, rect.endY)
    context.lineTo(rect.startX, rect.startY)
    context.fillStyle = rect.color
    context.fill()
    if (rect.isSelected) {
      context.strokeStyle = 'black'
      context.stroke()
    }
  }
}
```

## 功能3：保存当前画布的图像
&emsp;&emsp;利用HTMLCanvasElement.toDataURL()将画布图像转换成base64格式，并动态生成节点将图片展示出来，也可使用file-saver将图片保存到本地
``` js
function save () {
  const data = canvas.toDataURL('image/png', 1)
  const chileNode =document.createElement('img')
  chileNode.src = data
  document.getElementById('img-container').appendChild(chileNode)
}
```

## 功能4：清空画布
&emsp;&emsp;清空画布在之前很多代码中都有使用过，为了方便把它抽离出来
```js
function clearCanvas() {
  context.clearRect(0, 0, canvas.width, canvas.height)
}
```
&emsp;&emsp;但是要完全清空还要把存储矩形对象的数组也清空掉，如果没有清空矩形数组的话后续绘制会把之前的矩形都显示出来
``` js
function clearAll () {
  rectList = []
  clearCanvas()
}
```

## 功能5：撤销与反撤销
### 撤销
&emsp;&emsp;用undoArray存储每一步的操作，把每次绘制后的状态存储到数组中，每撤销一次就把数组末尾的pop出来，恢复到前一个状态，注意要加上判断，以免溢出报错
``` js
function mouseDown(e) {
  // 省略重复代码
  if (rectIndex !== -1) {
    currentRect = rectList[rectIndex]
    isDragging = true
    currentRect.isSelected = true
    undoArray.pop()
    const tempRectList = rectList.slice()
    const tempCurrentRect = Object.assign({}, currentRect)
    tempRectList.splice(rectIndex, 1, tempCurrentRect)
    undoArray.push(tempRectList)
  } else {
    isDrawing = true
  }
}
function mouseUp(e) {
  if (isDrawing) {
    rectList.unshift(new Rect(startX, startY, endX, endY, color))
  }
  if (isDragging) {
    rectList.forEach(item => {
      item.isSelected = false
    })
  }
  undoArray.push(rectList.slice()) // 在鼠标松开的时候把当前状态即rectList存进undoArray
  isDrawing = false
  isDragging = false
}
function undo () {
  if (undoArray.length > 0) {
    undoArray.pop()
    rectList = undoArray[undoArray.length - 1].slice()
  } else {
    rectList = []
  }
  drawRects()
}
```
### 反撤销
&emsp;&emsp;反撤销其实和撤销差不多，这里用一个redoArray数组来存储反撤销的状态，把撤销时pop出来的队形push进redoArray里面，每执行一次反撤销也是把数组末尾的pop出来，恢复到前一个状态，pop出来的对象在存回undoArray里面，就可以实现连续的撤销与反撤销，这里也要加上判断，以免溢出报错
``` js
function undo () {
  // context.clearRect(0, 0, canvas.width, canvas.height)
  if (undoArray.length > 0) {
    redoArray.push(undoArray.pop())
    rectList = undoArray[undoArray.length - 1].slice()
  } else {
    rectList = []
  }
  drawRects()
}
function redo () {
  // context.clearRect(0, 0, canvas.width, canvas.height)
  if (redoArray.length > 0) {
    rectList = redoArray[redoArray.length - 1].slice()
    undoArray.push(redoArray.pop())
  }
  drawRects()
}
```
以上代码看起来会迷糊，之后会做优化

### 参考文献
1. [canvas实现鼠标拖拽矩形移动改变大小](https://www.cnblogs.com/fengchaoran/p/8057338.html "")  
2. [HTML5 - Canvas的使用样例14（图形增加鼠标点击、拖动交互）](https://www.hangge.com/blog/cache/detail_1057.html "")
3. [探究 canvas 绘图中撤销（undo）功能的实现方式）](https://blog.csdn.net/weixin_33890499/article/details/88817414 "")

[pic1]:data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAOMAAADyCAIAAAAwZuUlAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAAJcEhZcwAADsMAAA7DAcdvqGQAAAycSURBVHhe7Zzfb1R1GsZ35rSdFtrSbbNNHGNqDOJG02yCabjYhOgmGk1MTA0x8YLUiF5oYiJKUiMKBYpSLGBKoTPtQGCmICAUKNMfXJhSTIgGNU340fbOf8Arb5QI7Pf0mR3e4dUN2aXzni95Pnkymefw2HaaT85wZgb/cpsQH6CpxA9oKvEDmkr8gKYSP6CpxA9oKvEDmkr8gKYSP6CpxA9oKvEDmkr8gKYSP6CpxA9oKvEDmkr8gKYSP6CpxA9oKvEDmkr8gKYSP6CpxA9oKvEDmkr8gKYSP6CpxA9oKvEDmkr8gKYSP6CpxA9oKvEDmkr8gKYSP6CpxA9oKvEDmkr8gKYSP6CpxA9oKvEDmkr8gKYSP6CpxA+iZWpA/oiqqqrEArW1tfX19Q0NDc0LJJPJ5f/hySefXLly5T8XaF+go6Oju7s7k8lMTk4Wfr8+EzlTDx+O2aanpyQbNhTuuD8q3l+3rnDE3bqDxePujju4Zs3dx5G7vrirSPF+8Xjxtpg/rO62NPFsNtixoyKXS2Sz1bt3N5882ZrPv0RT7z8w9fZts8zPxyYn7+To0VgmE95xx7/5pnB/cDC2fXt4xMUNdu0qHHf33e2pU7H160uOI9jLWvyOuI8/RUXkWNe5uditW3H3O3O5edOl8ubN6tnZxLVr9Tdu/HVmpvHSpcd/+mk1TV0UaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIse60lRLaCoqIselNb6Q0FGaagNNRUXkuLTSVGtoKioix6WVplpDU1EROS7W8+dDR11mZ4NbtyoQmlpuaCoqIsfF+oemLshKU8vIA2Pq8HBs587YwEBsZCR2/HjMPajR0TA9PXfiDiJumc2G+fzzMO6/dV/TfbU9e2J794bfzt1339198YXEnaMLgZ0yNLVceG2qM2x8PHbyZOztt0NHN2wIld2yJbZxY+z992Pr1oVHXnwx/sgjwWOPBU88EbS23klbWyGrVwcv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