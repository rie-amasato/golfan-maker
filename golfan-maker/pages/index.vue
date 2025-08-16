<template>
    <div class="container blue">
        <canvas id="mainCanvas"
            width="480"
            height="270"
        ></canvas>
    </div>
    <div class="container blue">
        <p>設定項目</p>
        <div class="grid">
            <div class="s1">向き</div>
            <div class="s2 e3">
                <input type="radio" name="direction" value="horizontal" 
                    id="directionHorizontal"
                    v-model="direction" @change="changeDirection"></input>
                <label for="directionHorizontal">横</label>
            </div>
            <div class="s3 e4">
                <input type="radio" name="direction" value="vertical"
                    id="directionVertical"
                    v-model="direction" @change="changeDirection"></input>
                <label for="directionVertical">縦</label>
            </div>
            <div class="s1">背景色</div>
            <div class="s2 e5">
                #<input v-model="bgColor" @input="reset"></input>
            </div>

            <div class="s1">画像</div>
            <div class="s2 e5">
                <input type="file" @change="charaImgUpload"></input>
            </div>
        </div>

        <div class="mt8">
            <button @click="previewMovie" :disabled="isProcess">ゴルファンプレビュー</button>
            <button @click="saveMovie" :disabled="isProcess" class="ml16">ゴルファン動画保存(1080p)</button>
            <span class="ml16">{{ processStatus }}</span>
        </div>
    </div>
    <div class="container blue">
    <p>素材テンプレート(クリッピングしての使用を推奨)</p>
    <img src="/template.png">
    </div>

    <a id="dlLink"></a>
    <canvas
        id="canvasP1080"
        width="1920"
        height="1080"
        class="none"
    ></canvas>
</template>

<style>
.mt8{
    margin-top: 8px;
}
.ml16{
    margin-left: 16px;
}

.none{
    display: none;
}
</style>

<script setup>
const imgs=ref([
    {
		path: "chara.png", img: null,
		startfrm: 10, endfrm: 13,
		sx:-1, ex: 0, sy: -0.1, ey: 0,
        effect: "uploadTarget"
	},
    {
		path: "chara.png", img: null,
		startfrm: 13, endfrm: 50,
		sx:0, ex: 0, sy: 0, ey: 0,
        effect: "uploadTarget"
	},
    {
		path: "chara.png", img: null,
		startfrm: 50, endfrm: 55,
		sx: 0, ex: -2, sy: 0, ey: 0,
        effect: "upscale uploadTarget"
	},
    {
        path: 'line.png', img: null,
        startfrm: 0, endfrm: 2,
        sx: -1, sy: 0, ex: 0, ey: 0
    },
    {
        path: 'line.png', img: null,
        startfrm: 2, endfrm: 6,
        sx: 0, ex: 0, sy: 0, ey: 0
    },
    {
        path: 'line.png', img: null,
        startfrm: 6, endfrm: 10,
        sx: 0, ex: 0, sy: 0, ey: -0.3
    },
    {
        path: 'line.png', img: null,
        startfrm: 6, endfrm: 10,
        sx: 0, ex: 0, sy: 0, ey: 0.5
    },
    {
        path: 'line.png', img: null,
        startfrm: 10, endfrm: 50,
        sx: 0, ex: 0, sy: -0.3, ey: -0.3
    },
    {
        path: 'line.png', img: null,
        startfrm: 10, endfrm: 50,
        sx: 0, ex: 0, sy: 0.5, ey: 0.5
    },
    {
        path: 'line.png', img: null,
        startfrm: 50, endfrm: 55,
        sx: 0, ex: 0, sy: -0.3, ey: -0.6
    },
    {
        path: 'line.png', img: null,
        startfrm: 50, endfrm: 55,
        sx: 0, ex: 0, sy: 0.5, ey: 1
    },
    {
		path: "mimi.png", img: null,
		startfrm: 10, endfrm:13,
		sx:-1, ex: 0, sy: -0.1, ey: 0,
	},
	{
		path: "mimi.png", img: null,
		startfrm: 13, endfrm: 50,
		sx: 0, ex: 0, sy: 0, ey: 0,
	},
	{
		path: "mimi.png", img: null,
		startfrm: 50, endfrm:55,
		sx:0, ex:-2, sy: 0, ey: 0,
        effect: "upscale",
	},
    {
        path: 'fa.png', img: null,
        startfrm: 18, endfrm: 22,
        sx: 0.2, ex: 0, sy: 1, ey: 0.1
    },
    {
		path: "fa.png", img: null,
		startfrm: 22, endfrm: 45,
		sx: 0, ex: 0, sy: 0.1, ey: 0.1
	},
    {
		path: "amaiamai.png", img: null,
		startfrm: 32, endfrm: 34,
		sx: 0.5, ex: -0.06, sy: 0.2, ey: 0.05
	},
    {
		path: "amaiamai.png", img: null,
		startfrm: 34, endfrm: 50,
		sx: -0.06, ex: -0.06, sy: 0.05, ey: 0.05
	},
    {
		path: "amaiamai.png", img: null,
		startfrm: 50, endfrm: 55,
		sx: -0.06, ex: 1, sy: 0.05, ey: 1
	},
])

