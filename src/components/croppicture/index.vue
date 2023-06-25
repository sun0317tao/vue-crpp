<template>
  <div
    ref="croppictureRef"
    class="croppicture"
    :style="{
      width: croppwidth + 'px',
      height: croppheight + 'px',
      backgroundColor: bacColor,
      backgroundImage: backImage,
    }"
    @mouseenter="croppMouse"
  >
    <!-- 展示图片的画布 -->
    <canvas
      id="cvs"
      ref="cvsRef"
      :width="croppwidth * ratio"
      :height="croppheight * ratio"
      :style="{ width: croppwidth + 'px', height: croppheight + 'px' }"
      >您的浏览器不支持canvas请升级</canvas
    >
    <!-- 遮罩的canvas -->
    <canvas
      id="cvs_mask"
      ref="cvsmaskRef"
      :width="croppwidth * ratio"
      :height="croppheight * ratio"
      :style="{ width: croppwidth + 'px', height: croppheight + 'px' }"
      @mousedown="canvasMousedown"
      @mouseup="maskCanvasMouse('up')"
      @mouseleave="maskCanvasMouse"
      @mousewheel="maskCanvasMouseWheel"
      @touchstart="maskCanvasTouchstart"
      @touchend="maskCanvasTouchend"
      >您的浏览器不支持canvas请升级</canvas
    >
    <!-- 裁剪的盒子 -->
    <div
      ref="croppRef"
      :style="{ width: croppBoxWidth + 'px', height: croppBoxHeight + 'px' }"
      class="cropp"
      @mousedown="croppMousedown"
      @mouseup="croppMouse('up')"
      @touchstart="maskCanvasTouchstart"
      v-show="isCropp"
    >
      <div
        class="right_top"
        ref="rightTopRef"
        @mousedown="rightTopmousedown"
      ></div>
      <div class="right_bottom" @mousedown="rightBottommousedown"></div>
      <div class="left_top" @mousedown="leftTopmousedown"></div>
      <div class="left_bottom" @mousedown="leftBottommousedown"></div>
    </div>
    <!-- <button
      v-show="isCropp"
      class="cropp_button"
      @click="confirmImage('confirm')"
    >
      确定
    </button>
    <button class="cropp_button_cancel" @click="cancel">取消</button> -->
  </div>
</template>

