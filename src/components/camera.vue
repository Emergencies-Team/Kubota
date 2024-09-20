<script setup lang="ts">
import { ref, onMounted } from "vue";
import frameSrc from "../assets/frame/vertical-frame-croped.png";

const showModal = ref(false);
const capturedImage = ref<string | null>(null);
const videoRef = ref<HTMLVideoElement | null>(null);

const startCamera = async () => {
  try {
    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
      const stream = await navigator.mediaDevices.getUserMedia({ video: true });
      if (videoRef.value) {
        videoRef.value.srcObject = stream;
        videoRef.value.play();
      }
    } else {
      console.error("getUserMedia not supported on this browser.");
    }
  } catch (error) {
    console.error("Error accessing the camera: ", error);
  }
};
const captureImage = () => {
  console.log("Capture started!");

  if (videoRef.value) {
    // Use the frame's resolution for the canvas
    const canvasWidth = 4688; // Frame width
    const canvasHeight = 9059; // Frame height

    const canvas = document.createElement("canvas");
    const ctx = canvas.getContext("2d");

    if (ctx) {
      canvas.width = canvasWidth;
      canvas.height = canvasHeight;

      console.log("Canvas dimensions: ", canvasWidth, canvasHeight);

      // Flip the context horizontally for the video
      ctx.save();
      ctx.translate(canvasWidth, 0);
      ctx.scale(-1, 1);

      // Draw the video frame on the canvas, scaling up the video to match the canvas size
      ctx.drawImage(
        videoRef.value, 
        0, 0, 
        canvasWidth, canvasHeight
      );

      // Restore the context to default state
      ctx.restore();

      const frame = new Image();
      frame.src = frameSrc; // Ensure this path is correct
      frame.onload = () => {
        console.log("Frame loaded!");

        // Draw the frame on top of the video
        ctx.drawImage(frame, 0, 0, canvasWidth, canvasHeight);

        // Save the image with the highest quality
        capturedImage.value = canvas.toDataURL("image/png", 1.0);
        showModal.value = true;
      };

      frame.onerror = () => {
        console.error("Failed to load frame image.");
      };
    } else {
      console.error("Failed to get canvas context.");
    }
  } else {
    console.error("Video element not found.");
  }
};

onMounted(() => {
  startCamera();
});
</script>

<template>
  <section
    class="w-full h-full bg-white-500 flex flex-col items-center justify-start overflow-hidden ยขถ"
  >
    <section class="relative w-full h-[80vh]">
      <video
        ref="videoRef"
        autoplay
        playsinline
        class="w-full h-full object-cover transform scale-x-[-1]"
      />
      <img
        src="../assets/frame/vertical-frame-croped.png"
        class="absolute inset-0 w-full h-full object-cover"
      />
    </section>

    <section class="w-full h-[20vh] items-center justify-center flex flex-row">
      <div class="h-full items-center justify-center flex w-1/3"></div>
      <div class="h-full items-center justify-center flex w-1/3">
        <button @click="captureImage">
          <img
            src="../assets/icon/capture.png"
            class="w-[5rem] aspect-square"
            alt="Capture"
          />
        </button>
      </div>
      <div class="h-full items-center justify-center flex w-1/3">
        <button>
          <img
            src="../assets/icon/switch.png"
            class="w-[3.5rem] aspect-square"
            alt="Switch"
          />
        </button>
      </div>
    </section>
    <div
      v-if="showModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center"
    >
      <div class="relative bg-white p-4">
        <button @click="showModal = false" class="absolute top-2 right-2">
          X
        </button>
        <img :src="capturedImage || ''" class="w-full h-auto" />
    </div>
    </div>
  </section>
</template>
