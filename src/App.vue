<script setup lang="ts">
import { useDeviceOrientation } from '@vueuse/core';
import { computed, onMounted, useTemplateRef, watch } from 'vue';

const orientation = useDeviceOrientation()

const north = useTemplateRef("north")

const alphaCorrected = computed(() => (360 - (orientation.alpha.value ?? 0)))

watch(alphaCorrected, () => {
  if (north.value) {
    north.value.style.rotate = `${(360 - (alphaCorrected.value ?? 0)).toFixed(2)}deg`
  }
}) 

function requestOrientationPermission(){
    if ( typeof( DeviceOrientationEvent ) !== "undefined" && "requestPermission" in DeviceOrientationEvent && typeof( DeviceOrientationEvent.requestPermission ) === "function" )
    {
      DeviceOrientationEvent.requestPermission()
        .then((response: any) => {
            if (response == 'granted') {
                window.addEventListener('deviceorientation', (e) => {
                    // do something with e
                })
            }
        })
        .catch(console.error)
    }
}

</script>

<template>
  <header>
    <h1>Compass</h1>
  </header>

  <main>
    <h2>{{ alphaCorrected?.toFixed(1) }}</h2>
    N
    <div id="north" ref="north">⬆️</div>
    S
    <button style="margin-top: 1rem;" @click="requestOrientationPermission">Give Permissions</button>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

#north {
  font-size: 3rem;
}

main {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
