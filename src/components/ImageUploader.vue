<template>
  <div class="image-uploader">
    <!-- <div class="background-animation"></div> -->
    <div class="content">
      <h1 class="title">Human Image Validator</h1>
      <p class="description">
        Upload an image to detect if it contains a human face. Drag and drop or click to select an image.
      </p>

      <div class="drop-zone" @dragover.prevent @drop.prevent="onDrop" @click="selectImage">
        <p v-if="!image" class="drop-text">Drag and drop an image here, or click to select</p>
        <img v-if="image" :src="imagePreview" alt="Preview" class="preview-image" />
      </div>

      <input type="file" ref="fileInput" class="hidden" accept="image/*" @change="onFileSelect" />

      <button class="upload-button" :disabled="!image" @click="uploadImage">
        Upload
      </button>
      <button v-if="image" @click="clearImage" class="clear">Clear</button>
      <div v-if="isLoading" class="loader-overlay">
        <div class="magic-stick-loader"></div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import VueLoading from "vue-loading-overlay";
import "sweetalert2/dist/sweetalert2.min.css";
import "animate.css";

export default defineComponent({
  name: "ImageUploader",
  components: { VueLoading },
  setup() {
    const image = ref<File | null>(null);
    const imagePreview = ref('');
    const fileInput = ref<HTMLInputElement | null>(null);
    const isLoading = ref(false);

    const selectImage = () => {
      fileInput.value?.click();
    };

    const clearImage = () => {
      image.value = null;
      imagePreview.value = '';
    };

    const onFileSelect = (event: Event) => {
      const files = (event.target as HTMLInputElement).files;
      if (files && files[0]) {
        setImage(files[0]);
      }
    };

    const onDrop = (event: DragEvent) => {
      if (event.dataTransfer && event.dataTransfer.files[0]) {
        setImage(event.dataTransfer.files[0]);
      }
    };

    const setImage = (file: File) => {
      if (file.type.startsWith("image/")) {
        image.value = file;
        imagePreview.value = URL.createObjectURL(file);
      } else {
        Swal.fire({
          icon: "error",
          title: "Invalid File",
          text: "Please upload a valid image file!",
        });
      }
    };

    const uploadImage = async () => {
      if (!image.value) return;
      isLoading.value = true;
      const formData = new FormData();
      formData.append("image", image.value);

      isLoading.value = true;

      try {
        const response = await axios.post(
          "http://127.0.0.1:8000/validated-images/",
          formData,
          { headers: { "Content-Type": "multipart/form-data" } }
        );

        if (response.status === 201) {
          const { is_valid, validation_message } =
            response.data;

          Swal.fire({
            icon: is_valid ? "success" : "error",
            title: is_valid ? "Validation Successful" : "Validation Failed",
            text: validation_message,
            showClass: {
              popup: "animate__animated animate__fadeInDown",
            },
            hideClass: {
              popup: "animate__animated animate__fadeOutUp",
            },
            timer: 3000, // Optional: auto-close after 2 seconds
            timerProgressBar: true, // Show progress bar for timer
          });
        }
      } catch (error: any) {
        if (error.response && error.response.status === 400) {
          const { is_valid, validation_message } = error.response.data;
          if (is_valid === false) {
            Swal.fire({
              // icon: "error",
              title: "🤔",
              text: validation_message,
              showClass: {
                popup: "animate__animated animate__fadeInDown",
              },
              hideClass: {
                popup: "animate__animated animate__fadeOutUp",
              },
            });
          }
        } else {
          Swal.fire({
            icon: "error",
            title: "Unexpected Error",
            text: "An unexpected error occurred. Please try again.",
            showClass: {
              popup: "animate__animated animate__fadeInDown",
            },
            hideClass: {
              popup: "animate__animated animate__fadeOutUp",
            },
          });
        }
      } finally {
        clearImage();
        isLoading.value = false;
      }
    };

    return {
      image,
      clearImage,
      imagePreview,
      fileInput,
      isLoading,
      selectImage,
      onFileSelect,
      onDrop,
      uploadImage,
    };
  },
});
</script>

<style scoped>
/* Styling for the component */
.image-uploader {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  /* min-height: 100vh; */
  padding: 20px;
  overflow: hidden;
}

.background-animation {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #6a11cb, #6a11cb);
  background-size: 400% 400%;
  z-index: -1;
  animation: gradient 5s ease infinite;
}

@keyframes gradient {
  0% {
    background-position: 0% 50%;
  }

  50% {
    background-position: 100% 50%;
  }

  100% {
    background-position: 0% 50%;
  }
}

.content {
  text-align: center;
  background: rgba(255, 255, 255, 0.85);
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

.title {
  font-size: 2rem;
  color: #333;
  margin-bottom: 10px;
}

.description {
  font-size: 1rem;
  color: #555;
  margin-bottom: 20px;
}

.drop-zone {
  width: 100%;
  max-width: 100%;
  height: 500px;
  border: 2px dashed #4f46e5;
  background: #eef2ff;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
}

.drop-zone:hover {
  background: #e0e7ff;
}

.drop-text {
  color: #4f46e5;
}

.preview-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: cover;
  border-radius: 8px;
}

.upload-button {
  margin-top: 20px;
  padding: 10px 20px;
  color: white;
  background-color: #4f46e5;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
}

.upload-button:hover {
  background-color: #3b82f6;
}

.hidden {
  display: none;
}

button.clear {
  background-color: #f44336;
  /* Red background */
  color: #fff;
  /* White text */
  margin-left: 10px;
  /* Space between buttons */
}

button.clear:hover {
  background-color: #d32f2f;
  /* Darker red on hover */
}

button.clear:disabled {
  background-color: #f9bdbb;
  /* Light red for disabled state */
  cursor: not-allowed;
}

.loader-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.6);
  /* Semi-transparent background */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
}

.magic-stick-loader {
  width: 80px;
  height: 8px;
  background: linear-gradient(90deg, #ff6ec4, #7873f5);
  border-radius: 4px;
  position: relative;
  animation: magic-stick 1.5s infinite ease-in-out;
}

@keyframes magic-stick {
  0% {
    transform: scaleX(0.2);
    box-shadow: 0 0 8px #ff6ec4, 0 0 12px #7873f5;
  }

  50% {
    transform: scaleX(1);
    box-shadow: 0 0 15px #ff6ec4, 0 0 20px #7873f5;
  }

  100% {
    transform: scaleX(0.2);
    box-shadow: 0 0 8px #ff6ec4, 0 0 12px #7873f5;
  }
}
</style>
