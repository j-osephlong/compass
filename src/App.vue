<script setup lang="ts">
import { ref, useTemplateRef, watch } from 'vue';

const north = useTemplateRef("north")

const direction = ref<number>()
const lastEvent = ref<string>()

watch(direction, () => {
  if (north.value) {
    north.value.style.rotate = `${direction.value?.toFixed(2)}deg`
  }
}) 

/**Listeners adapted from https://stackoverflow.com/a/75792197/5721675 */
function addEvents() {
  let absoluteListener = (e: DeviceOrientationEvent) => {
    console.debug("ABS L")
    if (!e.absolute || e.alpha == null || e.beta == null || e.gamma == null)
      return;
    let compass = -(e.alpha + e.beta * e.gamma / 90);
    compass -= Math.floor(compass / 360) * 360; // Wrap into range [0,360].
    window.removeEventListener("deviceorientation", webkitListener);
    direction.value = compass
  };
  type WebkitDeviceOrientationEvent = DeviceOrientationEvent & { webkitCompassHeading: number }
  let webkitListener = (e: DeviceOrientationEvent) => {
    console.debug("WBK L")

    let compass = (e as WebkitDeviceOrientationEvent).webkitCompassHeading;
    if (compass!=null && !isNaN(compass)) {
      direction.value = compass
      window.removeEventListener("deviceorientationabsolute", absoluteListener);
    }
  }
  window.addEventListener("deviceorientation", webkitListener);
  window.addEventListener("deviceorientationabsolute", absoluteListener);
}

function requestOrientationPermission(){
    if ( typeof( DeviceOrientationEvent ) !== "undefined" && "requestPermission" in DeviceOrientationEvent && typeof( DeviceOrientationEvent.requestPermission ) === "function" )
    {
      DeviceOrientationEvent.requestPermission()
        .then((response: any) => {
            if (response == 'granted') {
                addEvents()
            }
        })
        .catch(console.error)
    } else {
      addEvents()
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