<script>
export default {
  props: {
    // 画布宽
    croppwidth: {
      type: Number,
      default: 800,
      required: false,
    },
    // 画布高
    croppheight: {
      type: Number,
      default: 400,
      required: false,
    },
    // 裁剪盒子宽
    croppBoxWidth: {
      type: Number,
      default: 200,
      required: false,
    },
    // 裁剪盒子高
    croppBoxHeight: {
      type: Number,
      default: 200,
      required: false,
    },
    // 放大缩小的缓慢（值越大放大或缩小的越快）
    scalenum: {
      type: Number,
      default: 0.01,
      required: false,
    },
    // 文件对象或者图片地址
    fileOrUrl: {
      type: null,
      default: "",
    },
    // 背景色
    backGroundColor: {
      type: [String, Boolean],
      default: "#000",
    },
    // 裁剪框四个角颜色
    croppfourthColor: {
      type: [String],
      default: "#fff",
    },
  },
  data() {
    return {
      // 获取设备像素比  (解决图片模糊问题，放大canvas的像素比)
      ratio: window.devicePixelRatio || 1,
      isCropp: false, // 是否显示裁剪框
      croppicture: null, // 父盒子dom
      img: null, // 图片dom
      scale: 1, // 放大缩小倍率
      // 图片坐标
      PO: {
        px: null,
        py: null,
      },
      beginImg: {
        width: null,
        height: null,
      },
      // drawImg: {
      //   img: null, //规定要使用的图像、画布或视频
      //   sx: 0, // 开始剪切之前画布（就是刚前一位传入的dom）的 x 坐标位置
      //   sy: 0, // 开始剪切之前画布（就是刚前一位传入的dom）的 y 坐标位置
      //   swidth: 0, // 开始剪切之前画布（就是刚前一位传入的dom）的宽度
      //   sheight: 0, // 开始剪切之前画布（就是刚前一位传入的dom）的高度
      //   x: 0, //在新画布上放置图像的 x 坐标位置
      //   y: 0, //在新画布上放置图像的 y 坐标位置
      //   width: 0, //新画布要使用的图像的宽度
      //   height: 0, //新画布要使用的图像的高度
      // },
    };
  },
  mounted() {
    // 获取夫盒子dom
    this.croppicture = this.$refs["croppictureRef"];
    const cropp = this.$refs["croppRef"];
    cropp.style.setProperty(
      "--borderStyle",
      `2px solid ${this.croppfourthColor}`
    );
    console.log(
      "欢迎使用vue-cropp, gitee地址： https://gitee.com/sun0317tao/vue-cropp"
    );
  },
  watch: {
    fileOrUrl: {
      handler(newval, oldval) {
        if (newval && newval.type?.includes("image")) {
          this.isCropp = true;
          const readfile = new FileReader();
          readfile.readAsDataURL(newval);
          readfile.onload = () => {
            this.generateImage(readfile.result);
            this.initcropp();
            this.drawCropp();
          };
        }
      },
      immediate: true, // 立即执行
      deep: true, // 深度监听复杂类型内变化
    },
  },
  computed: {
    bacColor() {
      if (this.backGroundColor || typeof this.backGroundColor == "string") {
        return typeof this.backGroundColor == "boolean"
          ? "#000"
          : this.backGroundColor;
      } else {
        return "";
      }
    },
    backImage() {
      if (typeof this.backGroundColor == "boolean" && !this.backGroundColor) {
        return this.backGroundColor
          ? "#000"
          : "linear-gradient( 45deg, rgba(0, 0, 0, 0.25) 25%, transparent 0, transparent 75%, rgba(0, 0, 0, 0.25) 0 ), linear-gradient(45deg, rgba(0, 0, 0, 0.25) 25%, transparent 0, transparent 75%, rgba(0, 0, 0, 0.25) 0)";
      } else {
        return "";
      }
    },
  },
  methods: {
    // 初始化裁剪框位置
    initcropp() {
      const cropp = this.$refs["croppRef"];
      const cvs = this.$refs["cvsRef"];
      cropp.style.left =
        cvs.width / this.ratio / 2 - cropp.offsetWidth / 2 + "px";
      cropp.style.top =
        cvs.height / this.ratio / 2 - cropp.offsetHeight / 2 + "px";
      cropp.style.transform = "none";
    },
    // 生成图片
    generateImage(val) {
      const img = new Image();
      img.src = val;
      // "https://www.try0317.com/zb_users/upload/2023/03/previewFix3.jpg";
      // "https://www.try0317.com/zb_users/upload/2023/05/202305061683337551449485.png";
      // "https://himg.bdimg.com/sys/portrait/item/public.1.73fd24e5.8h-XE9oaTiAVGLdgYuNeuw.jpg";
      img.onload = () => {
        this.img = img;
        this.renderImage(img);
      };
    },
    // 给底层画布添加图片
    renderImage(img) {
      /** @type {HTMLCanvasElement} */
      const canvas = this.$refs["cvsRef"];
      const ctx = canvas.getContext("2d");
      let scale = 1; // 缩放倍率 （后期用来缩放图片使用）
      let imgWidth = img.width;
      let imgHeight = img.height;

      if (imgWidth < this.croppwidth && imgHeight < this.croppheight) {
        scale = 1;
      } else {
        if (imgWidth / imgHeight <= this.croppwidth / this.croppheight) {
          // 计算宽高比
          scale = (this.croppheight * this.ratio) / imgHeight;
        } else {
          scale = (this.croppwidth * this.ratio) / imgWidth;
        }
      }
      let x = (this.croppwidth * this.ratio - img.width * scale) / 2;
      let y = (this.croppheight * this.ratio - img.height * scale) / 2;
      this.scale = scale;
      // 记录图片大小
      this.beginImg.width = img.width * this.scale;
      this.beginImg.height = img.height * this.scale;
      // 记录原始坐标
      this.PO.px = x;
      this.PO.py = y;
      ctx.drawImage(
        img,
        0,
        0,
        img.width,
        img.height,
        x,
        y,
        img.width * this.scale,
        img.height * this.scale
      );
      // 初始化完图片生成一次当前坐标的图片
      this.confirmImage("up");
    },

    // 设置遮罩&在遮罩裁剪一个矩形框
    drawCropp() {
      /** @type {HTMLCanvasElement} */
      const canvasMask = this.$refs["cvsmaskRef"];
      const cropp = this.$refs["croppRef"];
      const ctx = canvasMask.getContext("2d");
      // 先清除画布
      ctx.clearRect(0, 0, canvasMask.width, canvasMask.height);
      // 设置遮罩
      ctx.fillStyle = "#00000090";
      ctx.fillRect(0, 0, canvasMask.width, canvasMask.height);
      // 在遮罩上设置裁剪区域（挖个洞）
      ctx.clearRect(
        cropp.offsetLeft * this.ratio,
        cropp.offsetTop * this.ratio,
        cropp.offsetWidth * this.ratio,
        cropp.offsetHeight * this.ratio
      );
    },

    /**
     * 裁剪框
     */
    // 裁剪框鼠标按下事件
    croppMousedown(event) {
      var rect = this.croppicture.getBoundingClientRect();
      const elementLeft = rect.left + window.scrollX;
      const elementTop = rect.top + window.scrollY;
      this.croppicture.onmousemove = (e) => {
        this.croppMousemove(e, event, elementLeft, elementTop);
      };
    },
    // 父盒子鼠标移动事件
    croppMousemove(e, event, elementLeft, elementTop) {
      let x = e.pageX - elementLeft;
      let y = e.pageY - elementTop;
      let sx = x - event.offsetX;
      let sy = y - event.offsetY;
      let wi = this.croppicture.offsetWidth - event.target.offsetWidth;
      let he = this.croppicture.offsetHeight - event.target.offsetHeight;
      // 判断边界
      if (sx <= 0) {
        sx = 0;
      }
      if (sy <= 0) {
        sy = 0;
      }
      if (sx >= wi) {
        sx = wi;
      }
      if (sy >= he) {
        sy = he;
      }
      event.target.style.setProperty("left", sx + "px");
      event.target.style.setProperty("top", sy + "px");
      this.drawCropp();
    },
    // 裁剪框鼠标弹起事件和父盒子鼠标离开事件（为了注销裁剪盒子的父盒子的移动事件）
    croppMouse(up) {
      this.croppicture.onmousemove = null;
      if (up == "up") {
        this.confirmImage("up");
      }
    },

    /**
     * 裁剪框四个角移动事件集合
     */
    // 右下
    rightBottommousedown(e) {
      // 阻止冒泡，防止触发父组件的鼠标按下事件
      e.stopPropagation();
      const cropp = this.$refs["croppRef"];
      const cvs_mask = this.$refs["cvsmaskRef"];

      cropp.style.pointerEvents = "none";
      // 裁剪盒子相对于父元素的距离
      const croppLeft = cropp.offsetLeft;
      const croppTop = cropp.offsetTop;
      // 计算出按下去的角对于遮罩盒子的距离
      let rightLeft = e.target.offsetLeft + croppLeft + e.target.offsetWidth;
      let rightTop = e.target.offsetTop + croppTop + e.target.offsetHeight;
      cvs_mask.onmousemove = (event) => {
        let newWidth = cropp.offsetWidth + (event.offsetX - rightLeft);
        let newHeight = cropp.offsetHeight + (event.offsetY - rightTop);

        // cropp.style.transform = `translateX(-${(event.offsetX - rightLeft) / 2}px)`;
        cropp.style.width = newWidth + "px";
        cropp.style.height = newHeight + "px";
        this.drawCropp();
        rightLeft = event.offsetX;
        rightTop = event.offsetY;
      };
    },
    // 左上
    leftTopmousedown(e) {
      // 阻止冒泡，防止触发父组件的鼠标按下事件
      e.stopPropagation();
      const cropp = this.$refs["croppRef"];
      const cvs_mask = this.$refs["cvsmaskRef"];

      cropp.style.pointerEvents = "none";
      let leftLeft = e.target.offsetLeft + cropp.offsetLeft;
      let leftTop = e.target.offsetTop + cropp.offsetTop;
      cvs_mask.onmousemove = (event) => {
        const offsetLeft = event.offsetX;
        const offsetTop = event.offsetY;
        // 需要注意右边的坐标需要原来的减去现在的。因为是往左移坐标坐标是越来越小。
        let newWidth = cropp.offsetWidth + (leftLeft - offsetLeft);
        let newHeight = cropp.offsetHeight + (leftTop - offsetTop);

        cropp.style.left = offsetLeft + "px";
        cropp.style.top = offsetTop + "px";

        cropp.style.width = newWidth + "px";
        cropp.style.height = newHeight + "px";

        this.drawCropp();
        leftLeft = event.offsetX;
        leftTop = event.offsetY;
      };
    },
    // 左下
    leftBottommousedown(e) {
      // 阻止冒泡，防止触发父组件的鼠标按下事件
      e.stopPropagation();
      const cropp = this.$refs["croppRef"];
      const cvs_mask = this.$refs["cvsmaskRef"];

      cropp.style.pointerEvents = "none";
      let leftLeft = e.target.offsetLeft + cropp.offsetLeft;
      let leftTop =
        e.target.offsetTop + cropp.offsetTop + e.target.offsetHeight;
      cvs_mask.onmousemove = (event) => {
        const offsetLeft = event.offsetX;
        const offsetTop = event.offsetY;
        // 需要注意右边的坐标需要原来的减去现在的。因为是往左移坐标坐标是越来越小。
        let newWidth = cropp.offsetWidth + (leftLeft - offsetLeft);
        let newHeight = cropp.offsetHeight + (offsetTop - leftTop);

        cropp.style.left = offsetLeft + "px";
        cropp.style.bottom = offsetTop + "px";

        cropp.style.width = newWidth + "px";
        cropp.style.height = newHeight + "px";

        this.drawCropp();
        leftLeft = event.offsetX;
        leftTop = event.offsetY;
      };
    },
    // 右上
    rightTopmousedown(e) {
      // 阻止冒泡，防止触发父组件的鼠标按下事件
      e.stopPropagation();
      const cropp = this.$refs["croppRef"];
      const cvs_mask = this.$refs["cvsmaskRef"];

      cropp.style.pointerEvents = "none";
      // 裁剪盒子相对于父元素的距离
      const croppLeft = cropp.offsetLeft;
      const croppTop = cropp.offsetTop;
      // 计算出按下去的角对于遮罩盒子的距离
      let rightLeft = e.target.offsetLeft + croppLeft + e.target.offsetWidth;
      let rightTop = e.target.offsetTop + croppTop;
      cvs_mask.onmousemove = (event) => {
        let newWidth = cropp.offsetWidth + (event.offsetX - rightLeft);
        let newHeight = cropp.offsetHeight + (rightTop - event.offsetY);

        cropp.style.top = event.offsetY + "px";
        cropp.style.width = newWidth + "px";
        cropp.style.height = newHeight + "px";
        this.drawCropp();
        rightLeft = event.offsetX;
        rightTop = event.offsetY;
      };
    },

    /**
     * 拖拽图片
     */
    // 遮罩盒子鼠标按下事件（为了拖拽图片）
    canvasMousedown(e) {
      if (!this.isCropp) return;
      let beginX = e.offsetX;
      let beginY = e.offsetY;
      if (!this.imgIsDown(e.offsetX * this.ratio, e.offsetY * this.ratio)) {
        return;
      }
      /** @type {HTMLCanvasElement} */
      const cvs_mask = this.$refs["cvsmaskRef"];
      const cvs = this.$refs["cvsRef"];
      const cropp = this.$refs["croppRef"];
      const ctx = cvs.getContext("2d");
      cvs_mask.onmousemove = (event) => {
        // pointer-events 设置为none后 则鼠标事件将会穿透该元素并传递给下层盒子。
        // 此属性为none会禁止鼠标和滚轮等事件，记得改回来
        cropp.style.pointerEvents = "none";
        const x = event.offsetX;
        const y = event.offsetY;
        //算出来移动的像素（每次都是减去上次的值）
        var Mx = x - beginX + this.PO.px;
        var My = y - beginY + this.PO.py;
        this.repeatDraw(cvs, ctx, Mx, My);
        // 每次画完更新坐标
        this.PO.px = Mx;
        this.PO.py = My;
        beginX = x;
        beginY = y;
      };
    },

    /**
     * 遮罩盒子鼠标离开和弹起事件，为了注销遮罩盒子的移动事件
     */
    maskCanvasMouse(up) {
      const cvs_mask = this.$refs["cvsmaskRef"];
      const cropp = this.$refs["croppRef"];
      // pointer-events 设置为none后 则鼠标事件将会穿透该元素并传递给下层盒子。
      //（这里在鼠标离开或弹起后在改为auto，避免后续裁剪框没有鼠标事件）
      cropp.style.pointerEvents = "auto";
      cvs_mask.onmousemove = null;
      if (up == "up") {
        this.confirmImage("up");
      }
    },

    /**
     * 缩放图片
     */
    // 遮罩盒子鼠标滚轮事件（为了缩放图片）
    maskCanvasMouseWheel(event) {
      if (!this.isCropp) return;
      event.preventDefault();
      /** @type {HTMLCanvasElement} */
      const cvs = this.$refs["cvsRef"];
      const ctx = cvs.getContext("2d");
      if (event.wheelDelta >= 0) {
        //放大的倍数可以根据实际情况定义，可以丝滑一点，也可以控制最大放大倍数和最小倍数
        this.scale += this.scalenum;
      } else {
        this.scale -= this.scalenum;
      }
      // 计算放大之后的坐标
      let width = this.img.width * this.scale;
      let height = this.img.height * this.scale;
      let sx = this.PO.px - (width / 2 - this.beginImg.width / 2); //y坐标
      let sy = this.PO.py - (height / 2 - this.beginImg.height / 2); //y坐标

      this.repeatDraw(cvs, ctx, sx, sy);

      // 缩放完更新坐标
      this.PO.px = sx;
      this.PO.py = sy;

      this.beginImg.width = width;
      this.beginImg.height = height;
    },

    /**
     * 移动端事件
     */
    maskCanvasTouchstart(e) {
      e.preventDefault();
      if (!this.isCropp) return;
      const cropp = this.$refs["croppRef"];
      cropp.style.pointerEvents = "none";
      if (e.targetTouches.length === 1) {
        const one = e.targetTouches[0];
        let beginX = one.clientX;
        let beginY = one.clientY;
        // if (!this.imgIsDown(one.clientX, one.clientY)) {
        //   return;
        // }
        /** @type {HTMLCanvasElement} */
        const cvs_mask = this.$refs["cvsmaskRef"];
        const cvs = this.$refs["cvsRef"];
        const ctx = cvs.getContext("2d");
        cvs_mask.ontouchmove = (event) => {
          const evone = event.targetTouches[0];
          const x = evone.clientX;
          const y = evone.clientY;
          //算出来移动的像素（每次都是减去上次的值）
          var Mx = x - beginX + this.PO.px;
          var My = y - beginY + this.PO.py;
          this.repeatDraw(cvs, ctx, Mx, My);
          // 每次画完更新坐标
          this.PO.px = Mx;
          this.PO.py = My;
          beginX = x;
          beginY = y;
        };
      } else if (e.touches.length === 2) {
        // 计算初始距离
        let initialDistance = this.getDistance(e.touches);

        /** @type {HTMLCanvasElement} */
        const cvs_mask = this.$refs["cvsmaskRef"];
        const cvs = this.$refs["cvsRef"];
        const ctx = cvs.getContext("2d");
        cvs_mask.ontouchmove = (event) => {
          // 计算当前距离，并与初始距离相比较
          let currentDistance = this.getDistance(event.touches);
          const distanceChange = currentDistance - initialDistance;

          // 如果移动距离大于一定阈值，则判断为捏合操作
          if (Math.abs(distanceChange) > 10) {
            if (distanceChange < 0) {
              // 手指向内捏合，触发缩小事件
              this.scale -= this.scalenum;
            } else {
              // 手指向外张开，触发放大事件
              this.scale += this.scalenum;
            }
            // 更新初始距离以支持多次捏合操作
            initialDistance = currentDistance;
          }

          // 计算放大之后的坐标
          // (核心思想就是拿到当前坐标和放大后图片的一半宽度相加。)
          var width = this.img.width * this.scale;
          var height = this.img.height * this.scale;
          var sx = this.PO.px - (width / 2 - this.beginImg.width / 2); //y坐标
          var sy = this.PO.py - (height / 2 - this.beginImg.height / 2); //y坐标
          this.repeatDraw(cvs, ctx, sx, sy);
          // 缩放完更新坐标
          this.PO.px = sx;
          this.PO.py = sy;
          this.beginImg.width = width;
          this.beginImg.height = height;
        };
      }
    },
    // 移动端手指离开事件
    maskCanvasTouchend() {
      const cvs_mask = this.$refs["cvsmaskRef"];
      cvs_mask.ontouchmove = null;
    },
    // 计算当前两个触摸点之间的距离
    getDistance(touches) {
      const dx = touches[0].clientX - touches[1].clientX;
      const dy = touches[0].clientY - touches[1].clientY;
      return Math.sqrt(dx * dx + dy * dy);
    },

    /**
     * 辅助函数
     */
    // 判断按下去的坐标是否在图片上
    imgIsDown(x, y) {
      const imgWidth = this.img && this.img.width;
      const imgHeight = this.img && this.img.height;
      const { px, py } = this.PO;
      return (
        x > px &&
        x < px + imgWidth * this.scale &&
        y > py &&
        y < py + imgHeight * this.scale
      );
    },
    // 重新绘制图片的辅助函数
    repeatDraw(cvs, ctx, sx, sy) {
      const imgWidth = this.img.width;
      const imgHeight = this.img.height;
      ctx.clearRect(0, 0, cvs.width, cvs.height);
      ctx.drawImage(
        this.img,
        sx,
        sy,
        imgWidth * this.scale,
        imgHeight * this.scale
      );
    },
    // 裁剪完成生成图片
    confirmImage(type) {
      if (!this.isCropp) return null;
      return new Promise((reslove, reject) => {
        let downCanvas = document.createElement("canvas");
        let downCtx = downCanvas.getContext("2d");
        const cvs = this.$refs["cvsRef"];

        const cropp = this.$refs["croppRef"];

        const croppWidth = cropp.offsetWidth * this.ratio;
        const croppHeight = cropp.offsetHeight * this.ratio;

        downCanvas.width = croppWidth;
        downCanvas.height = croppHeight;
        downCtx.drawImage(
          cvs,
          cropp.offsetLeft * this.ratio,
          cropp.offsetTop * this.ratio,
          croppWidth,
          croppHeight,
          0,
          0,
          croppWidth,
          croppHeight
        );
        if (type == "confirm") {
          downCanvas.toBlob((blob) => {
            // 将 Blob 对象转换为 File 对象
            const file = new File(
              [blob],
              Math.random().toString().slice(2) + ".png",
              { type: "image/png" }
            );
            downCanvas.remove();
            reslove(file);
          });
        } else {
          this.$emit("moveupCropp", downCanvas.toDataURL("image/png", 0.6));
        }
      });
    },
    clearCanvas() {
      const cvs = this.$refs["cvsRef"];
      const canvasMask = this.$refs["cvsmaskRef"];
      this.$nextTick(() => {
        const ctx = cvs?.getContext("2d");
        const mask_ctx = canvasMask?.getContext("2d");
        ctx.clearRect(0, 0, cvs.width, cvs.height);
        mask_ctx.clearRect(0, 0, canvasMask.width, canvasMask.height);
        this.isCropp = false;
      });
    },
  },
  beforeUnmount() {
    const cvs = this.$refs["cvsRef"];
    const cvs_mask = this.$refs["cvsmaskRef"];
    const cropp = this.$refs["croppRef"];
    cvs.onmouseup = null;
    cvs.onmousemove = null;
    cvs.onmousedown = null;
    cvs.onmouseleave = null;

    cvs_mask.onmouseup = null;
    cvs_mask.onmousemove = null;
    cvs_mask.onmousedown = null;
    cvs_mask.onmouseleave = null;

    cropp.onmouseup = null;
    cropp.onmousemove = null;
    cropp.onmousedown = null;
    cropp.onmouseleave = null;
  },
  beforeDestroy() {},
};
</script>

