<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const showAR = ref(false)
const loading = ref(false)
const error = ref(null)

const startAR = async () => {
    loading.value = true
    error.value = null

    try {
       
        await Promise.all([
            loadScript('https://aframe.io/releases/1.4.2/aframe.min.js'),
            loadScript('https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar-nft.js'),
            loadScript('https://raw.githack.com/donmccurdy/aframe-extras/master/dist/aframe-extras.min.js')
        ])

        setTimeout(() => {
            showAR.value = true
            loading.value = false
        }, 1000)
    } catch (err) {
        error.value = `Error al cargar scripts: ${err.message}`
        loading.value = false
        console.error('Error cargando scripts AR:', err)
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

const registerComponents = () => {
    if (window.AFRAME) {
        AFRAME.registerComponent('gesture-handler', {
            schema: {
                enabled: { default: true },
                rotationFactor: { default: 5 },
                minScale: { default: 0.3 },
                maxScale: { default: 8 }
            },

            init: function () {
                this.handleScale = this.handleScale.bind(this)
                this.handleRotation = this.handleRotation.bind(this)

                this.isVisible = false
                this.initialScale = this.el.object3D.scale.clone()
                this.scaleFactor = 1

                this.el.sceneEl.addEventListener('markerFound', () => {
                    this.isVisible = true
                })

                this.el.sceneEl.addEventListener('markerLost', () => {
                    this.isVisible = false
                })
            },

            update: function () {
                if (this.data.enabled) {
                    this.el.sceneEl.addEventListener('touchmove', this.handleRotation)
                    this.el.sceneEl.addEventListener('wheel', this.handleScale)
                } else {
                    this.el.sceneEl.removeEventListener('touchmove', this.handleRotation)
                    this.el.sceneEl.removeEventListener('wheel', this.handleScale)
                }
            },

            handleRotation: function (event) {
                if (this.isVisible) {
                    this.el.object3D.rotation.y += event.movementX / this.data.rotationFactor
                }
            },

            handleScale: function (event) {
                if (this.isVisible) {
                    this.scaleFactor -= event.deltaY / 1000
                    this.scaleFactor = Math.min(Math.max(this.scaleFactor, this.data.minScale), this.data.maxScale)

                    this.el.object3D.scale.x = this.scaleFactor * this.initialScale.x
                    this.el.object3D.scale.y = this.scaleFactor * this.initialScale.y
                    this.el.object3D.scale.z = this.scaleFactor * this.initialScale.z
                }
            },

            remove: function () {
                this.el.sceneEl.removeEventListener('touchmove', this.handleRotation)
                this.el.sceneEl.removeEventListener('wheel', this.handleScale)
            }
        })
    }
}

onMounted(() => {
    if (process.client) {
        registerComponents()
    }
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
            <a-scene embedded vr-mode-ui="enabled: false" device-orientation-permission-ui="enabled: false"
                renderer="logarithmicDepthBuffer: true; precision: medium; antialias: true"
                arjs="sourceType: webcam; debugUIEnabled: false; detectionMode: mono_and_matrix;">

                
                <a-entity camera look-controls="pointerLockEnabled: false" position="0 0 0">
                </a-entity>

                
                <a-entity id="world-anchor" position="0 0 -3">
                   
                    <a-entity id="model" position="0 0 0" rotation="0 0 0" scale="0.1 0.1 0.1"
                        gltf-model="/sprites/SamsungS10.gltf"
                        animation="property: rotation; to: 0 360 0; loop: true; dur: 10000; easing: linear"
                        animation-mixer="clip: *; loop: repeat" gesture-handler>
                    </a-entity>

                    
                    <a-entity light="type: ambient; color: #FFF; intensity: 0.8"></a-entity>
                    <a-entity light="type: directional; color: #FFF; intensity: 0.6" position="-1 1 1"></a-entity>
                </a-entity>
            </a-scene>

           
            <div class="ar-ui">
                <button @click="exitAR" class="exit-button">
                    Salir de AR
                </button>
                <div class="instructions">
                    Mueve tu dispositivo para ver el objeto desde diferentes ángulos
                </div>
            </div>
        </div>

        
        <button v-if="!showAR && !loading" @click="startAR"
            class="bg-green-600 hover:bg-green-700 shadow-md text-white px-4 py-2 text-lg rounded-md transition">
            Iniciar AR sin marcador
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

.ar-ui {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    padding: 16px;
    display: flex;
    flex-direction: column;
    align-items: center;
    z-index: 10000;
}

.exit-button {
    background-color: rgba(255, 0, 0, 0.7);
    color: white;
    padding: 8px 16px;
    border-radius: 4px;
    border: none;
    font-size: 16px;
    margin-bottom: 16px;
}

.instructions {
    background-color: rgba(0, 0, 0, 0.6);
    color: white;
    padding: 8px 16px;
    border-radius: 4px;
    text-align: center;
    max-width: 80%;
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