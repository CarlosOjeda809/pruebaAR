<script setup>
import { ref, onMounted } from 'vue'

const showAR = ref(false)

onMounted(() => {
    if (process.client) {
        
        const aframe = document.createElement('script')
        aframe.src = 'https://aframe.io/releases/1.2.0/aframe.min.js'
        document.head.appendChild(aframe)

        const arjs = document.createElement('script')
        arjs.src = 'https://cdn.jsdelivr.net/gh/AR-js-org/AR.js/aframe/build/aframe-ar.min.js'
        document.head.appendChild(arjs)
    }
})
</script>

<template>
    <div>
        <div v-if="showAR">
            <a-scene embedded arjs="sourceType: webcam;" vr-mode-ui="enabled: false"
                renderer="logarithmicDepthBuffer: true;">
                <a-camera gps-camera rotation-reader></a-camera>

                <a-entity gps-entity-place="latitude: 43.385; longitude: -4.290" look-at="[gps-camera]"
                    geometry="primitive: box" material="color: red;">
                </a-entity>
            </a-scene>
        </div>
        <button @click="showAR = true" v-if="!showAR" class="bg-red-900 shadow-md text-wgite p-2 text-2xl">Iniciar AR</button>
    </div>
</template>



<style scoped>
a-scene {
    height: 100vh;
}
</style>
