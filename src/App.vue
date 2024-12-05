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

/** Adapted from https://stackoverflow.com/a/21829819/5721675 */
function compassHeading(alpha: number, beta: number, gamma: number) {

  // Convert degrees to radians
  const alphaRad = alpha * (Math.PI / 180);
  const betaRad = beta * (Math.PI / 180);
  const gammaRad = gamma * (Math.PI / 180);

  // Calculate equation components
  const cA = Math.cos(alphaRad);
  const sA = Math.sin(alphaRad);
  const cB = Math.cos(betaRad);
  const sB = Math.sin(betaRad);
  const cG = Math.cos(gammaRad);
  const sG = Math.sin(gammaRad);

  // Calculate A, B, C rotation components
  const rA = - cA * sG - sA * sB * cG;
  const rB = - sA * sG + cA * sB * cG;
  const rC = - cB * cG;

  // Calculate compass heading
  let compassHeading = Math.atan(rA / rB);

  // Convert from half unit circle to whole unit circle
  if(rB < 0) {
    compassHeading += Math.PI;
  }else if(rA < 0) {
    compassHeading += 2 * Math.PI;
  }

  // Convert radians to degrees
  compassHeading *= 180 / Math.PI;

  return compassHeading;

}

function addEvent() {
  window.addEventListener('deviceorientation', (e) => {
      if (e.alpha && e.beta && e.gamma) {
        direction.value = compassHeading(e.alpha, e.beta, e.gamma)
      }
      lastEvent.value = `Alpha: ${e.alpha?.toFixed(2)}\nBeta: ${e.beta?.toFixed(2)}\nGamma: ${e.gamma?.toFixed(2)}`
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
    <p style="white-space: pre;">{{ lastEvent }}</p>
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
