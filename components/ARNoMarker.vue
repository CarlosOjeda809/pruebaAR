<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const showAR = ref(false)
const loading = ref(false)
const error = ref(null)
const modelLoaded = ref(false)

const webXRSupported = ref(false)

const loadScripts = async () => {
    try {

        await loadScript('https://aframe.io/releases/1.4.2/aframe.min.js')


        await new Promise(resolve => setTimeout(resolve, 300))

        await loadScript('https://unpkg.com/aframe-extras@6.1.1/dist/aframe-extras.min.js')


        if (navigator.xr) {
            try {
                webXRSupported.value = await navigator.xr.isSessionSupported('immersive-ar')
            } catch (err) {
                console.warn('Error al comprobar soporte WebXR:', err)
                webXRSupported.value = false
            }
        }


        if (webXRSupported.value) {
            await loadScript('https://unpkg.com/aframe-ar@0.2.0/dist/aframe-ar.min.js')
        } else {
            await loadScript('https://raw.githack.com/AR-js-org/AR.js/master/aframe/build/aframe-ar-nft.js')
        }

        return true
    } catch (err) {
        console.error('Error cargando scripts:', err)
        throw err
    }
}

const startAR = async () => {
    loading.value = true
    error.value = null

    try {
        await loadScripts()


        registerComponents()

        setTimeout(() => {
            showAR.value = true
            loading.value = false
        }, 1000)
    } catch (err) {
        error.value = `Error al iniciar AR: ${err.message}`
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
        script.onerror = () => reject(new Error(`No se pudo cargar: ${src}`))
        document.head.appendChild(script)
    })
}

const exitAR = () => {
    showAR.value = false
}

const registerComponents = () => {
    if (window.AFRAME) {

        AFRAME.registerComponent('model-loaded', {
            init: function () {
                this.el.addEventListener('model-loaded', () => {
                    modelLoaded.value = true
                    console.log('Modelo cargado correctamente')
                })

                this.el.addEventListener('model-error', (error) => {
                    console.error('Error cargando modelo:', error)
                })
            }
        })


        AFRAME.registerComponent('model-interaction', {
            schema: {
                rotationSpeed: { type: 'number', default: 0.2 }
            },

            init: function () {
                this.rotation = { x: 0, y: 0 }
                this.position = { x: 0, y: 0 }
                this.scale = 1.0
                this.touching = false
                this.touchPoints = []

                document.addEventListener('touchstart', this.onTouchStart.bind(this), { passive: true })
                document.addEventListener('touchmove', this.onTouchMove.bind(this), { passive: true })
                document.addEventListener('touchend', this.onTouchEnd.bind(this), { passive: true })


                document.addEventListener('mousedown', this.onMouseDown.bind(this))
                document.addEventListener('mousemove', this.onMouseMove.bind(this))
                document.addEventListener('mouseup', this.onMouseUp.bind(this))


                document.addEventListener('wheel', this.onWheel.bind(this), { passive: true })
            },

            onTouchStart: function (event) {
                this.touching = true
                this.touchPoints = []

                for (let i = 0; i < event.touches.length; i++) {
                    this.touchPoints.push({
                        x: event.touches[i].pageX,
                        y: event.touches[i].pageY
                    })
                }
            },

            onTouchMove: function (event) {
                if (!this.touching) return


                if (event.touches.length === 1 && this.touchPoints.length === 1) {
                    const dx = event.touches[0].pageX - this.touchPoints[0].x
                    this.rotation.y += dx * 0.01
                    this.el.object3D.rotation.y = this.rotation.y

                    this.touchPoints[0] = {
                        x: event.touches[0].pageX,
                        y: event.touches[0].pageY
                    }
                }


                if (event.touches.length === 2 && this.touchPoints.length === 2) {
                    const dist1 = Math.hypot(
                        this.touchPoints[0].x - this.touchPoints[1].x,
                        this.touchPoints[0].y - this.touchPoints[1].y
                    )

                    const dist2 = Math.hypot(
                        event.touches[0].pageX - event.touches[1].pageX,
                        event.touches[0].pageY - event.touches[1].pageY
                    )

                    const scaleFactor = dist2 / dist1
                    if (scaleFactor > 0.5 && scaleFactor < 2) {
                        this.scale *= scaleFactor
                        this.scale = Math.min(Math.max(this.scale, 0.1), 5)
                        this.el.object3D.scale.set(this.scale, this.scale, this.scale)
                    }

                    this.touchPoints = [
                        { x: event.touches[0].pageX, y: event.touches[0].pageY },
                        { x: event.touches[1].pageX, y: event.touches[1].pageY }
                    ]
                }
            },

            onTouchEnd: function () {
                this.touching = false
            },

            onMouseDown: function (event) {
                this.touching = true
                this.touchPoints = [{ x: event.pageX, y: event.pageY }]
            },

            onMouseMove: function (event) {
                if (!this.touching || this.touchPoints.length === 0) return

                const dx = event.pageX - this.touchPoints[0].x
                this.rotation.y += dx * 0.01
                this.el.object3D.rotation.y = this.rotation.y

                this.touchPoints[0] = { x: event.pageX, y: event.pageY }
            },

            onMouseUp: function () {
                this.touching = false
            },

            onWheel: function (event) {
                // No podemos usar preventDefault en eventos pasivos
                const scaleFactor = 1 + Math.sign(event.deltaY) * -0.05
                this.scale *= scaleFactor
                this.scale = Math.min(Math.max(this.scale, 0.1), 5)
                this.el.object3D.scale.set(this.scale, this.scale, this.scale)
            },

            remove: function () {
                document.removeEventListener('touchstart', this.onTouchStart)
                document.removeEventListener('touchmove', this.onTouchMove)
                document.removeEventListener('touchend', this.onTouchEnd)
                document.removeEventListener('mousedown', this.onMouseDown)
                document.removeEventListener('mousemove', this.onMouseMove)
                document.removeEventListener('mouseup', this.onMouseUp)
                document.removeEventListener('wheel', this.onWheel)
            }
        })
    }
}

