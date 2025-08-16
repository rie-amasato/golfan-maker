<template>
    <div class="container blue">
        <canvas id="mainCanvas"></canvas>
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
                <input type="file" @input="reset"></input>
            </div>
        </div>
        <button @click="mkMovie">ゴルファン生成</button>
    </div>
</template>

<script setup>
const imgs=ref([
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
            console.log({...i, img})
            imgs.value[index]= {...imgs.value[index], img}
        }
    })
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
    const maxFrm=100
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
        if(i.startfrm<=frm && frm<=i.endfrm){
            // 0から1になる値
            const p=(frm-i.startfrm)/(i.endfrm-i.startfrm)
            
            context.drawImage(i.img, 
                0, 0, i.img.width, i.img.height,
                elmCanvas.width*(i.sx+(i.ex-i.sx)*p),
                elmCanvas.height*(i.sy+(i.ey-i.sy)*p),
                elmCanvas.width, elmCanvas.height
            )
        }
    })
}
</script>