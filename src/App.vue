<script setup lang="ts">
import { useDeviceOrientation } from '@vueuse/core';
import { computed, onMounted, ref, useTemplateRef, watch } from 'vue';

const north = useTemplateRef("north")

const direction = ref<number>()
const lastEvent = ref<string>()

watch(direction, () => {
  if (north.value) {
    north.value.style.rotate = `${(360 - (direction.value ?? 0)).toFixed(2)}deg`
  }
}) 

function addEvent() {
  window.addEventListener('deviceorientation', (e) => {
      direction.value = 360 - (e.alpha ?? 0)
      if ("webkitCompassHeading" in e) {
        direction.value = e.webkitCompassHeading as number ?? 0
      }
      lastEvent.value = JSON.stringify(e)
  })
}

function requestOrientationPermission(){
    if ( typeof( DeviceOrientationEvent ) !== "undefined" && "requestPermission" in DeviceOrientationEvent && typeof( DeviceOrientationEvent.requestPermission ) === "function" )
    {
      DeviceOrientationEvent.requestPermission()
        .then((response: any) => {
            if (response == 'granted') {
                addEvent()
            }
        })
        .catch(console.error)
    } else {
      addEvent()
    }
}

</script>

<template>
  <header>
    <h1>Compass</h1>
  </header>

  <main>
    <h2>{{ direction?.toFixed(1) }}</h2>
    N
    <div id="north" ref="north">⬆️</div>
    S
    <button style="margin-top: 1rem;" @click="requestOrientationPermission">Give Permissions</button>
    <p>{{ lastEvent }}</p>
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
