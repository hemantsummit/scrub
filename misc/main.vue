<template>
    <div class="canvas-container">
        <canvas ref="foregroundCanvas" class="foreground-canvas" id="front"></canvas>
        <canvas ref="backgroundCanvas" class="background-canvas" id="back"></canvas>
    </div>
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
        const canvas = document.getElementById('front');
        const ctx = canvas.getContext('2d')
        this.loadPdf('https://arxiv.org/pdf/2010.05322.pdf'); l
        canvas.addEventListener('mousedown', e => {
            const rect = canvas.getBoundingClientRect()
            const x = e.clientX - rect.left
            const y = e.clientY - rect.top

            const box = {
                x: x,
                y: y,
                width: 0,
                height: 0
            }

            const onMouseMove = e => {

                const x = e.clientX - rect.left
                const y = e.clientY - rect.top

                box.width = x - box.x
                box.height = y - box.y
                
                ctx.clearRect(0, 0, canvas.width, canvas.height)
                this.drawBox(ctx, box)
            }

            const onMouseUp = e => {
                canvas.removeEventListener('mousemove', onMouseMove)
                canvas.removeEventListener('mouseup', onMouseUp)
            }

            canvas.addEventListener('mousemove', onMouseMove)
            canvas.addEventListener('mouseup', onMouseUp)
        })

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
                this.frontHeight = canvas.height;
                this.frontWidth = canvas.width;

                const renderContext = {
                    canvasContext: canvas.getContext('2d'),
                    viewport: viewport
                }
                const imageData = await page.render(renderContext).promise
                this.setBackgroundSize();
                const img = new Image()
                img.src = imageData.toDataURL()
                console.log(img.src)
                img.onload = () => {
                    canvas.getContext('2d').drawImage(img, 0, 0, canvas.width, canvas.height)
                }
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
        drawBox(ctx, box) {
            ctx.strokeStyle = 'red'
            ctx.lineWidth = 2
            ctx.strokeRect(box.x, box.y, box.width, box.height)
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
.canvas-container {
  position: relative;
}

.background-canvas, .foreground-canvas {
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
  