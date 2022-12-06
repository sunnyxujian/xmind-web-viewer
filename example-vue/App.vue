<template>
  <header>
    <div id="logo">
      <img width="32" height="32" src="./assets/icon.png" />
    </div>
    <button type="button" @click="openDialog">打开文件</button>
    <input ref="fileRef" @input="onOpenFile" type="file" style="display: none" />
  </header>
  <div id="page-content" ref="contentRef"></div>
  <div class="scale-btn">
    <span class="current-scale" @click="() => (showScale = true)">{{ scalePercent(scale) }}</span>
    <ul class="scale-list" v-show="showScale">
      <li class="scale-num" v-for="num in scaleList" :key="num" @click="onScaleChanged(num)">
        {{ scalePercent(num) }}
      </li>
      <hr class="fit" />
      <li class="scale-num">自适应</li>
    </ul>
  </div>
</template>
<script>
import contentData from './assets/content.json'
import JSZip from 'jszip'
import { loadFromXMind, SnowbrushRenderer } from '../src/index'

export default {
  data() {
    return {
      sheets: contentData,
      scale: 1,
      scaleList: [0.2, 0.5, 0.8, 1, 1.2, 1.5, 2, 3],
      showScale: false,
    }
  },
  computed: {},
  mounted() {
    this.load(this.sheets)
    window.document.onclick = e => {
      const { className } = e.target
      if (className && className === 'current-scale') return
      this.showScale = false
    }
  },
  methods: {
    onOpenFile(e) {
      const inputEle = e.target
      if (inputEle.files.length === 0) {
        return
      }

      const file = inputEle.files[0]
      const fileName = inputEle.value
      const reader = new FileReader()
      reader.onload = e => {
        const jszip = new JSZip()
        return Promise.all([
          Promise.resolve(fileName),
          jszip.loadAsync(e.target.result).then(zip => {
            loadFromXMind(zip).then(result => {
              this.sheets = result.sheets
              this.load(result.sheets)
              console.log(result.sheets)
            })
          }),
        ])
      }

      reader.readAsArrayBuffer(file)
    },
    load(data, scale = 1) {
      const container = this.$refs.contentRef
      if (container.children.length > 0) {
        container.innerHTML = ''
      }

      const renderer = new SnowbrushRenderer(data)
      this.renderer = renderer
      renderer.render()
      const rendererBounds = renderer.bounds

      console.log(rendererBounds)

      const clientWidth = container.clientWidth
      const clientHeight = container.clientHeight
      const width = Math.max(clientWidth, rendererBounds.width * scale)
      const height = Math.max(clientHeight, rendererBounds.height * scale)

      const rendererContainer = document.createElement('div')
      rendererContainer.setAttribute(
        'style',
        `width: ${width * 2}; height: ${height * 2}; position: relative;`,
      )
      rendererContainer.className = 'sheet-container'
      renderer.svg.addTo(rendererContainer)
      rendererContainer.style.backgroundColor = renderer.svg.node.style.backgroundColor
      container.append(rendererContainer)

      renderer.translate(width + rendererBounds.x, height + rendererBounds.y)
      container.scrollTo(width - clientWidth / 2, height - clientHeight / 2)
    },
    openDialog() {
      this.$refs.fileRef.click()
    },
    scalePercent(scale) {
      return scale * 100 + '%'
    },
    onScaleChanged(scale) {
      if (scale === this.scale) return
      this.scale = scale
      this.load(this.sheets, scale)
      this.renderer.svg.scale(scale)
    },
  },
}
</script>

<style scoped>
#page-content {
  position: absolute;
  width: 100%;
  height: calc(100vh - 40px);
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  margin-top: 40px;
  overflow: scroll;
}
/* ::-webkit-scrollbar {
  width: 0;
  height: 0;
  background-color: transparent;
} */
header {
  display: flex;
  height: 40px;
  background-color: #2e2e2e;
  padding: 0 16px;
  align-items: center;
}

#logo {
  display: flex;
  height: 100%;
  align-items: center;
}

button {
  border: none;
  background: none;
  color: #ffffff;
  font-size: 12px;
  font-weight: 600;
  margin: 4px 6px;
  border-radius: 4px;
  height: 32px;
  cursor: pointer;
}

button:hover {
  background-color: rgba(223, 223, 223, 0.2);
}

button:focus {
  outline: 0;
}
.scale-btn {
  width: 72px;
  height: 44px;
  padding: 6px;
  text-align: center;
  background-color: #ffffff;
  box-shadow: 0 0px 10px 0 rgb(0 0 0 / 5%), inset 0 0 1px 0 rgb(0 0 0 / 20%),
    0 12px 40px 0 rgb(0 0 0 / 10%);
  box-sizing: border-box;
  position: absolute;
  left: 30px;
  bottom: 30px;
  cursor: pointer;
  border-radius: 6px;
}
.scale-btn .current-scale {
  display: inline-block;
  width: 100%;
  height: 100%;
  line-height: 32px;
  user-select: none;
}
.scale-btn .current-scale:hover {
  background-color: rgba(223, 223, 223, 0.4);
  border-radius: 6px;
}

.scale-list {
  list-style: none;
  margin: 0;
  position: absolute;
  top: -300px;
  left: 0;
  padding: 9px;
  box-shadow: 0 0px 10px 0 rgb(0 0 0 / 5%), inset 0 0 1px 0 rgb(0 0 0 / 20%),
    0 12px 40px 0 rgb(0 0 0 / 10%);
  font-size: 14px;
  width: 120px;
  box-sizing: border-box;
  border-radius: 6px;
  background: #ffffff;
}
.scale-num {
  padding: 5px 10px;
}
.scale-num:hover {
  background-color: rgba(223, 223, 223, 0.4);
  border-radius: 6px;
}
.fit {
  border: none;
  background: #f0f0f0;
  height: 1px;
  margin: 5px 0;
}
</style>