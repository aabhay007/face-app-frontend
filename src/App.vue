<template>
  <div>
    <ImageUploader />
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import ImageUploader from "./components/ImageUploader.vue";

export default defineComponent({
  name: "App",
  components: { ImageUploader },
});
</script>
