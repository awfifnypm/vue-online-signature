<template>
    <canvas v-if="esignReset" ref="canvasRef" @mousedown="onMouseDown" @mousemove="onMouseMove" @mouseup="onMouseUp" @touchstart="onTouchStart" @touchmove="onTouchMove" @touchend="onTouchEnd"></canvas>
</template>

<script>
export default {
  props: {
    width: {
      type: Number,
      default: 0
    },
    height: {
      type: Number,
      default: 0
    },
    lineWidth: {
      type: Number,
      default: 4
    },
    lineColor: {
      type: String,
      default: '#000000'
    },
    canvasBack: {
      type: String,
      default: ''
    },
    isCrop: {
      type: Boolean,
      default: false
    },
    edg: {
      type: Number,
      default: 0
    },
    fullScreen: {
      type: Boolean,
      default: false
    },
    domId: {
      type: String,
      default: ''
    },
    imgBack: {
      type: String,
      default: ''
    },
    isRepeat: {
      type: String,
      default: 'no-repeat'
    },
    noRotation: {
      type: Boolean,
      default: false
    },
    imgType: {
      type: String,
      default: 'image/png'
    },
    backIsCenter: { // domId和宽度有效
      type: Boolean,
      default: false
    },
    verticalDeductWidth: {
      type: Number,
      default: 0
    },
    verticalDeductHeight: {
      type: Number,
      default: 0
    },
    acrossDeductWidth: {
      type: Number,
      default: 0
    },
    acrossDeductHeight: {
      type: Number,
      default: 0
    }
  },
  data () {
    return {
      hasDrew: false,
      isOrientationchange: false,
      esignReset: true,
      resultImg: '',
      points: [],
      canvasTxt: null,
      canvasRef: null,
      cropCanvas: null,
      cropCanvasTxt: null,
      startX: 0,
      startY: 0,
      sratio: 1,
      isDrawing: false,
      imgBackDom: null,
      canvasBackDom: null,
      isload: false,
      screenPatams: {
        width: 0,
        height: 0
      },
      domPatams: {
        width: 0,
        height: 0
      }
    }
  },
  watch: {
    canvasBackground (newVal) {
      this.$nextTick(() => {
        this.$refs.canvasRef && (this.$refs.canvasRef.style.background = newVal)
      })
    },
    hasDrew (newVal) {
      this.$emit('onDrawingStatus', newVal)
    }
  },
  computed: {
    ratio () {
      return (this.domPatams.height ? this.domPatams.height : this.fullScreen ? this.screenPatams.height : this.height) / (this.domPatams.width ? this.domPatams.width : this.fullScreen ? this.screenPatams.width : this.width)
    },
    canvasBackground () {
      return this.canvasBack ? this.canvasBack : 'rgba(255, 255, 255, 0)'
    }
  },
  methods: {
    getSizeRatio () {
      return !this.fullScreen && this.backIsCenter
    },
    setCanvasImageBack (status) {
      const canvas = this.$refs.canvasRef
      let pat = this.canvasTxt.createPattern(this.canvasBackDom, (this.isRepeat || 'no-repeat'))
      this.canvasTxt.rect(0, 0, canvas.width, canvas.height)
      this.canvasTxt.fillStyle = pat
      this.canvasTxt.fill()
      if (status) {
        this.autoDraw()
      }
    },
    setCanvasBack (status) {
      const canvas = this.$refs.canvasRef
      if (this.canvasBack && this.canvasBackDom && this.isImgaes(this.canvasBack)) {
        this.setCanvasImageBack(status)
      } else {
        canvas.style.background = this.canvasBackground
      }
    },
    resizeHandler (status) {
      if (this.isOrientationchange) return false
      const canvas = this.$refs.canvasRef
      canvas.style.width = (this.domPatams.width ? this.domPatams.width : this.fullScreen ? this.screenPatams.width : this.width) + 'px'
      const realw = parseFloat(window.getComputedStyle(canvas).width)
      canvas.style.height = this.ratio * realw + 'px'
      this.canvasTxt = canvas.getContext('2d')
      this.canvasTxt.scale(1 * this.sratio, 1 * this.sratio)
      this.sratio = realw / (this.domPatams.width ? this.domPatams.width : this.fullScreen ? this.screenPatams.width : this.width)
      this.canvasTxt.scale(1 / this.sratio, 1 / this.sratio)
      if (this.canvasBack) {
        let IntervaId = setInterval(() => {
          if (this.canvasBackDom || !this.isImgaes(this.canvasBack)) {
            this.setCanvasBack(status)
            clearInterval(IntervaId)
          }
        }, 100)
      } else {
        if (status) {
          this.autoDraw()
        }
      }
    },
    orientationchangeEvent () {
      let directionWidth = window.orientation == 0 || window.orientation == 180 ? this.verticalDeductWidth : this.acrossDeductWidth
      let directionHeight = window.orientation == 0 || window.orientation == 180 ? this.verticalDeductHeight : this.acrossDeductHeight
      this.screenPatams.width = window.navigator.platform.indexOf('Win') || window.navigator.platform.indexOf('Mac') ? (document.body.clientHeight || document.body.offsetHeight) - directionWidth : (document.body.clientWidth || document.body.offsetWidth) - directionWidth
      this.screenPatams.height = window.navigator.platform.indexOf('Win') || window.navigator.platform.indexOf('Mac') ? (document.body.clientWidth || document.body.offsetWidth) - directionHeight : (document.body.clientHeight || document.body.offsetHeight) - directionHeight
      this.getImages()
      this.isOrientationchange = true
      this.esignReset = false
      let setIntervalId = setInterval(() => {
        if (this.isload) {
          clearInterval(setIntervalId)
          this.esignReset = true
          this.isOrientationchange = false
          this.$nextTick(() => {
            this.getDomSize()
            this.resizeHandler(true)
          })
        }
      }, 100)
    },
    isImgaes (params) {
      let imgType = ['.jpeg', '.bmp', '.jpg', '.gif', '.webp', '.pcx', '.tif', '.tga', '.exif', '.fpx', '.svg', '.cdr', '.pcd', '.dxf', '.ufo', '.eps', '.ai', '.png', '.hdri', '.raw', '.wmf', '.flic', '.emf', '.ico', '.avif', '.apng']
      let regex = /^\s*data:([a-z]+\/[a-z0-9-+.]+(;[a-z-]+=[a-z0-9-]+)?)?(;base64)?,([a-z0-9!$&',()*+;=\-._~:@\/?%\s]*?)\s*$/i
      let status = params.includes('http://') || params.includes('https://') || regex.test(params) || imgType.some(item => params.includes(item))
      return status
    },
    rotateBase64Img (src, edg, type = 'not') {
      let _this = this
      return new Promise((resolve, reject) => {
        let canvas = document.createElement('canvas')
        let ctx = canvas.getContext('2d')
        let imgW// 图片宽度
        let imgH// 图片高度
        let size// canvas初始大小
        if (edg % 90 != 0) {
          reject('旋转角度必须是90的倍数!')
          throw '旋转角度必须是90的倍数!'
        }
        (edg < 0) && (edg = (edg % 360) + 360)
        const quadrant = (edg / 90) % 4 // 旋转象限
        const cutCoor = { sx: 0, sy: 0, ex: 0, ey: 0 } // 裁剪坐标
        let image = new Image()
        image.crossOrigin = 'anonymous'
        image.src = src
        image.onload = function () {
          imgW = image.width
          imgH = image.height
          size = imgW > imgH ? imgW : imgH
          canvas.width = size * 2
          canvas.height = size * 2
          switch (quadrant) {
            case 0:
              let domWidth = 0
              let domHeight = 0
              if (_this.domId) {
                let dom = document.getElementById(_this.domId)
                domWidth = dom.clientWidth || dom.offsetWidth
                domHeight = dom.clientHeight || dom.offsetHeight
              }
              let Xwidth = _this.domId ? imgW > domWidth && domWidth : _this.width ? imgW > _this.width && _this.width : false
              console.log(Xwidth, 'Xwidth')
              cutCoor.sx = _this.getSizeRatio() && type == 'init' && Xwidth ? size + ((imgW - Xwidth) / 2) : size
              cutCoor.sy = size
              cutCoor.ex = size + imgW
              cutCoor.ey = size + imgH
              break
            case 1:
              let Cwidth = (_this.domPatams.width ? _this.domPatams.width : _this.fullScreen ? _this.screenPatams.width : _this.width)
              let ratio = _this.getSizeRatio() && type == 'init'
              cutCoor.sx = ratio ? size - _this.height : window.orientation == 0 || window.orientation == 180 ? size - Cwidth : size - imgH
              cutCoor.sy = size
              cutCoor.ex = size
              cutCoor.ey = size + imgW
              break
            case 2:
              cutCoor.sx = size - imgW
              cutCoor.sy = size - imgH
              cutCoor.ex = size
              cutCoor.ey = size
              break
            case 3:
              cutCoor.sx = size
              cutCoor.sy = size - imgW
              cutCoor.ex = size + imgH
              cutCoor.ey = size + imgW
              break
          }
          ctx.translate(size, size)
          ctx.rotate(edg * Math.PI / 180)
          ctx.drawImage(image, 0, 0)
          var imgData = ctx.getImageData(cutCoor.sx, cutCoor.sy, cutCoor.ex, cutCoor.ey)
          if (quadrant % 2 == 0) {
            canvas.width = imgW
            canvas.height = imgH
          } else {
            canvas.width = imgH
            canvas.height = imgW
          }
          // putImageData() 将图像数据放回画布
          ctx.putImageData(imgData, 0, 0)
          resolve(canvas.toDataURL(this.imgType))
        }
      })
    },
    reset () {
      let canvas = this.$refs.canvasRef
      this.canvasTxt.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
      )
      this.points = []
      this.hasDrew = false
      this.resultImg = ''
    },
    getCropArea (imgData) {
      if (this.imgBack) {
        this.cropCanvas = document.createElement('canvas')
        if (this.domId) {
          let dom = document.getElementById(this.domId)
          let domWidth = dom ? dom.clientWidth || dom.offsetWidth : this.fullScreen ? this.screenPatams.width : this.width
          let domHeight = dom ? dom.clientHeight || dom.offsetHeight : this.fullScreen ? this.screenPatams.height : this.height
          this.cropCanvas.height = domHeight
          this.cropCanvas.width = domWidth
          this.domPatams.width = domWidth
          this.domPatams.height = domHeight
        } else {
          this.cropCanvas.height = this.fullScreen ? this.screenPatams.height : this.height
          this.cropCanvas.width = this.fullScreen ? this.screenPatams.width : this.width
        }
        this.cropCanvasTxt = this.cropCanvas.getContext('2d')
        if (this.isImgaes(this.imgBack)) {
          let pat = this.cropCanvasTxt.createPattern(this.imgBackDom, (this.isRepeat || 'no-repeat'))
          this.cropCanvasTxt.rect(0, 0, this.cropCanvas.width, this.cropCanvas.height)
          this.cropCanvasTxt.fillStyle = pat
          this.cropCanvasTxt.fill()
        } else if (!this.isImgaes(this.imgBack)) {
          this.cropCanvasTxt.fillStyle = this.imgBack
          this.cropCanvasTxt.fillRect(0, 0, this.cropCanvas.width, this.cropCanvas.height)
        }
        this.autoDraw(this.cropCanvas, this.cropCanvasTxt)
      }
      let canvas = this.cropCanvas || this.$refs.canvasRef
      let topX = canvas.width
      var btmX = 0
      var topY = canvas.height
      var btnY = 0
      for (var i = 0; i < canvas.width; i++) {
        for (var j = 0; j < canvas.height; j++) {
          var pos = (i + canvas.width * j) * 4
          if (imgData[pos] > 0 || imgData[pos + 1] > 0 || imgData[pos + 2] || imgData[pos + 3] > 0) {
            btnY = Math.max(j, btnY)
            btmX = Math.max(i, btmX)
            topY = Math.min(j, topY)
            topX = Math.min(i, topX)
          }
        }
      }
      topX++
      btmX++
      topY++
      btnY++
      const data = [topX, topY, btmX, btnY]
      return data
    },
    getDirection () {
      return window.orientation == 90 || window.orientation == -90 ? 'across' : 'vertical'
    },
    // pc
    onMouseDown (e) {
      e = e || event
      e.preventDefault()
      this.isDrawing = true
      this.hasDrew = true
      let params = {
        x: e.offsetX,
        y: e.offsetY,
        direction: this.getDirection()
      }
      this.drawStart(params)
      this.$emit('onMouseDown', e)
    },
    onMouseMove (e) {
      e = e || event
      e.preventDefault()
      if (this.isDrawing) {
        let obj = {
          x: e.offsetX,
          y: e.offsetY,
          direction: this.getDirection()
        }
        this.drawMove(obj)
      }
      this.$emit('onMouseMove', e)
    },
    onMouseUp (e) {
      e = e || event
      e.preventDefault()
      let obj = {
        x: e.offsetX,
        y: e.offsetY,
        direction: this.getDirection()
      }
      this.drawEnd(obj)
      this.isDrawing = false
      this.$emit('onMouseUp', e)
    },
    // mobile
    onTouchStart (e) {
      e = e || event
      e.preventDefault()
      this.hasDrew = true
      if (e.touches.length === 1) {
        let canvas = this.$refs.canvasRef
        let obj = {
          x: e.targetTouches[0].clientX - canvas.getBoundingClientRect().left,
          y: e.targetTouches[0].clientY - canvas.getBoundingClientRect().top,
          direction: this.getDirection()
        }
        this.drawStart(obj)
      }
      this.$emit('onTouchStart', e)
    },
    onTouchMove (e) {
      e = e || event
      e.preventDefault()
      if (e.touches.length === 1) {
        let canvas = this.$refs.canvasRef
        let obj = {
          x: e.targetTouches[0].clientX - canvas.getBoundingClientRect().left,
          y: e.targetTouches[0].clientY - canvas.getBoundingClientRect().top,
          direction: this.getDirection()
        }
        this.drawMove(obj)
      }
      this.$emit('onTouchMove', e)
    },
    onTouchEnd (e) {
      e = e || event
      e.preventDefault()
      if (e.touches.length === 1) {
        let canvas = this.$refs.canvasRef
        let obj = {
          x: e.targetTouches[0].clientX - canvas.getBoundingClientRect().left,
          y: e.targetTouches[0].clientY - canvas.getBoundingClientRect().top,
          direction: this.getDirection()
        }
        this.drawEnd(obj)
      } else {
        this.points.push({ x: -1, y: -1, direction: this.getDirection() })
      }
      this.$emit('onTouchEnd', e)
    },
    // 绘制
    drawStart (params) {
      this.startX = params.x
      this.startY = params.y
      this.canvasTxt.beginPath()
      this.canvasTxt.moveTo(this.startX, this.startY)
      this.canvasTxt.lineTo(params.x, params.y)
      this.canvasTxt.lineCap = 'round'
      this.canvasTxt.lineJoin = 'round'
      this.canvasTxt.lineWidth = this.lineWidth * this.sratio
      this.canvasTxt.stroke()
      this.canvasTxt.closePath()
      this.points.push(params)
    },
    drawMove (params) {
      this.canvasTxt.beginPath()
      this.canvasTxt.moveTo(this.startX, this.startY)
      this.canvasTxt.lineTo(params.x, params.y)
      this.canvasTxt.strokeStyle = this.lineColor
      this.canvasTxt.lineWidth = this.lineWidth * this.sratio
      this.canvasTxt.lineCap = 'round'
      this.canvasTxt.lineJoin = 'round'
      this.canvasTxt.stroke()
      this.canvasTxt.closePath()
      this.startY = params.y
      this.startX = params.x
      this.points.push(params)
    },
    drawEnd (params) {
      this.canvasTxt.beginPath()
      this.canvasTxt.moveTo(this.startX, this.startY)
      this.canvasTxt.lineTo(params.x, params.y)
      this.canvasTxt.lineCap = 'round'
      this.canvasTxt.lineJoin = 'round'
      this.canvasTxt.stroke()
      this.canvasTxt.closePath()
      this.points.push(params)
      this.points.push({ x: -1, y: -1 })
    },
    autoDraw (canvasRef, canvas2d) {
      if (this.points && this.points.length) {
        let canvas = canvasRef || this.$refs.canvasRef
        let canvasText = canvas2d || this.canvasTxt
        this.points.reduce((acc, cur) => {
          if (cur.x != -1 && cur.y != -1 && acc.x != -1 && acc.y != -1) {
            canvasText.beginPath()
            let position = { accX: acc.x, accY: acc.y, curX: cur.x, curY: cur.y }
            if (this.fullScreen) {
              if ((window.orientation == 0 || window.orientation == 180) && acc.direction == 'across' && cur.direction == 'across') { // 横屏笔画横屏转竖屏
                position = { accX: canvas.width - acc.y, accY: acc.x, curX: canvas.width - cur.y, curY: cur.x }
              } else if ((window.orientation == 90 || window.orientation == -90) && acc.direction == 'vertical' && cur.direction == 'vertical') { // 竖屏笔画竖屏转横屏
                position = { accX: acc.y, accY: canvas.height - acc.x, curX: cur.y, curY: canvas.height - cur.x }
              }
            }
            canvasText.moveTo(position.accX, position.accY)
            canvasText.lineTo(position.curX, position.curY)
            canvasText.strokeStyle = this.lineColor
            canvasText.lineWidth = this.lineWidth * this.sratio
            canvasText.lineCap = 'round'
            canvasText.lineJoin = 'round'
            canvasText.stroke()
            canvasText.closePath()
          }
          return cur
        })
      }
    },
    async getImages (status = 'init') {
      this.isload = false
      let edg = window.orientation == 0 || window.orientation == 180 ? 90 : 0
      let ratio = this.fullScreen ? edg : 0
      if (this.isImgaes(this.imgBack)) {
        let res = await this.rotateBase64Img(this.imgBack, ratio, status)
        if (res) {
          this.imgBackDom = new Image()
          this.imgBackDom.crossOrigin = 'anonymous'
          this.imgBackDom.src = res
        }
      }
      if (this.isImgaes(this.canvasBack)) {
        let res = await this.rotateBase64Img(this.canvasBack, ratio, status)
        if (res) {
          this.canvasBackDom = new Image()
          this.canvasBackDom.crossOrigin = 'anonymous'
          this.canvasBackDom.src = res
          this.canvasBackDom.onload = () => {
            this.isload = true
          }
        }
      }
    },
    // 操作
    confirm () {
      return new Promise((resolve, reject) => {
        if (!this.hasDrew) {
          reject(`Warning: Not Signned!`)
          return
        }
        let canvas = this.$refs.canvasRef
        let resImgData = this.canvasTxt.getImageData(0, 0, canvas.width, canvas.height)
        this.canvasTxt.globalCompositeOperation = 'destination-over'
        if (this.canvasBack && this.imgBack) {
          this.canvasTxt.clearRect(0, 0, canvas.width, canvas.height)
          this.autoDraw()
        }
        // canvas背景图没有的情况下才生成图片背景图
        if (this.imgBack && this.isImgaes(this.imgBack)) {
          let pat = this.canvasTxt.createPattern(this.imgBackDom, (this.isRepeat || 'no-repeat'))
          this.canvasTxt.rect(0, 0, canvas.width, canvas.height)
          this.canvasTxt.fillStyle = pat
          this.canvasTxt.fill()
        } else if (this.imgBack && !this.isImgaes(this.imgBack)) {
          this.canvasTxt.fillStyle = this.imgBack
          this.canvasTxt.fillRect(0, 0, canvas.width, canvas.height)
        }
        this.resultImg = canvas.toDataURL()
        let resultImgs = this.resultImg
        this.canvasTxt.clearRect(0, 0, canvas.width, canvas.height)
        this.canvasTxt.putImageData(resImgData, 0, 0)
        this.canvasTxt.globalCompositeOperation = 'source-over'
        if (this.isCrop) {
          const crop_area = this.getCropArea(resImgData.data)
          let crop_canvas = document.createElement('canvas')
          const crop_ctx = crop_canvas.getContext('2d')
          crop_canvas.width = crop_area[2] - crop_area[0]
          crop_canvas.height = crop_area[3] - crop_area[1]
          const crop_imgData = (this.cropCanvasTxt || this.canvasTxt).getImageData(...crop_area)
          crop_ctx.globalCompositeOperation = 'destination-over'
          crop_ctx.putImageData(crop_imgData, 0, 0)
          resultImgs = crop_canvas.toDataURL()
          crop_canvas = null
        }
        let edg = (this.fullScreen && ((window.orientation == 0 || window.orientation == 180) || ((window.orientation == 90 || window.orientation == -90) && !this.noRotation))) || (window.orientation === undefined) && this.noRotation ? this.edg : 0
        this.rotateBase64Img(resultImgs, edg)
          .then(base64 => {
            resolve(base64)
          })
      })
    },
    getDomSize () {
      const canvas = this.$refs.canvasRef
      if (this.domId) {
        let dom = document.getElementById(this.domId)
        let domWidth = dom ? dom.clientWidth || dom.offsetWidth : this.fullScreen ? this.screenPatams.width : this.width
        let domHeight = dom ? dom.clientHeight || dom.offsetHeight : this.fullScreen ? this.screenPatams.height : this.height
        canvas.height = domHeight
        canvas.width = domWidth
        this.domPatams.width = domWidth
        this.domPatams.height = domHeight
      } else {
        canvas.height = this.fullScreen ? this.screenPatams.height : this.height
        canvas.width = this.fullScreen ? this.screenPatams.width : this.width
      }
    }
  },
  mounted () {
    let directionWidth = window.orientation == 0 || window.orientation == 180 ? this.verticalDeductWidth : this.acrossDeductWidth
    let directionHeight = window.orientation == 0 || window.orientation == 180 ? this.verticalDeductHeight : this.acrossDeductHeight
    this.screenPatams.width = (document.body.clientWidth || document.body.offsetWidth) - directionWidth
    this.screenPatams.height = (document.body.clientHeight || document.body.offsetHeight) - directionHeight
    this.getImages()
    this.getDomSize()
    window.addEventListener('orientationchange', this.orientationchangeEvent)
    this.resizeHandler(false)
    // 在画板以外松开鼠标后冻结画笔
    document.onmouseup = () => {
      this.isDrawing = false
    }
  },
  beforeMount () {
    if (this.fullScreen) {
      let bodyDom = document.getElementsByTagName('body')
      bodyDom[0].style = 'height: 100vh'
    }
    window.addEventListener('resize', this.resizeHandler)
  },
  beforeDestroy () {
    if (this.fullScreen) {
      let bodyDom = document.getElementsByTagName('body')
      bodyDom[0].style = ''
    }
    window.removeEventListener('resize', this.resizeHandler)
    window.removeEventListener('orientationchange', this.orientationchangeEvent)
  }
}
</script>

<style scoped>
canvas{
  max-width: 100%;
  display: block;
}
</style>
