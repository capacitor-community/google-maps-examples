<template>
  <div class="container col">
    <div ref="exampleMap" class="flex-grow" style="background: purple"></div>

    <div class="row gap-4">
      <CButton v-if="!exampleMapId" @click="startMap"> CreateMap </CButton>
      <CButton @click="addMarker"> AddMarker </CButton>
      <CButton @click="moveCamera"> MoveCamera </CButton>
    </div>
  </div>
</template>

<script setup lang="ts">
import { CapacitorGoogleMaps } from '@capacitor-community/google-maps';
import { ref, onMounted } from 'vue';

import CButton from './CButton.vue';

const exampleMapId = ref<string | null>(null);
const exampleMap = ref<InstanceType<typeof HTMLElement> | null>(null);

const getRandomArbitrary = (min: number, max: number) => {
  return Math.random() * (max - min) + min;
};

const startMap = async () => {
  // if map already (being) initialized, avoid a new map being initialized
  if (exampleMapId.value != null) {
    return;
  }

  // find element where element should be attached to
  const element = exampleMap.value;
  if (!element) {
    return;
  }

  // prevent race condition, where firing `startMap` quickly after one another, starts up multiple maps
  exampleMapId.value = '';

  // remove background, so map can be seen
  element.style.background = '';

  // get boundaries of element
  const boundingRect = element.getBoundingClientRect();

  // finally create the map
  try {
    const result = await CapacitorGoogleMaps.createMap({
      boundingRect: {
        width: Math.round(boundingRect.width),
        height: Math.round(boundingRect.height),
        x: Math.round(boundingRect.x),
        y: Math.round(boundingRect.y),
      },
      cameraPosition: {
        target: {
          latitude: -33.86,
          longitude: 151.2,
        },
        zoom: 12,
      },
      preferences: {
        appearance: {
          isTrafficShown: true,
        },
      },
    });

    // remember mapId for curent element for later use
    exampleMapId.value = result.googleMap.mapId;

    // set `data-maps-id` attribute for delegating touch events
    element.setAttribute('data-maps-id', exampleMapId.value);

    // remove background, so map can be seen
    element.style.background = '';

    alert('map created: ' + JSON.stringify(result, null, 1));

    // since we now know the mapId, we can start listening to events

    CapacitorGoogleMaps.didTapMarker(
      {
        mapId: exampleMapId.value,
      },
      () => {
        // this will never fire, because it is overwritten by the next `didTapMarker` listener
        alert('this will never be shown');
      }
    );

    CapacitorGoogleMaps.didTapMarker(
      {
        mapId: exampleMapId.value,
        preventDefault: true, // set to true if you do not want the default behaviour
      },
      (result) => {
        // but this will fire
        alert('didTapMarker: ' + JSON.stringify(result, null, 1));
      }
    );

    CapacitorGoogleMaps.didEndMovingCamera(
      {
        mapId: exampleMapId.value,
      },
      (result) => {
        const latitude = result.cameraPosition.target?.latitude;
        const longitude = result.cameraPosition.target?.longitude;
      }
    );
  } catch (e) {
    console.error(e);
    exampleMapId.value = null;
  }
};

const addMarker = async () => {
  if (!exampleMapId.value) {
    return;
  }

  const result = await CapacitorGoogleMaps.addMarker({
    mapId: exampleMapId.value,
    position: {
      latitude: -getRandomArbitrary(33, 34),
      longitude: getRandomArbitrary(151, 152),
    },
    preferences: {
      title: 'Some title',
      snippet: 'Some snippet',
      metadata: {
        some: 'data',
      },
    },
  });

  alert('marker created: ' + JSON.stringify(result, null, 1));
};

const moveCamera = async () => {
  if (!exampleMapId.value) {
    return;
  }

  await CapacitorGoogleMaps.moveCamera({
    mapId: exampleMapId.value,
    cameraPosition: {
      target: {
        latitude: -33.86,
        longitude: 151.2,
      },
      zoom: 0,
    },
    duration: 1000,
  });
};

onMounted(async () => {
  // initialize map
  await CapacitorGoogleMaps.initialize({
    key: 'your_api_key',
    devicePixelRatio: window.devicePixelRatio,
  });
});
</script>

<style scoped>
.row {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-evenly;
}

.col {
  display: flex;
  flex-direction: column;
}

.flex-grow {
  flex-grow: 1;
}

.gap-4 {
  gap: 1rem;
}
</style>
