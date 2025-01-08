<template>
    <div class="webcam-capture">
      <div class="content">
        <h1 class="title">Live Webcam Validator</h1>
        <p class="description">
          Use your webcam to capture a photo and validate if it contains a human face.
        </p>
  
        <div class="webcam-container">
          <video ref="video" autoplay playsinline class="webcam-feed"></video>
          <canvas ref="canvas" class="hidden"></canvas>
        </div>
        <button class="capture-button" @click="captureImage">Capture</button>
  
        <div v-if="imagePreview" class="image-preview">
          <img :src="imagePreview" alt="Captured Preview" />
        </div>
        <button v-if="imagePreview" @click="clearPreview" class="clear-button">Clear</button>
        <button v-if="imagePreview" @click="uploadImage" class="upload-button">Upload</button>
  
        <div v-if="isLoading" class="loader-overlay">
          <div class="magic-stick-loader"></div>
        </div>
      </div>
    </div>
  </template>
  
  <script lang="ts">
  import { defineComponent, ref, onMounted, onBeforeUnmount } from "vue";
  import axios from "axios";
  import Swal from "sweetalert2";
  import "sweetalert2/dist/sweetalert2.min.css";
  
  export default defineComponent({
    name: "WebcamCapture",
    setup() {
      const video = ref<HTMLVideoElement | null>(null);
      const canvas = ref<HTMLCanvasElement | null>(null);
      const imagePreview = ref("");
      const isLoading = ref(false);
      let stream: MediaStream | null = null;
  
      const startWebcam = async () => {
        try {
          stream = await navigator.mediaDevices.getUserMedia({ video: true });
          if (video.value) {
            video.value.srcObject = stream;
          }
        } catch (error) {
          console.error("Error accessing webcam:", error);
          Swal.fire({
            icon: "error",
            title: "Webcam Access Denied",
            text: "Please allow access to your webcam.",
          });
        }
      };
  
      const stopWebcam = () => {
        if (stream) {
          stream.getTracks().forEach((track) => track.stop());
        }
      };
  
      const captureImage = () => {
        if (video.value && canvas.value) {
          const context = canvas.value.getContext("2d");
          if (context) {
            canvas.value.width = video.value.videoWidth;
            canvas.value.height = video.value.videoHeight;
            context.drawImage(video.value, 0, 0);
            imagePreview.value = canvas.value.toDataURL("image/png");
          }
        }
      };
  
      const clearPreview = () => {
        imagePreview.value = "";
      };
  
      const uploadImage = async () => {
        if (!imagePreview.value) return;
  
        isLoading.value = true;
        try {
          const response = await axios.post(
            "http://127.0.0.1:8000/validated-images/webcam/",
            { image_data: imagePreview.value },
            { headers: { "Content-Type": "application/json" } }
          );
  
          if (response.status === 201) {
            const { is_valid, validation_message } = response.data;
  
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
              timer: 3000,
              timerProgressBar: true,
            });
          }
        } catch (error: any) {
          if (error.response && error.response.status === 400) {
            const { is_valid, validation_message } = error.response.data;
            if (is_valid === false) {
              Swal.fire({
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
          clearPreview();
          isLoading.value = false;
        }
      };
  
      onMounted(() => {
        startWebcam();
      });
  
      onBeforeUnmount(() => {
        stopWebcam();
      });
  
      return {
        video,
        canvas,
        imagePreview,
        isLoading,
        captureImage,
        clearPreview,
        uploadImage,
      };
    },
  });
  </script>
  
  <style scoped>
  .webcam-capture {
    text-align: center;
  }
  .webcam-feed {
    width: 100%;
    max-width: 500px;
    border: 2px solid #ddd;
    border-radius: 8px;
    margin-bottom: 10px;
  }
  .capture-button,
  .upload-button,
  .clear-button {
    margin: 10px;
  }
  .hidden {
    display: none;
  }
  .image-preview img {
    width: 100%;
    max-width: 300px;
    margin: 10px auto;
  }
  .loader-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.6);
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
  