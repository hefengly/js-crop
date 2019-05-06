<template>
  <div>
      <div id="crop_image_content">
        <!-- 裁剪部分 -->
        <div id="imgContainer" ref="imgContainer">
          <img id="cropimg" alt="" :src="imgURL" ref="img">
        </div>
        <!-- 裁剪框 -->
        <div id="mainBox" draggable="true" @dragstart="mainBoxDragS" @dragend="mainBoxDragE">
            <!-- <div id="left-up" class="minBox left-up" draggable="true" @dragstart="changeDragS" @drag="changeE"></div> -->
            <div id="up" class="minBox up" draggable="true" @dragstart="changeDragS" @drag="changeE"></div>
            <!-- <div id="right-up" class="minBox right-up" draggable="true" @dragstart="changeDragS" @drag="changeE"></div> -->
            <div id="left" class="minBox left" draggable="true" @dragstart="changeDragS" @drag="changeE"></div>
            <div id="right" class="minBox right" draggable="true" @dragstart="changeDragS" @drag="changeE"></div>
            <!-- <div id="left-down" class="minBox left-down" draggable="true" @dragstart="changeDragS" @drag="changeE"></div> -->
            <div id="down" class="minBox down" draggable="true" @dragstart="changeDragS" @drag="changeE"></div>
            <!-- <div id="right-down" class="minBox right-down" draggable="true" @dragstart="changeDragS" @drag="changeE"></div> -->
        </div>
      </div>

    <form id="uploadForm" action="">
      <div id="uploadImage">
        <a href="javascript:;" class="addImage">
          <input id="imgFile" type="file" name="imgFile" @change="imgChange">
        </a>
      </div>
    </form>

    <div id="dataContainer">
      <P>位置相对于左上角（top left）</P>
      <p>左上角：{{top_T}} {{left_L}}</p>
      <p>右上角：{{top_T}} {{left_R}}</p>
      <p>左下角：{{top_B}} {{left_L}}</p>
      <p>右下角：{{top_B}} {{left_R}}</p>
    </div>
  </div>
