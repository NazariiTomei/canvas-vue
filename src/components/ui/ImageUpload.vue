<template>
  <div class="upload-file">
    <input type="file" @change="onFileChange" accept="image/*">
    <button class="btn-blue" @click="uploadImage" :disabled="!selectedFile">Upload</button>
    <div v-if="imageUrl">
      <img :src="imageUrl" alt="Image Preview" style="width: 200px;" />
    </div>
    <h2 style="color: black;">Or Choose from your recent images</h2>
    <template v-if="recentImages">
      <div class="grid-container">
        <div v-for="(image, index) in recentImages" :key="index" :class="{ 'grid-item': true }">
          <img :src="image.url" alt="Image Preview" style="width: 200px;" />
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import axios from 'axios';
import { ref, toRefs } from 'vue';

export default {
  name: 'ImageUpload',
  props: {
    uploadedFileName: {
      type: Object,
    },
    imageUploaded: {
      type: Function,
    }
  },
  setup(props) {
    const { uploadedFileName } = toRefs(props);
    const { imageUploaded } = toRefs(props);
    const selectedFile = ref(null);
    const imageUrl = ref('');
    const recentImages = ref([]);

    const onFileChange = (event) => {
      const file = event.target.files[0];
      if (file) {
        selectedFile.value = file;
        const reader = new FileReader();
        reader.onload = (e) => {
          imageUrl.value = e.target.result;

        };
        reader.readAsDataURL(file);
      }
    };

    const uploadImage = async () => {
      if (!selectedFile.value) return;

      const formData = new FormData();
      formData.append('file', selectedFile.value);

      try {
        const response = await axios.post('http://localhost:5000/upload',
          formData,
          {
            headers: {
              'Content-Type': 'multipart/form-data',
            },
          });
        const data = await response.data;
        uploadedFileName.url = data.filePath;
        props.imageUploaded(uploadedFileName);
      } catch (error) {
        console.error('Error uploading image:', error);
      }
    };

    return {
      selectedFile,
      imageUrl,
      onFileChange,
      uploadImage,
    };
  },
};
</script>

<style>
img {
  max-width: 100%;
  height: auto;
}

.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  padding: 10px;
}

.grid-item {
  display: inline-block;
}
</style>