const isProcess=ref(false)
const direction=ref("horizontal")
const bgColor=ref("00FF00");

const processCanvas=ref("mainCanvas")
const processStatus=ref("")

onMounted(()=>{
    loadimg()
    reset()
})

const loadimg=()=>{
    imgs.value.forEach((i, index)=>{
        const img=new Image()

        const isVerticalPath=(direction.value=="vertical" &&
            ["amaiamai.png", "chara.png", "fa.png"].includes(i.path))
        
        img.src= isVerticalPath?"vertical/"+i.path:i.path
        img.onload=async()=>{
            imgs.value[index]= {...imgs.value[index], img}
        }
    })
}

const mimiDisable=()=>{
    imgs.value.forEach((i, index)=>{
        if (i.path=="mimi.png"){
            imgs.value[index]={...imgs.value[index], disabled: true}
        }
    })
}

const changeDirection=()=>{
    mimiDisable()
    loadimg()
}

const charaImgUpload=(e)=>{
    mimiDisable()
    const charaImg=e.target.files[0]

    const blobUrl=window.URL.createObjectURL(charaImg)
    imgs.value.forEach((i, index)=>{
        if (i.effect?.includes("uploadTarget")){
            imgs.value[index]={...imgs.value[index], path: blobUrl}
        }
    })
    loadimg()
}

const reset=()=>{
    const elmCanvas=document.getElementById(processCanvas.value)
    const context = elmCanvas.getContext('2d');

    context.beginPath();
    context.fillStyle = "#"+bgColor.value;
    context.fillRect(0, 0, elmCanvas.width, elmCanvas.height);
}

const mkMovie=(recorder=false)=>{
    const maxFrm=55
    let frm=0

    const interval=setInterval(()=>{
        mkFrame(frm)
        if (frm<maxFrm)frm+=1
        else {
            clearInterval(interval)
            isProcess.value=false
            if(Object.prototype.toString.call(recorder).
                includes("MediaRecorder"))recorder.stop()
        }
    }, 50)
}

const mkFrame=(frm)=>{
    reset()
    const elmCanvas=document.getElementById(processCanvas.value)
    const context = elmCanvas.getContext('2d');

    imgs.value.forEach((i)=>{
        if(!i.disabled && i.startfrm<=frm && frm<=i.endfrm){
            // 0から1になる値
            const p=(frm-i.startfrm)/(i.endfrm-i.startfrm)
            
            if (i.effect?.includes("upscale")){
                const scale=0.5
                context.drawImage(i.img, 
                    i.img.width*(scale*p), i.img.height*(scale*p), 
                    i.img.width*(1-scale*p), i.img.height*(1-scale*p),
                    
                    elmCanvas.width*(i.sx+(i.ex-i.sx)*p)*scale*p,
                    elmCanvas.height*(i.sy+(i.ey-i.sy)*p),
                    elmCanvas.width, elmCanvas.height*(1+scale*p)
                )
            }else{
                context.drawImage(i.img, 
                    0, 0, i.img.width, i.img.height,
                    elmCanvas.width*(i.sx+(i.ex-i.sx)*p),
                    elmCanvas.height*(i.sy+(i.ey-i.sy)*p),
                    elmCanvas.width, elmCanvas.height
                )
            }
        }
    })
}

const previewMovie=()=>{
    isProcess.value=true
    processCanvas.value="mainCanvas"
    mkMovie()
}

const saveMovie=async ()=>{
    processStatus.value="mp4生成中..."
    isProcess.value=true
    processCanvas.value="canvasP1080"

    const canvas=document.getElementById(processCanvas.value)
    const stream=canvas.captureStream()
    const recorder = new MediaRecorder(stream, {mimeType: "video/mp4;"})

    recorder.ondataavailable=(e)=>{
        const videoBlob=new Blob([e.data], {type: e.data.type})
        const blobUrl=window.URL.createObjectURL(videoBlob)

        const anchor=document.getElementById("dlLink")
        anchor.download="golfan.mp4"
        anchor.href=blobUrl
        anchor.click()
        processStatus.value=""
    }
    recorder.start()
    await mkMovie(recorder)
}
</script>