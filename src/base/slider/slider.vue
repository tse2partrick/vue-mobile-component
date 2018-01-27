<template>
  <div class="slider" ref="slider">
    <div class="slider-group" ref="sliderGroup" @touchstart="onTouchStart" @touchmove="onTouchMove" @touchend="onTouchEnd">
      <slot></slot>
    </div>
    <ul class="dots">
      <li class="dot" :class="{current: index === currentPage}" v-for="(dot, index) in dots" :key="dot" :data-page="index">{{dot}}</li>
    </ul>
  </div>
</template>

<script>
  export default {
    props: {
      threshold: {
        type: Number,
        default: 0.3
      },
      interval: {
        type: Number,
        default: 4000,
        validator(val) {
          return val >= 1000
        }
      }
    },
    data() {
      return {
        dots: [],
        currentPage: 0
      }
    },
    created() {
      this.touch = {}
      this.touch.page = 0
      this.touch.lastX = 0
    },
    mounted() {
      setTimeout(() => {
        this._setSliderGroupWidth()
        this._initDots()
        this._autoPlay()

        window.addEventListener('transitionend', this._transitionEnd)

        window.addEventListener('resize', this._resize)
      }, 20)
    },
    activated() {
      this._autoPlay()
    },
    deactivated() {
      clearTimeout(this.playTimer)
      window.removeEventListener('transitionend', this._transitionEnd)
    },
    beforeDestroyed() {
      clearTimeout(this.playTimer)
      window.removeEventListener('transitionend', this._transitionEnd)
    },
    methods: {
      onTouchStart(e) {
        clearTimeout(this.playTimer)
        this.touch.start = true
        this.touch.pos = ''

        this.touch.x1 = e.touches[0].pageX
      },
      onTouchMove(e) {
        if (!this.touch.start) {
          return
        }
        this.touch.move = true

        this.touch.x2 = e.touches[0].pageX

        this.touch.delta = this.touch.x2 - this.touch.x1 + this.touch.lastX

        this.$refs.sliderGroup.style.transition = ''
        this.$refs.sliderGroup.style.transform = `translate3d(${this.touch.delta}px, 0, 0)`

        let delta = this.touch.x2 - this.touch.x1
        if (delta > this.threshold * this.sliderWidth) {
          this.touch.pos = 'right'
        } else if (delta < 0 && Math.abs(delta) > this.threshold * this.sliderWidth) {
          this.touch.pos = 'left'
        }
      },
      onTouchEnd() {
        if (!this.touch.move) {
          return
        }

        this.$refs.sliderGroup.style.transition = 'all 0.3s'

        if (this.touch.pos === 'left') {
          this._moveRight()
        } else if (this.touch.pos === 'right') {
          this._moveLeft()
        } else {
          this._notMove()
        }

        this.touch.start = false
        this.touch.move = false
      },
      _transitionEnd() {
        this.$refs.sliderGroup.style.transition = ''
        if (this.touch.crossing) {
          let x = this.touch.imgWidth[this.currentPage]
          this.$refs.sliderGroup.style.transform = `translate3d(${x}px, 0, 0)`
          this.touch.crossing = false
        }
        this._autoPlay()
      },
      _autoPlay() {
        clearTimeout(this.playTimer)
        if (this.touch.start) {
          return
        }
        this.playTimer = setTimeout(() => {
          this.$refs.sliderGroup.style.transition = 'all 0.3s'
          this._moveRight()
        }, this.interval)
      },
      _resize() {
        clearTimeout(this.playTimer)
        this.$refs.sliderGroup.style.transition = ''
        this.sliderWidth = this.$refs.slider.clientWidth
        let width = 0
        this.touch.imgWidth = []
        let items = document.querySelectorAll('.slider-item')
        for (let i = 0; i < items.length; i++) {
          items[i].style.width = this.sliderWidth + 'px'

          width += this.sliderWidth
          this.touch.imgWidth.push(-width)
        }

        let imgStart = document.querySelector('.copied-start')
        let imgEnd = document.querySelector('.copied-end')
        imgStart.style.width = this.sliderWidth + 'px'
        imgEnd.style.width = this.sliderWidth + 'px'

        width += this.sliderWidth * 2
        this.$refs.sliderGroup.style.width = width + 'px'

        this.$refs.sliderGroup.style.transform = `translate3d(${this.touch.imgWidth[this.currentPage]}px, 0, 0)`
        this.touch.lastX = this.touch.imgWidth[this.currentPage]

        this._autoPlay()
      },
      _moveLeft() {
        let x
        if (this.currentPage - 1 < 0) {
          this.currentPage = this.dots.length - 1
          this.touch.crossing = true
          x = 0
          this.$refs.sliderGroup.style.transform = `translate3d(${x}px, 0, 0)`
          this.touch.lastX = this.touch.imgWidth[this.currentPage]
          return
        } else {
          this.currentPage -= 1
        }
        x = this.touch.imgWidth[this.currentPage]
        this.$refs.sliderGroup.style.transform = `translate3d(${x}px, 0, 0)`
        this.touch.lastX = x
      },
      _moveRight() {
        let x
        if (this.currentPage + 1 >= this.dots.length) {
          this.currentPage = 0
          this.touch.crossing = true
          x = this.touch.imgWidth[0] + this.touch.imgWidth[this.touch.imgWidth.length - 1]
          this.$refs.sliderGroup.style.transform = `translate3d(${x}px, 0, 0)`
          this.touch.lastX = this.touch.imgWidth[this.currentPage]
          return
        } else {
          this.currentPage += 1
        }
        x = this.touch.imgWidth[this.currentPage]
        this.$refs.sliderGroup.style.transform = `translate3d(${x}px, 0, 0)`
        this.touch.lastX = x
      },
      _notMove() {
        let x
        x = this.touch.imgWidth[this.currentPage]
        this.$refs.sliderGroup.style.transform = `translate3d(${x}px, 0, 0)`
        this.touch.lastX = x
      },
      _setSliderGroupWidth() {
        this.sliderWidth = this.$refs.slider.clientWidth

        this.children = this.$refs.sliderGroup.children

        let width = 0
        this.touch.imgWidth = []
        for (let i = 0; i < this.children.length; i++) {
          this.children[i].className = 'slider-item'
          this.children[i].style.width = this.sliderWidth + 'px'
          this.children[i].style.display = 'inline-block'
          this.children[i].style.height = '100%'
          width += this.sliderWidth

          this.touch.imgWidth.push(-width)
        }

        setTimeout(() => {
          let imgStart = this.children[this.children.length - 1].cloneNode(true)
          let imgEnd = this.children[0].cloneNode(true)
          imgStart.className = 'copied-start'
          imgEnd.className = 'copied-end'
          this.$refs.sliderGroup.insertBefore(imgStart, this.children[0])
          this.$refs.sliderGroup.appendChild(imgEnd)
        }, 20)

        width += this.sliderWidth * 2
        this.$refs.sliderGroup.style.width = width + 'px'

        this.$refs.sliderGroup.style.transform = `translate3d(${this.touch.imgWidth[0]}px, 0, 0)`
        this.touch.lastX = this.touch.imgWidth[0]
      },
      _initDots() {
        this.dots = new Array(this.children.length)
      }
    }
  }
</script>

<style scoped>
  .slider {
    position: relative;
    width: 100%;
    height: 100%;
    overflow: hidden;
  }
  .slider-group {
    width: 100%;
    height: 100%;
  }
  .dots {
    position: absolute;
    display: flex;
    top: 85%;
    left: 50%;
    transform: translateX(-50%);
    padding: 0;
    margin: 0;
  }
  .dot {
    display: inline-block;
    padding: 4px;
    background: #999;
    border-radius: 50%;
    margin: 0 5px;
  }
  .current {
    background: #FFF;
  }
</style>