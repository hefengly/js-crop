<template>
  <div>
       <div id="crop_image_content">
            <!-- 裁剪部分 -->
            <div id="cropBox">
                <div id="imgContainer">
                  <img id="cropimg1" alt="" :src="imgURL">
                </div>
                <!-- 裁剪框 -->
                <div id="mainBox" draggable="true" @dragstart="mainBoxDragS" @dragend="mainBoxDragE">
                    <div id="left-up" class="minBox left-up" draggable="true" @dragstart="changeDragS" @drag="changeE"></div>
                    <div id="up" class="minBox up" draggable="true" @dragstart="changeDragS" @drag="changeE"></div>
                    <div id="right-up" class="minBox right-up" draggable="true" @dragstart="changeDragS" @drag="changeE"></div>
                    <div id="left" class="minBox left" draggable="true" @dragstart="changeDragS" @drag="changeE"></div>
                    <div id="right" class="minBox right" draggable="true" @dragstart="changeDragS" @drag="changeE"></div>
                    <div id="left-down" class="minBox left-down" draggable="true" @dragstart="changeDragS" @drag="changeE"></div>
                    <div id="down" class="minBox down" draggable="true" @dragstart="changeDragS" @drag="changeE"></div>
                    <div id="right-down" class="minBox right-down" draggable="true" @dragstart="changeDragS" @drag="changeE"></div>
                </div>
            </div>
            <!-- 预览部分 -->
            <!-- <div id="preview">
                <img id="cropimg3" alt="" :src="imgURL">
            </div> -->
        </div>

    <form id="uploadForm" action="">
      <div id="uploadImage">
        <!-- <img id="cropimg4" alt="" :src="imgURL"> -->
        <a href="javascript:;" class="addImage">
          <input id="imgFile" type="file" name="imgFile" @change="imgChange">
        </a>
      </div>
    </form>
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
        imgContainerHeight: 0  // 图片剪切区域的高
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
        // 图片容器的位置
        let img_container = document.getElementById('imgContainer')
        this.imgContainerLeft = img_container.style.left
        this.imgContainerTop = img_container.style.top
        // 图片容器的宽高
        this.imgContainerWidth = img_container.clientWidth
        this.imgContainerHeight = img_container.clientHeight
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
        // 判断剪切框被脱出的情况，并处理
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
        let mainBoxUp = Number(mainBox.style.up.slice(0, -2))
        switch (e.target.id) {
          case 'right': {
            this.rightChange({clientX,clientY}, mainBox, mainBoxLeft)
            break
          }
          case 'left': {
            this.leftChange({clientX,clientY}, mainBox, mainBoxLeft)
            break
          }
          case 'up': {
            this.upChange({clientX,clientY}, mainBox, mainBoxUp)
          }
        }
      },
      // 向右移动
      rightChange: function (obj, mainBox, mainBoxLeft) {
        let {clientX, clientY} = obj
        // 剪切框的width
        let width = clientX - 30 - mainBoxLeft
        // 剪切框可以向右伸长的最长长度
        let largeWidth = this.imgContainerLeft + this.imgContainerWidth - mainBoxLeft
        // 如果超出的话，做截断处理
        width = width > largeWidth ? largeWidth : width
        mainBox.style.width = width + 'px'
        
      },
      // 向左移动
      leftChange: function(obj, mainBox, mainBoxLeft) {
        let {clientX, clientY} = obj
        // 当drop时，clientX突然会变为0，小坑
        if(clientX === 0) {
          return false
        }
        // 剪切框原本的宽
        let oldWidth = mainBox.clientWidth
        // 剪切框的width
        let width = oldWidth + mainBoxLeft + 30 - clientX
        // 剪切框移动后的left
        let leftMove = clientX -30
        // 剪切框可以向左伸长的最长长度
        let largeWidth = mainBoxLeft + oldWidth
        // 如果超出的话，做截断处理
        if(width > largeWidth) {
          width = largeWidth
          leftMove = 0
        }
        mainBox.style.width = width + 'px'
        mainBox.style.left = leftMove + 'px'
      }
    },
    created () {
      // window.onmousedown = (e) => {
      //   e.stopPropagation()
      //   this.ifDown = true
      //   // console.log(e)
      //   // console.log('鼠标按下啦')
      // }
      // window.onmouseup = (e) => {
      //   e.stopPropagation()
      //   // this.ifDown = false
      //   // console.log('鼠标没按啦')
      // }
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

#cropBox {
  position: relative;
}
#crop_image_content #imgContainer {
  font-size: 0;
  position: absolute;
  top: 0;
  left: 0;
  width: 30%;
  border: 1px solid pink;
}
#crop_image_content #cropimg1 {
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
</style>
