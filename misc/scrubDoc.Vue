<template>
  <v-app>
      <v-card class="mx-2 my-2" elevation="6" outlined height="calc(-50px + 100vh)" width="calc(2*100vw/3 - 50px)">
          <v-app-bar color="#F4F3F4" dense>
              <v-spacer></v-spacer>
              <v-icon @click="zoomOut">
                  mdi-magnify-minus-outline
              </v-icon>
              <v-icon @click="zoomIn">
                  mdi-magnify-plus-outline
              </v-icon>
              <v-icon @click="fitWidth">
                  mdi-arrow-collapse-all
              </v-icon>
              <v-icon @click="fitAuto">
                  mdi-arrow-expand-all
              </v-icon>
              <v-spacer></v-spacer>
          </v-app-bar>
          <div class="pdf-container">
              <div class="pdf-viewers">
                  <div class="pdf-viewer">
                      <canvas ref="foregroundCanvas" class="foreground-canvas" id="front"></canvas>
                      <canvas ref="backgroundCanvas" class="background-canvas" id="back"></canvas>
                  </div>
              </div>
          </div>
      </v-card>
  </v-app>
</template>

<script>
import * as pdfjsLib from 'pdfjs-dist';

pdfjsLib.GlobalWorkerOptions.workerSrc =
  "https://cdn.jsdelivr.net/npm/pdfjs-dist@3.4.120/build/pdf.worker.min.js";

export default {
  data() {
      return {
          pdfDoc: null,
          numPages: 0,
          currentPage: 1,
          pageRendering: false,
          pageNumPending: null,
          thumbnailScale: 0.1,
          increment: 1,
          imageScale: 1,
          isDrawing: false,
          startX: null,
          startY: null,
          endX: null,
          endY: null,
          frontHeight: null,
          frontWidth: null,
      }
  },
  mounted() {
      this.loadPdf('https://arxiv.org/pdf/2010.05322.pdf');
  },
  methods: {
      async loadPdf(url) {
          try {
              const pdf = await pdfjsLib.getDocument(url).promise
              this.pdfDoc = pdf
              this.numPages = pdf.numPages
              this.renderPage(this.currentPage)
          } catch (error) {
              console.error(error)
          }
      },
      async renderPage(pageNumber) {
          if (this.pageRendering) {
              this.pageNumPending = pageNumber
              return
          }
          this.pageRendering = true
          try {
              const page = await this.pdfDoc.getPage(pageNumber)
              const canvas = this.$refs.backgroundCanvas
              const viewport = page.getViewport({ scale: this.imageScale })
              canvas.height = viewport.height;
              canvas.width = viewport.width;
              const renderContext = {
                  canvasContext: canvas.getContext('2d'),
                  viewport: viewport
              }
              this.frontHeight = canvas.height;
              this.frontWidth = canvas.width;
              this.setBackgroundSize();
              await page.render(renderContext).promise
              this.pageRendering = false
              if (this.pageNumPending !== null) {
                  const pageNum = this.pageNumPending
                  this.pageNumPending = null
                  this.renderPage(pageNum)
              }
          } catch (error) {
              console.error(error)
          }
      },
      goToPage(pageNumber) {
          if (pageNumber > 0 && pageNumber <= this.numPages) {
              this.currentPage = pageNumber
              this.renderPage(this.currentPage)
          }
      },
      zoomIn() {
          this.imageScale += 0.25
          this.renderPage(this.currentPage)
      },
      zoomOut() {
          if (this.imageScale <= this.increment) return;
          this.imageScale -= 0.25
          this.renderPage(this.currentPage)
      },
      fitWidth() {
          this.imageScale = 1
          this.renderPage(this.currentPage)
      },
      fitAuto() {
          this.imageScale = 2
          this.renderPage(this.currentPage)
      },
      setBackgroundSize() {
          const backgroundCanvas = this.$refs.backgroundCanvas;
          const foregroundCanvas = this.$refs.foregroundCanvas;
          foregroundCanvas.width = this.frontWidth;
          foregroundCanvas.height = this.frontHeight;
      }
  }
}
</script>

<style>
.pdf-container {
  margin-top: 10px;
  margin-right: 10px;
  margin-left: 10px;
  margin-bottom: 10px;
  display: flex;
  flex-direction: row;
  height: calc(100vh - 120px)
}

.pdf-viewer {
  width: max-content;
  margin: 0 auto;
}

.pdf-viewers {
  width: 100%;
  overflow: scroll;
  position: relative;
}

.background-canvas,
.foreground-canvas {
  position: absolute;
  top: 0;
  left: 0;
}

.background-canvas {
  z-index: 1;
}

.foreground-canvas {
  z-index: 2;
}
</style>
