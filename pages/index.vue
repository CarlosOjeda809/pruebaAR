
<script setup>
import { ref } from 'vue'
import ARNoMarker from '~/components/ARNoMarker.vue'
</script>

<template>
  <div class="p-4 max-w-2xl mx-auto">
    <header class="mb-6 text-center">
      <h1 class="font-bold text-2xl mb-2">Explora en Realidad Aumentada</h1>
      <p class="text-gray-600">Visualiza el objeto 3D en tu entorno real</p>
    </header>

    <!-- Componente AR sin marcador -->
    <ARNoMarker />

    <!-- Instrucciones adicionales -->
    <div class="mt-6 bg-blue-50 p-4 rounded-lg">
      <h3 class="font-semibold text-lg mb-2">Consejos para mejor experiencia AR:</h3>
      <ul class="list-disc pl-5 text-gray-700 space-y-1">
        <li>Usa tu dispositivo en un ambiente bien iluminado</li>
        <li>Permite todos los permisos de cámara cuando se soliciten</li>
        <li>Mueve tu dispositivo lentamente al principio para que el sistema AR pueda detectar las superficies</li>
        <li>Puedes tocar la pantalla para rotar el objeto</li>
        <li>Usa dos dedos (pellizco) para cambiar el tamaño del objeto</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
/* Estilos adicionales si son necesarios */
</style>