</template>
<script>
export default {
    data () {
      return {
        imgURL: require('./../assets/huya.jpg'),
        ifDown: false, // 判断鼠标是否按下去
        mouseInDom: {
          clientX: 0,
          clientY: 0,
        },             // 拖拽事件，鼠标在dom中的位置
        imgContainerLeft: 0,  // 图片剪切区域距离浏览器的left 距离
        imgContainerTop: 0,    // 图片剪切区域距离浏览器的top 距离
        imgContainerWidth: 0,  // 图片剪切区域的宽
        imgContainerHeight: 0,  // 图片剪切区域的高
        BOX_MINIMUM: 10,          // 裁剪框的最小范围
        left_L: '0%',              // 剪切框左边的left位置
        top_T: '0%',               // 剪切框上边的top位置
        left_R: '0%',             // 剪切框右边的left位置,
        top_B: '0%',               // 剪切框下边的top位置
        isImgOnload: false         // 图片是否加载完
      }
    },
    mounted: function () {
      this.getImgCData()
      // 监听页面变化
      window.onresize = () => {
        this.getImgCData()
        this.computePosition()
      }
    },
    methods : {
      // 导入图片时，更换展示区块的URL
      imgChange: function (event) {
        // 根据这个 <input> 获取文件的 HTML5 js对象
        var files = event.target.files
        if (files && files.length > 0) {
            // 获取目前上传的文件
            let file
            file = files[0]
            // 获取window的 URL工具
            var URL = window.URL || window.webkitURL
            // 通过 file生成目标 url
            var imgURL = URL.createObjectURL(file)
            // 用这个URL产生一个 <img> 将其显示出来
            this.imgURL = imgURL
        }
      },
      // 剪切框拖拽开始,记录鼠标距离拖拽dom 的位置
      mainBoxDragS: function (e) {
        // 记录此时鼠标的在dom中位置
        this.mouseInDom.clientX = e.clientX - this.imgContainerLeft - 30 - Number(e.target.style.left.slice(0,-2))
        this.mouseInDom.clientY = e.clientY - this.imgContainerTop - 30 - Number(e.target.style.top.slice(0,-2))
      },
      // 剪切框拖拽结束，判断拖拽的是大方块还是小方块，大方块就移动，小方块就缩放
      mainBoxDragE: function (e) {
        if (e.target.id !== 'mainBox') {
          return false
        } else {
          this.moveBox(e)
        }
        // 计算剪切框的位置
        this.computePosition()
      },
      // 移动剪切框
      moveBox: function(e) {
        // 鼠标的位置
        let clientX = e.clientX
        let clientY = e.clientY
        // 移动后剪切框的位置
        let top = clientY - this.imgContainerTop - 30 - this.mouseInDom.clientY 
        let left = clientX - this.imgContainerLeft - 30 - this.mouseInDom.clientX
        // 获取剪切框的宽高
        let mainBox = document.getElementById('mainBox')
        let mainboxWidth = mainBox.clientWidth
        let mainboxHeight = mainBox.clientHeight
        // 判断剪切框被拖出的情况，并处理
        if (top < this.imgContainerTop) {
          if(!this.imgContainerTop) {
            top = 0
          }
        }
        if (left < this.imgContainerLeft){
          if(!this.imgContainerLeft) {
            left = 0
          }
        }
        if (top + mainboxHeight > this.imgContainerTop + this.imgContainerHeight) {
          top = this.imgContainerTop + this.imgContainerHeight - mainboxHeight
        }
        if (left + mainboxWidth > this.imgContainerLeft + this.imgContainerWidth) {
          left = this.imgContainerLeft + this.imgContainerWidth - mainboxWidth
        }
        // 改变剪切框的位置
        e.target.style.top = top + 'px'
        e.target.style.left = left + 'px'
      },
      // 剪切框缩放的开始
      changeDragS: function (e) {
        e.stopPropagation()
        // console.log('sdafas')
      },
      // 剪切框大小的切换
      changeE: function (e) {
        let clientX = e.clientX
        let clientY = e.clientY
        let mainBox = document.getElementById('mainBox')
        // 剪切框的left
        let mainBoxLeft = Number(mainBox.style.left.slice(0, -2))
        // 剪切框的up
        let mainBoxTop = Number(mainBox.style.top.slice(0, -2))
        // 剪切宽原本的宽width
        let oldWidth = mainBox.clientWidth
        // 剪切框原本的高height
        let oldHeight = mainBox.clientHeight
        switch (e.target.id) {
          case 'right': {
            this.rightOrDown(clientX, mainBox, mainBoxLeft, 'width')
            break
          }
          case 'left': {
            this.leftOrUp(clientX, mainBox, mainBoxLeft, oldWidth, 'width')
            break
          }
          case 'up': {
            this.leftOrUp(clientY, mainBox, mainBoxTop, oldHeight, 'height')
            break
          }
          case 'down': {
            this.rightOrDown(clientY, mainBox, mainBoxTop, 'height')
          }
        }
        // 计算剪切框的位置
        this.computePosition()
      },
      // 向右或向下拉伸
      rightOrDown: function (clientXY, mainBox, LeftOrTop, typeWH) {
        // 剪切框的width或height
        let widthOrHeight = clientXY - 30 - LeftOrTop
        let imgLOrT, imgWOrH
        // 判断类型时右还是down
        if (typeWH === 'width') {
          imgLOrT = this.imgContainerLeft
          imgWOrH = this.imgContainerWidth
        } else {
          imgLOrT = this.imgContainerTop
          imgWOrH = this.imgContainerHeight
        }
        // 剪切框可以向右或向上伸长的最长长度
        let largeWOrH = imgLOrT + imgWOrH - LeftOrTop
        // 如果超出的话，做截断处理
        widthOrHeight = widthOrHeight > largeWOrH ? largeWOrH : widthOrHeight
        // 当裁剪宽小于最小宽度时，不作处理，否则会出现裁剪框一直移动的问题
        if(widthOrHeight < this.BOX_MINIMUM) {
          return
        }
        mainBox.style[typeWH] = widthOrHeight + 'px'
      },
      // 向上或者向左拉伸
      leftOrUp: function(clientXY, mainBox, LeftOrTop, oldHW, typeWH) {
        // 当drop时，clientX或者clientY突然会变为0，小坑
        if(clientXY === 0) {
          return false
        }
        // 剪切框移动后的width或height
        let widthOrHeight = oldHW + LeftOrTop + 30 - clientXY
        // 剪切框移动后的left或top
        let newLOrT = clientXY - 30
        // 剪切框可以向左或向上伸长的最长长度
        let largeWOrH = LeftOrTop + oldHW
        // 如果超出的话，做截断处理
        if(widthOrHeight > largeWOrH) {
          widthOrHeight = largeWOrH
          newLOrT = 0
        }
        // 当裁剪宽高小于最小宽高时，不作处理，否则会出现裁剪框一直移动的问题
        if(widthOrHeight < this.BOX_MINIMUM) {
          return
        }
        // 判断时left还是top
        let typeLT = (typeWH === 'width') ? 'left' : 'top'
        mainBox.style[typeWH] = widthOrHeight + 'px'
        mainBox.style[typeLT] = newLOrT + 'px'
      },
      // 计算裁剪框的位置
      computePosition: function () {
        let mainBox = document.getElementById('mainBox')
        let width = mainBox.clientWidth
        let height = mainBox.clientHeight
        let left = Number(mainBox.style.left.slice(0,-2))
        let top = Number(mainBox.style.top.slice(0,-2))
        this.left_L = ((left / this.imgContainerWidth) * 100 + '').slice(0, 5) + '%'
        this.top_T = ((top / this.imgContainerHeight) * 100 + '').slice(0, 5) + '%'
        this.left_R = (((left + width) / this.imgContainerWidth) * 100 + '').slice(0, 5) + '%'
        this.top_B = (((top + height) / this.imgContainerHeight) * 100 + '').slice(0, 5) + '%'
      },
      // 得到图片位置的信息
      getImgCData: function () {
        // 图片容器的位置
        let img_container = this.$refs.imgContainer
        this.imgContainerLeft = img_container.style.left
        this.imgContainerTop = img_container.style.top
        // 图片容器的宽高
        this.imgContainerWidth = img_container.clientWidth
        // 因为图片加载时异步的，要等加载完才能获取到正确的高度
        if(!this.isImgOnload) {
          this.$refs.img.onload = ()=>{
              this.imgContainerHeight = img_container.clientHeight
              this.isImgOnload = true
          }
        } else {
          this.imgContainerHeight = img_container.clientHeight
        }
      }
    }
}
</script>
<style>
body {
  background-color: #eee;
}
#crop_image_content {
  float: left;
  position: relative;
  text-align: center;
  width: 92%;
  margin: 30px;
}
#crop_image_content #imgContainer {
  font-size: 0;
  position: absolute;
  top: 0;
  left: 0;
  width: 40%;
  border: 1px solid pink;
}
#crop_image_content #cropimg {
  width: 100%;
}

#crop_image_content #mainBox {
  border: 1px solid rgb(52, 231, 7);
  position: absolute;
  top: 0;
  left: 0;
  width: 150px;
  height: 150px;
  cursor: move;
}
.minBox {
  position: absolute;
  height: 8px;
  width: 8px;
  background-color: rgb(52, 231, 7);
}
.left-up {
  top: -4px;
  left: -4px;
  cursor: nw-resize;
}

.up {
  left: 50%;
  margin-left: -4px;
  top: -4px;
  cursor: n-resize;
}

.right-up {
  right: -4px;
  top: -4px;
  cursor: ne-resize;
}

.left {
  top: 50%;
  margin-top: -4px;
  left: -4px;
  cursor: w-resize;
}

.right {
  top: 50%;
  margin-top: -4px;
  right: -4px;
  cursor: w-resize;
}

.left-down {
  bottom: -4px;
  left: -4px;
  cursor: sw-resize;
}

.down {
  bottom: -4px;
  left: 50%;
  margin-left: -4px;
  cursor: s-resize;
}

.right-down {
  bottom: -4px;
  right: -4px;
  cursor: se-resize;
}
#uploadForm {
  clear: both;
}
#dataContainer {
  margin-top: 5%;
}
</style>
