<template>
  <div class="hello">
    <div id="container"></div>
  </div>
</template>

<script>
import PDFJS from "pdfjs-dist";
import { TextLayerBuilder } from "pdfjs-dist/web/pdf_viewer";
import "pdfjs-dist/web/pdf_viewer.css";
PDFJS.GlobalWorkerOptions.workerSrc = "pdfjs-dist/build/pdf.worker.js";

var container;
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  mounted() {
    this.$nextTick(() => {
        let url =
        "http://mozilla.github.io/pdf.js/web/compressed.tracemonkey-pldi-09.pdf";
        this.getPDF(url);
    });
  },
  methods: {
    async getPDF(url) {
        let pdf = await PDFJS.getDocument(url)
        container = container || document.querySelector('#container')
        for(let i = 0; i < pdf.numPages; i++) {
            try{
                await this.rendPDF(pdf, i)
            } catch(e) {
                // console.error(e)
            }
        }
    },
    async rendPDF(pdf, num) {
        let page = await pdf.getPage(num)
        // 设置展示比例
        let scale = 1.5;
        let viewport = page.getViewport(scale);

        let pageDiv = document.createElement('div');
        pageDiv.setAttribute('id', 'page-' + (page.pageIndex + 1));
        pageDiv.setAttribute('style', 'position: relative');
        container.appendChild(pageDiv);

        let canvas = document.createElement('canvas');
        pageDiv.appendChild(canvas);
        let context = canvas.getContext('2d');
        canvas.height = viewport.height;
        canvas.width = viewport.width;
        
        let renderContext = {
            canvasContext: context,
            viewport: viewport
        };
        
        await page.render(renderContext);
        // debugger
        let textContent = await page.getTextContent()
        // 创建文本图层div
        const textLayerDiv = document.createElement('div');
        textLayerDiv.setAttribute('class', 'textLayer');
        textLayerDiv.setAttribute('style', `width: ${viewport.width}px; margin: 0 auto;`)
        // 将文本图层div添加至每页pdf的div中
        pageDiv.appendChild(textLayerDiv);
        
        // 创建新的TextLayerBuilder实例
        var textLayer = new TextLayerBuilder({
            textLayerDiv: textLayerDiv,
            pageIndex: page.pageIndex,
            viewport: viewport
        });
        
        textLayer.setTextContent(textContent);
        
        textLayer.render();

    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
