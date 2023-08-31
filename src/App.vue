<script setup>
import {ref} from "vue";
import Editor from "@/components/editor.vue";
import html2canvas from "html2canvas";
import * as canvg from "canvg"

console.log(canvg)

const editorValueHtml = ref("");
const editorValueJson = ref({});

const resSvg = ref("")

const toSvgEl = ref(null)
const downSvg = () => {
    const dom = toSvgEl.value
    const cloneDom = dom.cloneNode(true)

    let htmlSvg = 'data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" width="' + dom.offsetWidth + '" height="' + dom.offsetHeight + '"><foreignObject x="0" y="0" width="100%" height="100%">' +
        new XMLSerializer().serializeToString(cloneDom) +
        document.querySelector('style').outerHTML +
        '</foreignObject></svg>';

    resSvg.value = htmlSvg

    htmlSvg = htmlSvg.replace(/\n/g, '').replace(/\t/g, '').replace(/#/g, '%23');

    const img = new Image();

    img.src = htmlSvg;
    img.onload = function () {
        const canvas = document.createElement('canvas');
        canvas.width = dom.offsetWidth;
        canvas.height = dom.offsetHeight;
        const ctx = canvas.getContext('2d');
        ctx.drawImage(img, 0, 0);
        const dataUrl = canvas.toDataURL('image/png');
        const file = base64ToFile(dataUrl);
        const a = document.createElement('a');
        a.download = 'test.png';
        a.href = URL.createObjectURL(file);
        a.click();
    }

}

function base64ToFile(dataurl) {
    let arr = dataurl.split(',');
    let mime = arr[0].match(/:(.*?);/)[1];
    let bstr = atob(arr[1]);
    let n = bstr.length;
    let u8arr = new Uint8Array(n);
    while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
    }
    return new Blob([u8arr], {
        type: mime
    });
}

</script>

<template>
    <div class="box">
        <div class="box1">
            <h1>在这里输入</h1>
            <hr>
            <editor v-model="editorValueHtml" v-model:model-json="editorValueJson"></editor>
        </div>
        <div class="box2">
            <h1>html位置</h1>
            <hr>
            <div class="top">
                <h1>原文</h1>
                <hr>
                {{ editorValueHtml }}
                <hr>
            </div>
            <div class="below">
                <h1>实时展示</h1>
                <hr>
                <div ref="toSvgEl" class="ProseMirror ty-editor" v-html="editorValueHtml"></div>
            </div>
        </div>
        <div class="box3">
            <h1>json 格式的</h1>
            <hr>
            {{ editorValueJson }}
        </div>
        <div class="box4">
            <h1>to Svg</h1>
            <button type="button" @click="downSvg">下载jpg</button>
            <div v-text="resSvg"></div>
        </div>
    </div>
</template>

<style lang="scss" scoped>
hr {
    margin: 20px;
}

.box {
    height: 100vh;
    width: 100vw;
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    grid-template-rows: repeat(2, 1fr);

    @for $i from 1 through 4 {
        .box#{$i} {
            height: 100%;
            width: 100%;
            border: 1px solid red;
            overflow: auto;
        }
    }

}
</style>
