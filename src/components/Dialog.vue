<template>
    <div class="dialog-overlay" :class="{'no-overlay': !overlay}" v-if="value">
        <div ref="dialog" class="dialog" :style="styleCompute" v-if="value">
            <div ref="dialogHeader" class="dialog-header">
                <div class="dialog-title">{{title}}</div>
                <button class="dialog-close" @click="onClose">X</button>
            </div>
            <div class="dialog-content"><slot/></div>
        </div>
    </div>
</template>

<script>
export default {
  name: 'Dialog',
  model: {
    prop: 'value',
    event: 'click'
  },
  props: {
    title: {
      type: String,
      default: ''
    },
    value: {
      type: Boolean,
      default: false
    },
    center: {
      type: Boolean,
      default: true
    },
    width: {
      type: Number,
      default: null
    },
    height: {
      type: Number,
      default: null
    },
    draggable: {
      type: Boolean,
      default: true
    },
    resizable: {
      type: Boolean,
      default: true
    },
    restoreToCenter: {
      type: Boolean,
      default: false
    },
    overlay: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    styleCompute () {
      return {
        top: this.top + 'px',
        left: this.left + 'px',
        width: this.widthLocal + 'px',
        height: this.heightLocal + 'px',
        cursor: this.cursor,
        'z-index': this.zIndex
      }
    }
  },
  data: () => ({
    maxX: null,
    maxY: null,
    startX: null,
    startY: null,
    startW: null,
    startH: null,
    resizePixel: 5,
    leftPos: 0,
    topPos: 0,

    isDrag: false,
    isResize: false,

    resizeMode: '',
    zIndex: 1,
    zIndexFlag: false,
    minW: 300,
    minH: 100,

    top: 0,
    left: 0,
    widthLocal: null,
    heightLocal: null,
    cursor: 'default',
    first: true
  }),
  methods: {
    centerize () {
      const { dialog } = this.$refs
      this.top = ((window.innerHeight - dialog.clientHeight) / 2)
      this.left = ((window.innerWidth - dialog.clientWidth) / 2)
    },
    mouseDown (evt) {
      evt = evt || window.event
      this.zIndexFlag = true

      const { dialog, dialogHeader } = this.$refs
      const rect = this.getOffset()
      const { documentElement, body } = document

      this.maxX = Math.max(documentElement.clientWidth, body.scrollWidth, documentElement.scrollWidth, body.offsetWidth, documentElement.offsetWidth)

      this.maxY = Math.max(
        documentElement.clientHeight,
        body.scrollHeight,
        documentElement.scrollHeight,
        body.offsetHeight,
        documentElement.offsetHeight
      )

      if (rect.right > this.maxX) {
        this.maxY = rect.right
      }
      if (rect.bottom > this.maxY) {
        this.maxY = rect.bottom
      }

      this.startX = evt.pageX
      this.startY = evt.pageY
      this.startW = dialog.clientWidth
      this.startH = dialog.clientHeight
      this.leftPos = rect.left
      this.topPos = rect.top

      if (evt.target === dialogHeader && this.resizeMode === '') {
        this.cursor = this.draggable ? 'move' : 'default'
        this.isDrag = this.draggable
      } else if (this.resizeMode !== '') {
        this.isResize = this.resizable
      }

      evt.stopPropagation()
      evt.preventDefault()
    },
    mouseMove (evt) {
      evt = evt || window.event
      const { dialog, dialogHeader } = this.$refs
      if (!(evt.target === dialog || evt.target === dialogHeader) && !this.isDrag && this.resizeMode === '') return

      if (this.isDrag) {
        const dx = this.startX - evt.pageX
        const dy = this.startY - evt.pageY

        let scrollL = Math.max(document.body.scrollLeft, document.documentElement.scrollLeft)
        let scrollT = Math.max(document.body.scrollTop, document.documentElement.scrollTop)
        let left = this.leftPos - dx
        let top = this.topPos - dy
        if (dx < 0) {
          if (left + this.startW > this.maxX) {
            left = this.maxX - this.startW
          }
        }
        if (dx > 0) {
          if (left < 0) {
            left = 0
          }
        }
        if (dy < 0) {
          if (top + this.startH > this.maxY) {
            top = this.maxY - this.startH
          }
        }
        if (dy > 0) {
          if (top < 0) {
            top = 0
          }
        }

        this.left = left
        this.top = top

        if (evt.clientY > window.innerHeight - 32) {
          scrollT += 32
        } else if (evt.clientY < 32) {
          scrollT -= 32
        }
        if (evt.clientX > window.innerWidth - 32) {
          scrollL += 32
        } else if (evt.clientX < 32) {
          scrollL -= 32
        }
        if (top + this.startH === this.maxY) {
          scrollT = this.maxY - window.innerHeight + 20
        } else if (top === 0) {
          scrollT = 0
        }
        if (left + this.startW === this.maxX) {
          scrollL = this.maxX - window.innerWidth + 20
        } else if (left === 0) {
          scrollL = 0
        }
        if (this.startH > window.innerHeight) {
          if (evt.clientY < window.innerHeight / 2) {
            scrollT = 0
          } else {
            scrollT = this.maxY - window.innerHeight + 20
          }
        }
        if (this.startW > window.innerWidth) {
          if (evt.clientX < window.innerWidth / 2) {
            scrollL = 0
          } else {
            scrollL = this.maxX - window.innerWidth + 20
          }
        }
        window.scrollTo(scrollL, scrollT)
      } else if (this.isResize) {
        let dw, dh, w, h
        if (this.resizeMode === 'w') {
          dw = this.startX - evt.pageX
          if (this.leftPos - dw < 0) {
            dw = this.leftPos
          }
          w = this.startW + dw
          if (w < this.minW) {
            w = this.minW
            dw = w - this.startW
          }
          this.widthLocal = w
          this.left = this.leftPos - dw + 2
        } else if (this.resizeMode === 'e') {
          dw = evt.pageX - this.startX
          if (this.leftPos + this.startW + dw > this.maxX) {
            dw = this.maxX - this.leftPos - this.startW
          }
          w = this.startW + dw
          if (w < this.minW) {
            w = this.minW
          }
          this.widthLocal = w
        } else if (this.resizeMode === 'n') {
          dh = this.startY - evt.pageY
          if (this.topPos - dh < 0) {
            dh = this.topPos
          }
          h = this.startH + dh
          if (h < this.minH) {
            h = this.minH
            dh = h - this.startH
          }
          this.heightLocal = h
          this.top = this.topPos - dh
        } else if (this.resizeMode === 's') {
          dh = evt.pageY - this.startY
          if (this.topPos + this.startH + dh > this.maxY) {
            dh = this.maxY - this.topPos - this.startH
          }
          h = this.startH + dh
          if (h < this.minH) {
            h = this.minH
          }
          this.heightLocal = h
        } else if (this.resizeMode === 'nw') {
          dw = this.startX - evt.pageX
          dh = this.startY - evt.pageY
          if (this.leftPos - dw < 0) {
            dw = this.leftPos
          }
          if (this.topPos - dh < 0) {
            dh = this.topPos
          }
          w = this.startW + dw
          h = this.startH + dh
          if (w < this.minW) {
            w = this.minW
            dw = w - this.startW
          }
          if (h < this.minH) {
            h = this.minH
            dh = h - this.startH
          }
          this.widthLocal = w
          this.heightLocal = h
          this.left = this.leftPos - dw
          this.top = this.topPos - dh
        } else if (this.resizeMode === 'sw') {
          dw = this.startX - evt.pageX
          dh = evt.pageY - this.startY
          if (this.leftPos - dw < 0) {
            dw = this.leftPos
          }
          if (this.topPos + this.startH + dh > this.maxY) {
            dh = this.maxY - this.topPos - this.startH
          }
          w = this.startW + dw
          h = this.startH + dh
          if (w < this.minW) {
            w = this.minW
            dw = w - this.startW
          }
          if (h < this.minH) {
            h = this.minH
          }
          this.widthLocal = w
          this.heightLocal = h
          this.left = this.leftPos - dw
        } else if (this.resizeMode === 'ne') {
          dw = evt.pageX - this.startX
          dh = this.startY - evt.pageY
          if (this.leftPos + this.startW + dw > this.maxX) {
            dw = this.maxX - this.leftPos - this.startW
          }
          if (this.topPos - dh < 0) {
            dh = this.topPos
          }
          w = this.startW + dw
          h = this.startH + dh
          if (w < this.minW) {
            w = this.minW
          }
          if (h < this.minH) {
            h = this.minH
            dh = h - this.startH
          }
          this.widthLocal = w
          this.heightLocal = h
          this.top = this.topPos - dh
        } else if (this.resizeMode === 'se') {
          dw = evt.pageX - this.startX
          dh = evt.pageY - this.startY
          if (this.leftPos + this.startW + dw > this.maxX) {
            dw = this.maxX - this.leftPos - this.startW
          }
          if (this.topPos + this.startH + dh > this.maxY) {
            dh = this.maxY - this.topPos - this.startH
          }
          w = this.startW + dw
          h = this.startH + dh
          if (w < this.minW) {
            w = this.minW
          }
          if (h < this.minH) {
            h = this.minH
          }
          this.widthLocal = w
          this.heightLocal = h
        }
      } else {
        let mode = ''
        let cursor = ''

        if (evt.target === dialog) {
          const rect = this.getOffset()
          if (evt.pageY < rect.top + this.resizePixel) mode = 'n'
          else if (evt.pageY > rect.bottom - this.resizePixel) mode = 's'
          if (evt.pageX < rect.left + this.resizePixel) mode += 'w'
          else if (evt.pageX > rect.right - this.resizePixel) mode += 'e'
        }

        if (mode !== '' && this.resizeMode !== mode) {
          if (['n', 's'].includes(mode)) cursor = 'ns-resize'
          else if (['e', 'w'].includes(mode)) cursor = 'ew-resize'
          else if (['ne', 'sw'].includes(mode)) cursor = 'nesw-resize'
          else if (['nw', 'se'].includes(mode)) cursor = 'nwse-resize'
          this.resizeMode = mode
          this.cursor = this.resizable ? cursor : 'default'
        } else if (mode === '' && this.resizeMode !== '') {
          this.cursor = 'default'
          this.resizeMode = ''
        }
      }
    },
    mouseUp (evt) {
      evt = evt || window.event
      const { dialog, dialogHeader } = this.$refs

      if (this.zIndexFlag) {
        this.zIndex += 1
        this.zIndexFlag = false
      } else {
        this.zIndex = 1
      }

      if (!([dialog, dialogHeader].includes(evt.target)) && !this.isDrag && this.resizeMode === '') return

      if (this.isDrag) {
        this.cursor = 'default'
        this.isDrag = false
      } else if (this.isResize) {
        this.cursor = 'default'
        this.isResize = false
        this.resizeMode = ''
      }

      evt.preventDefault()
      evt.stopPropagation()
    },
    getOffset () {
      const rect = this.$refs.dialog.getBoundingClientRect()
      const offsetX = window.scrollX || document.documentElement.scrollLeft
      const offsetY = window.scrollY || document.documentElement.scrollTop
      return {
        left: rect.left + offsetX,
        top: rect.top + offsetY,
        right: rect.right + offsetX,
        bottom: rect.bottom + offsetY
      }
    },
    registerEventListeners () {
      this.$refs.dialog.addEventListener('mousedown', this.mouseDown)
      document.addEventListener('mousemove', this.mouseMove)
      document.addEventListener('mouseup', this.mouseUp)
    },
    unRegisterEventListeners () {
      this.$refs.dialog.removeEventListener('mousedown', this.mouseDown)
      document.removeEventListener('mousemove', this.mouseMove)
      document.removeEventListener('mouseup', this.mouseUp)
    },
    onClose () {
      this.$emit('close')
    }
  },
  created () {
    this.widthLocal = this.width
    this.heightLocal = this.height

    this.$watch('value', (show) => {
      if (show) {
        this.$nextTick(() => this.restoreToCenter && this.centerize())
      }
    })
  },
  beforeUpdate () {
    if (this.$refs.dialog) {
      this.unRegisterEventListeners()
    }
  },
  updated () {
    if (this.value) {
      if (this.first) {
        this.first = false
        this.center && this.centerize()
      }
      this.registerEventListeners()
    }
  },
  beforeDestroy () {
    this.unRegisterEventListeners()
  }
}
</script>

<style lang="scss" scoped>
.dialog {
  background: #FFF;
  width: 400px;
  min-height: 100px;
  position: fixed;
  border: 1px solid #ddd;
  box-sizing: border-box;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  font-size: 14px;
  border-radius: 4px;
  padding: 4px;
  box-shadow: 0 2px 30px rgba($color: #CCC, $alpha: .5);
  display: flex;
  flex-direction: column;
  z-index: 1;

    &-overlay {
        position: fixed;
        top:0;
        left:0;
        bottom:0;
        right:0;
        background: rgba($color: #000, $alpha: .8);

        &.no-overlay {
            background: transparent;
            position: static;
        }
    }

  &-header {
    padding: 10px;
    background: #f8f8f8;
    color: #333;
    display: block;
    border-radius: 4px;
    display: flex;
    transition: all 250ms cubic-bezier(0.250, 0.460, 0.450, 0.940);

    &:hover {
        background: #eee;
    }
  }

  &-content {
      line-height: 1.6em;
      overflow: auto;
      flex-basis: 100%;
  }

  &-close {
      margin-left: auto;
  }
}
</style>
