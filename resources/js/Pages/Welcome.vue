<template>
    <div class="container mx-auto p-4">
      <h1 class="text-2xl font-bold mb-4">Image Processor</h1>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div class="bg-gray-100 p-4 rounded-md">
          <h2 class="text-lg font-semibold mb-2">Upload Image</h2>
          <input
            type="file"
            @change="handleFileUpload"
            accept="image/*"
            class="mb-4 block w-full border border-gray-300 rounded-md px-2 py-1"
          />
          <div v-if="currentImage">
            <img :src="currentImage.processedSource || currentImage.source" alt="Original Image" class="w-full" />
            <div class="flex items-center mt-2">
              <label class="mr-2">Brightness:</label>
              <input
                type="range"
                min="-100"
                max="100"
                v-model="currentImage.brightness"
                @input="processImage()"
                class="flex-grow"
              />
            </div>
            <div class="flex items-center mt-2">
              <label class="mr-2">Sharpness:</label>
              <input
                type="range"
                min="0"
                max="10"
                v-model="currentImage.sharpness"
                @input="processImage()"
                class="flex-grow"
              />
            </div>
            <button
              v-if="currentImage.processedSource"
              @click="downloadProcessedImage(currentImage.processedSource)"
              class="mt-2 bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
            >
              Download
            </button>
          </div>
        </div>
        <div class="bg-gray-100 p-4 rounded-md">
          <h2 class="text-lg font-semibold mb-2">Processed Images</h2>
          <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
            <div v-for="(image, index) in processedImages" :key="index" class="mb-4">
              <img :src="image.processedSource" alt="Processed Image" class="w-full" />
              <button
                @click="downloadProcessedImage(image.processedSource)"
                class="mt-2 bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
              >
                Download
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, reactive } from 'vue';
  
  const images = ref([]);
  const currentImage = ref(null);
  const processedImages = ref([]);
  
  const handleFileUpload = (event) => {
    const file = event.target.files[0];
    const reader = new FileReader();
    reader.onload = () => {
      if (currentImage.value) {
        processedImages.value.push(currentImage.value);
      }
      currentImage.value = {
        source: reader.result,
        brightness: 0,
        sharpness: 0,
        processedSource: null,
      };
    };
    reader.readAsDataURL(file);
  };
  
  const processImage = () => {
    const canvas = document.createElement('canvas');
    const context = canvas.getContext('2d');
    const image = new Image();
    image.src = currentImage.value.source;
  
    image.onload = () => {
      canvas.width = image.width;
      canvas.height = image.height;
      context.filter = `brightness(${currentImage.value.brightness}%) contrast(${100 + (currentImage.value.sharpness * 20)}%)`;
      context.drawImage(image, 0, 0);
      currentImage.value.processedSource = canvas.toDataURL('image/jpeg');
    };
  };
  
  const downloadProcessedImage = (processedSource) => {
    const link = document.createElement('a');
    link.href = processedSource;
    link.download = 'processed_image.jpg';
    link.click();
  };
  </script>