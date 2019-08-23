<template>
  <div class="preview-container">
    <!-- 缩略图 -->
    <div class="thumbnail_box" v-if="thumbnail">
      <!-- <div
        v-for="(item,index) of thumbnailList"
        :key="index"
        @click="handleCurrentChange(index,item)"
      >
        <img :class="item.cla" :src="encodeURI(item.url)" alt="#" />
      </div> -->
      <img
        v-for="(item,index) of thumbnailList"
        class="image-show"
        :key="index"
        v-lazy="encodeURI(item.url)"
        :class="item.cla"
        @click="handleCurrentChange(index,item)"
      >
      <!-- <el-image v-for="(item,index) of thumbnailList" class="image-show" :key="index" :src="encodeURI(item.url)" :class="item.cla" @click="handleCurrentChange(index,item)" lazy></el-image> -->
    </div>
    <!-- 缩略图 end -->
    <canvas
      id="canvas"
      class="canvas"
      @mousedown="down($event)"
      @mousemove="move($event)"
      @mouseup="up($event)"
      ref="canvas"
    ></canvas>
    <!-- <button @click="close" class="btn_close">关闭</button> -->
    <img src="../assets/img/board/close.png" @click="close" class="btn_close" alt />
  </div>
</template>
<script>
export default {
  props: ["imgObj"],
  data() {
    return {
      RI: {},
      thumbnail: false,
      thumbnailList: [],
      pre: 0,
      src: 'https://cube.elemecdn.com/6/94/4d3ea53c084bad6931a56d5158a48jpeg.jpeg'
    };
  },
  watch: {
    imgObj: {
      //深度监听，可监听到对象、数组的变化
      handler(newV, oldV) {
      },
      deep: true
    }
  },
  mounted() {
    this.init();
    if (this.imgObj.type == "img") {
      this.img(this.imgObj.url);
      this.thumbnail = false;
    } else {
      if (this.imgObj.urlList.length > 0) {
        this.thumbnail = true;
        this.thumbnailList = this.imgObj.thumbnail;
        this.img(this.imgObj.urlList[0]);
        this.thumbnailList[0].cla = "active";
      } else {
        this.$message({
          type: "info",
          message: "文件正在转换...请稍后再试"
        });
      }
    }
  },
  methods: {
    close() {
      this.thumbnail = false;
      this.thumbnailList = [];
      this.pre = 0;
      this.$emit("close", 1);
    },
    handleCurrentChange(i, item) {
      // 点击切换页码
      this.thumbnailList[i].cla = "active";
      this.thumbnailList[this.pre].cla = "";
      this.pre = i;
      this.init();
      this.img(this.imgObj.urlList[i]);
    },
    init() {
      this.RI.canvas = document.getElementById("canvas"); // 获取到第一层canvas元素
      this.RI.ctxCanvas = this.RI.canvas.getContext("2d"); // 获取到第一层canvas元素的上下文
      this.RI.canvasWidth = this.$refs.canvas.clientWidth;
      this.RI.canvasHeight = this.$refs.canvas.clientHeight;
      this.RI.canvas.width = this.$refs.canvas.clientWidth;
      this.RI.canvas.height = this.$refs.canvas.clientHeight;
      this.RI.zoom = 1;
      this.RI.canvas.addEventListener("mousewheel", this.handleScroll); // chrome and ie
      this.RI.canvas.addEventListener("DOMMouseScroll", this.handleScroll);
    },
    img(url) {
      this.RI.img = new Image();
      this.RI.img.onload = () => {
        // 图片加载完毕后执行以下方法
        this.RI.imgWidth = this.RI.img.naturalWidth; // 获取图片的原始高宽
        this.RI.imgHeight = this.RI.img.naturalHeight;
        let initImg = this.initLoadImg(
          this.RI.canvasWidth,
          this.RI.canvasHeight,
          this.RI.imgWidth,
          this.RI.imgHeight
        ); // 返回图片应该加载的初始位置和结束位置
        this.RI.x = initImg.x;
        this.RI.y = initImg.y;
        this.RI.x1 = initImg.x;
        this.RI.y1 = initImg.y;
        this.RI.endx = initImg.endx;
        this.RI.endy = initImg.endy;
        this.RI.endx1 = initImg.endx;
        this.RI.endy1 = initImg.endy;
        this.RI.ctxCanvas.clearRect(
          0,
          0,
          this.RI.canvasWidth,
          this.RI.canvasHeight
        ); // 加载背景图之前先清空背景图canvas
        this.RI.ctxCanvas.drawImage(
          this.RI.img,
          0,
          0,
          this.RI.imgWidth,
          this.RI.imgHeight,
          this.RI.x,
          this.RI.y,
          this.RI.endx,
          this.RI.endy
        ); // 加载背景图
      };
      this.RI.img.src = encodeURI(url); // src = 原图路径
    },
    initLoadImg(canvasWidth, canvasHeight, imageWidth, imageHeight) {
      if (imageWidth / imageHeight > canvasWidth / canvasHeight) {
        let info = {};
        info.endx = canvasWidth;
        info.endy = parseInt(canvasWidth / (imageWidth / imageHeight));
        info.x = 0;
        info.y = parseInt((canvasHeight - info.endy) / 2);
        return info;
      } else if (imageWidth / imageHeight < canvasWidth / canvasHeight) {
        let info = {};
        info.endx = parseInt(canvasHeight * (imageWidth / imageHeight));
        info.endy = canvasHeight;
        info.x = parseInt((canvasWidth - info.endx) / 2);
        info.y = 0;
        return info;
      } else {
        let info = {};
        info.endx = canvasWidth;
        info.endy = canvasHeight;
        info.x = 0;
        info.y = 0;
        return info;
      }
    },
    down(e) {
      this.RI.downx = e.offsetX;
      this.RI.downy = e.offsetY; // 保存鼠标点下的位置
      this.RI.flag = true; // 确认鼠标点下，可以触发移动
    },
    up(e) {
      this.RI.flag = false; // 确认鼠标点下，可以触发移动
      this.RI.x1 = this.RI.x;
      this.RI.y1 = this.RI.y;
      this.RI.movex = e.offsetX - this.RI.downx;
      this.RI.movey = e.offsetY - this.RI.downy;
    },
    move(e) {
      if (this.RI.flag) {
        // 判断是否点下了
        this.RI.movex = e.offsetX - this.RI.downx;
        this.RI.movey = e.offsetY - this.RI.downy;
        this.RI.x = this.RI.movex + this.RI.x1;
        this.RI.y = this.RI.movey + this.RI.y1;
        this.RI.ctxCanvas.clearRect(
          0,
          0,
          this.RI.canvasWidth,
          this.RI.canvasHeight
        ); // 清空背景图
        this.RI.ctxCanvas.drawImage(
          this.RI.img,
          0,
          0,
          this.RI.imgWidth,
          this.RI.imgHeight,
          this.RI.x,
          this.RI.y,
          this.RI.endx,
          this.RI.endy
        ); // 重新加载背景
      } else {
        if (
          e.offsetX > this.RI.x &&
          e.offsetX < this.RI.endx + this.RI.x &&
          e.offsetY > this.RI.y &&
          e.offsetY < this.RI.endy + this.RI.y
        ) {
          this.RI.Roller = true; // 判断鼠标移动在图片上才能进行缩放操作
        } else {
          this.RI.Roller = false;
        }
      }
    },
    handleScroll(e) {
      var delta = e.wheelDelta || -e.detail;
      if (this.RI.Roller && !this.RI.flag) {
        this.RI.Rollerx = e.offsetX - this.RI.x;
        this.RI.Rollery = e.offsetY - this.RI.y;
        if (delta > 0) {
          // 向上滚动 放大
          if (this.RI.endx1 * this.RI.zoom < this.RI.imgWidth * 10) {
            this.RI.zoom += 0.2;
            this.RI.zoom = parseFloat(this.RI.zoom.toFixed(2));
          }
          this.RI.x =
            this.RI.x -
            (this.RI.endx1 * this.RI.zoom - this.RI.endx) *
              (this.RI.Rollerx / this.RI.endx);
          this.RI.y =
            this.RI.y -
            (this.RI.endy1 * this.RI.zoom - this.RI.endy) *
              (this.RI.Rollery / this.RI.endy);
          this.RI.endx = this.RI.endx1 * this.RI.zoom;
          this.RI.endy = this.RI.endy1 * this.RI.zoom;
          this.RI.ctxCanvas.clearRect(
            0,
            0,
            this.RI.canvasWidth,
            this.RI.canvasHeight
          );
          this.RI.ctxCanvas.drawImage(
            this.RI.img,
            0,
            0,
            this.RI.imgWidth,
            this.RI.imgHeight,
            this.RI.x,
            this.RI.y,
            this.RI.endx,
            this.RI.endy
          );
          this.RI.x1 = this.RI.x;
          this.RI.y1 = this.RI.y;
        } else {
          // 向下滚动 缩小
          this.RI.zoom -= 0.2;
          this.RI.zoom = parseFloat(this.RI.zoom.toFixed(2));
          if (this.RI.zoom < 0.1) {
            this.RI.zoom = 0.1;
          }
          this.RI.x =
            this.RI.x -
            (this.RI.endx1 * this.RI.zoom - this.RI.endx) *
              (this.RI.Rollerx / this.RI.endx);
          this.RI.y =
            this.RI.y -
            (this.RI.endy1 * this.RI.zoom - this.RI.endy) *
              (this.RI.Rollery / this.RI.endy);
          this.RI.endx = this.RI.endx1 * this.RI.zoom;
          this.RI.endy = this.RI.endy1 * this.RI.zoom;
          this.RI.ctxCanvas.clearRect(
            0,
            0,
            this.RI.canvasWidth,
            this.RI.canvasHeight
          );
          this.RI.ctxCanvas.drawImage(
            this.RI.img,
            0,
            0,
            this.RI.imgWidth,
            this.RI.imgHeight,
            this.RI.x,
            this.RI.y,
            this.RI.endx,
            this.RI.endy
          );
          this.RI.x1 = this.RI.x;
          this.RI.y1 = this.RI.y;
        }
      }
    }
  }
};
</script>
<style lang="scss" scoped>
@import "scss_vars";
.preview-container {
  width: 100%;
  height: 100%;
  position: fixed;
  top: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.6);
  z-index: 100;
  .canvas {
    width: 100%;
    height: 100%;
  }
  .btn_close {
    position: fixed;
    top: 10px;
    right: 10px;
    cursor: pointer;
  }
  /* 缩略图样式 */
  .thumbnail_box {
    position: fixed;
    left: 0;
    top: 0;
    bottom: 0;
    width: 150px;
    background: rgba(0, 0, 0, 0.432);
    padding: 10px;
    overflow-y: auto;
    img {
      width: 97%;
      cursor: pointer;
      margin-bottom: 10px;
      border: 2px solid rgb(255, 255, 255);
    }
    img:hover {
      border: 2px solid rgb(3, 245, 253);
    }
    .active {
      border: 2px solid rgb(224, 20, 20);
    }
    .image-show{
      display: flex;
      justify-content: flex-start;
      margin: 5px auto;
    }
  }
}
</style>