onBeforeUnmount(() => {
    showAR.value = false
})
</script>

<template>
    <div class="ar-container">
        <!-- Loader y estado de carga -->
        <div v-if="loading" class="loading-overlay">
            <div class="loader"></div>
            <p>Inicializando experiencia AR...</p>
        </div>


        <div v-if="error" class="error-message">
            {{ error }}
        </div>


        <div v-if="showAR" class="ar-scene-container">

            <a-scene embedded vr-mode-ui="enabled: false" device-orientation-permission-ui="enabled: true"
                renderer="antialias: true; logarithmicDepthBuffer: true; colorManagement: true; precision: highp;"
                arjs="sourceType: webcam; debugUIEnabled: false; trackingMethod: best;">


                <a-entity camera look-controls></a-entity>


                <a-entity position="0 0 -3" rotation="0 0 0">

                    <a-entity id="movil samsung" gltf-model="SamsungS10.gltf" position="0 0 0"
                        rotation="0 0 0" scale="0.1 0.1 0.1" animation-mixer="clip: *; loop: repeat" model-loaded
                        model-interaction>
                    </a-entity>


                    <a-entity light="type: ambient; color: #FFF; intensity: 0.8"></a-entity>
                    <a-entity light="type: directional; color: #FFF; intensity: 0.6" position="-1 1 1"></a-entity>
                </a-entity>
            </a-scene>


            <div class="ar-interface">

                <button @click="exitAR" class="exit-button">
                    Salir de AR
                </button>


                <div v-if="!modelLoaded" class="model-loading">
                    Cargando modelo 3D...
                </div>


                <div class="instructions">
                    <div>Mueve el dispositivo para explorar</div>
                    <div>Toca para rotar | Pellizca para escalar</div>
                </div>
            </div>
        </div>


        <button v-if="!showAR && !loading" @click="startAR" class="start-button">
            Iniciar experiencia AR sin marcador
        </button>
    </div>
</template>

<style scoped>
.ar-container {
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
}

.ar-scene-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    z-index: 9999;
}

.ar-interface {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    padding: 20px;
    z-index: 10000;
    display: flex;
    flex-direction: column;
    align-items: center;
}

.exit-button {
    background-color: rgba(255, 0, 0, 0.7);
    color: white;
    padding: 12px 20px;
    border-radius: 8px;
    border: none;
    font-size: 16px;
    margin-bottom: 16px;
    cursor: pointer;
}

.instructions {
    background-color: rgba(0, 0, 0, 0.6);
    color: white;
    padding: 12px 20px;
    border-radius: 8px;
    text-align: center;
    max-width: 300px;
    margin-top: 10px;
    line-height: 1.5;
}

.model-loading {
    background-color: rgba(0, 0, 0, 0.6);
    color: yellow;
    padding: 12px 20px;
    border-radius: 8px;
    text-align: center;
    margin-bottom: 10px;
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
    max-width: 80%;
    text-align: center;
}

.start-button {
    background-color: #4CAF50;
    color: white;
    padding: 12px 24px;
    border-radius: 8px;
    border: none;
    font-size: 16px;
    cursor: pointer;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    transition: all 0.3s ease;
}

.start-button:hover {
    background-color: #45a049;
    transform: translateY(-2px);
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
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