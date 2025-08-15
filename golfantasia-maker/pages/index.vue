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
const bgColor=ref("00FF00");

onMounted(()=>{
    reset()
})

const reset=()=>{
    const elmCanvas=document.getElementById("mainCanvas")
    const context = elmCanvas.getContext('2d');

    //background color
    context.beginPath();
    context.fillStyle = "#"+bgColor.value;
    context.fillRect(0, 0, elmCanvas.width, elmCanvas.height);
}

const mkMovie=()=>{
    const maxFrm=20
    let frm=0

    const interval=setInterval(()=>{
        mkFrame(frm)
        if (frm<maxFrm)frm+=1
        else clearInterval(interval)
    }, 50)
}

const mkFrame=(frm)=>{
    const elmCanvas=document.getElementById("mainCanvas")
    const context = elmCanvas.getContext('2d');

    //background color
    context.beginPath();
    
    context.fillStyle = "rgb(0, "+(255-frm*10)+", 0)";
    context.fillRect(0, 0, elmCanvas.width, elmCanvas.height);
}
</script>