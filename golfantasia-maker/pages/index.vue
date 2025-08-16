<template>
    <div class="container blue">
        <canvas id="mainCanvas"
            width="480"
            height="270"
        ></canvas>


        <div>1080P=>1920pxx1080px</div>
    </div>
    <div class="container blue">
        <p>設定項目</p>
        <div class="grid">
            <div class="s1">背景色</div>
            <div class="s2 e5">
                #<input v-model="bgColor" @input="reset"></input>
            </div>

            <div class="s1">画像</div>
            <div class="s2 e5">
                <input type="file" @change="charaImgUpload"></input>
            </div>
        </div>
        <button @click="mkMovie">ゴルファン生成</button>
    </div>
    <div class="container blue">
    <p>素材テンプレート(クリッピングしての使用を推奨)</p>
    <img src="/template.png">
    </div>
</template>

<script setup>

const imgs=ref([
    {
		path: "chara.png", img: null,
		startfrm: 15, endfrm: 18,
		sx:-1, ex: 0, sy: -0.1, ey: 0,
        effect: "uploadTarget"
	},
    {
		path: "chara.png", img: null,
		startfrm: 18, endfrm: 50,
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
		startfrm: 15, endfrm:18,
		sx:-1, ex: 0, sy: -0.1, ey: 0,
	},
	{
		path: "mimi.png", img: null,
		startfrm: 18, endfrm: 50,
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
		startfrm: 22, endfrm: 35,
		sx: 0, ex: 0, sy: 0.1, ey: 0.1
	},
    {
		path: "amaiamai.png", img: null,
		startfrm: 32, endfrm: 34,
		sx: 0.5, ex: -0.05, sy: 0.2, ey: 0.05
	},
    {
		path: "amaiamai.png", img: null,
		startfrm: 34, endfrm: 50,
		sx: -0.05, ex: -0.05, sy: 0.05, ey: 0.05
	},
    {
		path: "amaiamai.png", img: null,
		startfrm: 50, endfrm: 55,
		sx: -0.05, ex: 1, sy: 0.05, ey: 1
	},
])
const bgColor=ref("00FF00");

onMounted(()=>{
    loadimg()
    reset()
})

const loadimg=()=>{
    imgs.value.forEach((i, index)=>{
        const img=new Image()
        img.src=i.path
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
    const elmCanvas=document.getElementById("mainCanvas")
    const context = elmCanvas.getContext('2d');

    //background color
    context.beginPath();
    context.fillStyle = "#"+bgColor.value;
    context.fillRect(0, 0, elmCanvas.width, elmCanvas.height);
}

const mkMovie=async()=>{
    const maxFrm=55
    let frm=0

    const interval=setInterval(()=>{
        mkFrame(frm)
        if (frm<maxFrm)frm+=1
        else clearInterval(interval)
    }, 50)
}

const mkFrame=(frm)=>{
    reset()
    const elmCanvas=document.getElementById("mainCanvas")
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
</script>