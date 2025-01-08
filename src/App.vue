<template>
  <div class="toggle-container">
    <div class="toggle-buttons">
      <button
        :class="{ active: activeComponent === 'uploader' }"
        @click="setActiveComponent('uploader')"
      >
        Upload Image
      </button>
      <button
        :class="{ active: activeComponent === 'webcam' }"
        @click="setActiveComponent('webcam')"
      >
        Webcam Capture
      </button>
    </div>

    <div class="component-container">
      <ImageUploader v-if="activeComponent === 'uploader'" />
      <WebcamCapture v-else-if="activeComponent === 'webcam'" />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import ImageUploader from "./components/ImageUploader.vue";
import WebcamCapture from "./components/WebcamCapture.vue";

export default defineComponent({
  name: "ParentComponent",
  components: { ImageUploader, WebcamCapture },
  setup() {
    const activeComponent = ref<"uploader" | "webcam">("uploader");

    const setActiveComponent = (component: "uploader" | "webcam") => {
      activeComponent.value = component;
    };

    return {
      activeComponent,
      setActiveComponent,
    };
  },
});
</script>

<style scoped>
.toggle-container {
  text-align: center;
  margin-top: 20px;
}

.toggle-buttons {
  margin-bottom: 20px;
}

.toggle-buttons button {
  margin: 0 10px;
  padding: 10px 20px;
  font-size: 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  background-color: #2d4cff;
  transition: background-color 0.3s ease, color 0.3s ease;
}

.toggle-buttons button.active {
  background-color: #6a11cb;
  color: white;
}

.toggle-buttons button:hover {
  background-color: #0056b3;
  color: white;
}

.component-container {
  margin-top: 20px;
}
</style>