<style scoped>
.croppicture {
  position: relative;
  background-size: 30px 30px;
  background-position: 0 0, 15px 15px;
  text-align: initial;
}
#cvs {
  position: absolute;
  z-index: 9;
  image-rendering: optimizeSpeed;
  image-rendering: -moz-crisp-edges;
  image-rendering: -webkit-optimize-contrast;
  image-rendering: optimize-contrast;
  -ms-interpolation-mode: nearest-neighbor;
  image-rendering: pixelated;
}

#cvs_mask {
  position: absolute;
  z-index: 99;
}

.cropp {
  --borderStyle: 2px solid #fff;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  z-index: 999;
  width: 200px;
  height: 200px;
  box-sizing: border-box;
  /* border: 1px solid red; */
}

.cropp .right_top,
.right_bottom,
.left_top,
.left_bottom {
  position: absolute;
  width: 20px;
  height: 20px;
  z-index: 9999;
  box-sizing: border-box;
  /* background-color: red; */
}
.right_top {
  right: 0;
  top: 0;
  border-top: var(--borderStyle);
  border-right: var(--borderStyle);
  cursor: nesw-resize;
}
.right_bottom {
  right: 0;
  bottom: 0;
  border-right: var(--borderStyle);
  border-bottom: var(--borderStyle);
  cursor: nwse-resize;
}
.left_top {
  left: 0;
  top: 0;
  border-left: var(--borderStyle);
  border-top: var(--borderStyle);
  cursor: nwse-resize;
}
.left_bottom {
  left: 0;
  bottom: 0;
  border-left: var(--borderStyle);
  border-bottom: var(--borderStyle);
  cursor: nesw-resize;
}

.cropp_button {
  position: absolute;
  right: 0;
  bottom: 0;
  color: #fff;
  border: 1px solid #fff !important;
  padding: 10px 20px;
  z-index: 999;
  background-color: transparent;
  cursor: pointer;
}
.cropp_button_cancel {
  position: absolute;
  left: 0;
  bottom: 0;
  color: #fff;
  border: 1px solid #fff !important;
  padding: 10px 20px;
  z-index: 999;
  background-color: transparent;
  cursor: pointer;
}
</style>
