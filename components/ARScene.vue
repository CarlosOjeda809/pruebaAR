<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const showAR = ref(false)
const loading = ref(false)
const error = ref(null)

const scriptsLoaded = ref({ aframe: false, arjs: false })

const startAR = async () => {
  loading.value = true
  error.value = null

  try {
    
    await loadScript('https://aframe.io/releases/1.4.2/aframe.min.js')
    scriptsLoaded.value.aframe = true
    
    
    await new Promise(resolve => setTimeout(resolve, 300))
    
    // Luego cargar AR.js
    await loadScript('https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar.js')
    scriptsLoaded.value.arjs = true

    
    setTimeout(() => {
      showAR.value = true
      loading.value = false
    }, 1000)
  } catch (err) {
    error.value = `Error al cargar scripts: ${err.message}`
    loading.value = false
  }
}

const loadScript = (src) => {
  return new Promise((resolve, reject) => {
    if (document.querySelector(`script[src="${src}"]`)) {
      resolve()
      return
    }

    const script = document.createElement('script')
    script.src = src
    script.onload = resolve
    script.onerror = () => reject(new Error(`Failed to load script: ${src}`))
    document.head.appendChild(script)
  })
}

const exitAR = () => {
  showAR.value = false
}

onBeforeUnmount(() => {

  showAR.value = false
})

onBeforeUnmount(() => {

  showAR.value = false
})
</script>

<template>
  <div class="ar-container">

    <div v-if="loading" class="loading-overlay">
      <div class="loader"></div>
      <p>Cargando AR...</p>
    </div>

    <div v-if="error" class="error-message">
      {{ error }}
    </div>


    <div v-if="showAR" class="ar-scene-container">
      <a-scene embedded
        arjs="trackingMethod: best; sourceType: webcam; debugUIEnabled: false; detectionMode: mono_and_matrix;"
        renderer="logarithmicDepthBuffer: true; precision: medium;" vr-mode-ui="enabled: false">

        <a-entity camera></a-entity>


        <a-marker preset="hiro">
          
          <a-entity 
            gltf-model="https://cdn.aframe.io/examples/models/duck/duck.gltf" 
            scale="0.1 0.1 0.1" 
            position="0 0 0" 
            rotation="0 0 0">
          </a-entity>

          
          <a-entity light="type: ambient; color: #FFF; intensity: 0.8"></a-entity>
        </a-marker>

        
        <div class="exit-button-container">
          <button @click="exitAR" class="exit-button">
            Salir de AR
          </button>
        </div>
      </a-scene>
    </div>

   
    <button v-if="!showAR && !loading" @click="startAR"
      class="bg-red-600 shadow-md text-white px-4 py-2 text-lg rounded-md">
      Iniciar AR con marcador
    </button>
  </div>
</template> 

<style scoped>
.ar-container {
  position: relative;
}

.ar-scene-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 9999;
}

.exit-button-container {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 10000;
}

.exit-button {
  background-color: rgba(255, 0, 0, 0.7);
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  border: none;
}

.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color: white;
  z-index: 9999;
}

.loader {
  border: 5px solid #f3f3f3;
  border-top: 5px solid #3498db;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 2s linear infinite;
  margin-bottom: 16px;
}

.error-message {
  background-color: rgba(255, 0, 0, 0.7);
  color: white;
  padding: 16px;
  border-radius: 8px;
  margin: 16px 0;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}
</style>