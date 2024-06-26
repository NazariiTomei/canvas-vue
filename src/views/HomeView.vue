<template>
  <div class="home">
    <div class="canvas">
      <div v-if="loading" class="spinner-overlay">
        <div class="spinner"></div>
      </div>
      <div v-else>
        <div v-if="editing">
          <div class="image-cropper">
            <h1>Image Cropper</h1>
            <div class="cropper-container" style="display: flex;">
              <div v-if="cropperImage">
                <img ref="image" id="cropper-image" style="max-width: 800px; max-height: 600px; height: 600px"
                  :src="cropperImage" alt="Source Image" />
              </div>
              <div class="controls">
                <button @click="rotateImage(90)">Rotate 90°</button>
                <button @click="applyColorFilter('grayscale(100%)')">Grayscale</button>
                <button @click="saveCroppedImage">Save</button>
                <button @click="setEditing(false)">Cancel</button>
              </div>
            </div>

          </div>
        </div>
        <div v-else class="grid-container">
          <div v-for="(card, index) in cards" :key="card.id"
            :class="{ 'grid-item': true, 'display-none': index === 4 }">
            <template class="ItemCard" v-if="index !== 4">
              <div class="card" @click="handleOpen(card.id)">
                <div class="card-media">
                  <img :src="card.image === '' ? '/sample.jpeg' : card.image" alt="Image" />
                </div>
                <div v-if="card.image !== ''" class="image-list-item-bar">
                  <button class="icon-button" @click.stop="handleEdit(card.image)">ℹ️</button>
                </div>
              </div>
            </template>
          </div>

        </div>
      </div>
      <div v-if="!editing" class="controls-dash">
          <a @click="onRefresh" class="refresh">Refresh</a>
          <a class="preview">Preview</a>
      </div>
      <div v-if="openDialog" @click.self="closeModal" class="modal">
        <div :style="modalStyle">
          <button @click="handleClose" style="float: right;">Close</button>
          <h1 style="color: black;">Select Image</h1>
          <div class="upload-file">
            <input type="file" @change="onFileChange" accept="image/*" />
            <button class="btn-blue" @click="uploadImage" :disabled="!selectedFile">Upload</button>
            <div v-if="imageUrl">
              <img :src="imageUrl" alt="Image Preview" style="width: 240px; border-radius: 20px; height: 200px;" />
            </div>
            <div v-if="recentImages.length">
              <h2 style="color: black;">Or Choose from your recent images</h2>
              <div class="recent-grid-container" style="max-height: 280px; overflow-y: scroll;">
                <div v-for="(recentimage, index) in recentImages" :key="index" :class="{ 'grid-item': true }">
                  <img @click="imageUploaded(recentimage)" :src="recentimage" alt="Image Preview"
                    style="width: 240px; height: 210px; padding: 10px; border-radius: 20px;" />
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<script setup>
import { ref, computed, onMounted, onUnmounted, watch, nextTick } from 'vue';
import axios from 'axios';
import Cropper from 'cropperjs';
import 'cropperjs/dist/cropper.css';

const selectedFile = ref(null);
const openDialog = ref(false);
const selectId = ref(0);
const uploadStatus = ref('');
const recentImages = ref(JSON.parse(localStorage.getItem('recentImages')) || []);
const uploadedFileName = ref({});
const imageUrl = ref('');
const cropperImage = ref(null);
const loading = ref(false);
const editing = ref(false);
const editImage = ref('');
const cropper = ref(null);
const croppedImage = ref(null);
const cards = ref(JSON.parse(localStorage.getItem('cards')) || [
  { image: "", id: 0 },
  { image: "", id: 1 },
  { image: "", id: 2 },
  { image: "", id: 3 },
  { image: "", id: 4 },
  { image: "", id: 5 },
  { image: "", id: 6 },
  { image: "", id: 7 },
  { image: "", id: 8 },
]);

const handleOpen = (id) => {
  selectId.value = id;
  openDialog.value = true;
};

const handleEdit = (image) => {
  editImage.value = image;
  cropperImage.value = image;
  editing.value = true;
  nextTick(() => {
    const imageElement = document.getElementById('cropper-image');
    if (imageElement) {
      if (cropper.value) {
        cropper.value.destroy();
      }
      cropper.value = new Cropper(imageElement, {
        aspectRatio: 0,
        viewMode: 0
      });
    }
  });
};

const handleClose = () => {
  openDialog.value = false;
};

const imageUploaded = (filePath) => {
  uploadStatus.value = 'Upload successful';
  cards.value.forEach((item) => {
    if (item.id === selectId.value) {
      item.image = filePath;
    }
  });
  loading.value = false;
  handleClose();
  saveToLocalStorage();
};

const onFileChange = (event) => {
  const file = event.target.files[0];
  if (file) {
    selectedFile.value = file;
    const reader = new FileReader();
    reader.onload = (e) => {
      imageUrl.value = e.target.result;
      cropperImage.value = e.target.result;
      nextTick(() => {
        if (cropper.value) {
          cropper.value.destroy();
        }
        cropper.value = new Cropper(document.getElementById('image'), {
          aspectRatio: 0,
          viewMode: 0,
        });
      });
    };
    reader.readAsDataURL(file);
  }
};


const saveCroppedImage = () => {
  if (cropper.value) {
    const canvas = cropper.value.getCroppedCanvas();
    croppedImage.value = canvas.toDataURL('image/png');
  }
  if (croppedImage.value) {
    imageUploaded(croppedImage.value);
    setEditing(false);
  }
};

const setEditing = (state) => {
  editing.value = state;
};

const onRefresh = () => {
  localStorage.removeItem('cards');
  cards.value = [
    { image: "", id: 0 },
    { image: "", id: 1 },
    { image: "", id: 2 },
    { image: "", id: 3 },
    { image: "", id: 4 },
    { image: "", id: 5 },
    { image: "", id: 6 },
    { image: "", id: 7 },
    { image: "", id: 8 },
  ];
};

const uploadImage = async () => {
  if (!selectedFile.value) return;
  loading.value = true;
  const formData = new FormData();
  formData.append('file', selectedFile.value);

  try {
    const response = await axios.post('http://195.14.123.132:5000/upload', formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
    });
    const data = await response.data;
    uploadedFileName.url = data.filePath;
    recentImages.value.push(data.filePath);
    imageUploaded(uploadedFileName.url);
  } catch (error) {
    loading.value = false;
    console.error('Error uploading image:', error);
  }
};

const saveToLocalStorage = () => {
  localStorage.setItem('cards', JSON.stringify(cards.value));
  localStorage.setItem('recentImages', JSON.stringify(recentImages.value));
};

const rotateImage = (degrees) => {
  if (cropper.value) {
    cropper.value.rotate(degrees);
  }
};

const applyColorFilter = (filter) => {
  if (cropper.value) {
    cropper.value.style.filter = filter;
  }
};

onMounted(() => {
  if (localStorage.getItem('cards')) {
    cards.value = JSON.parse(localStorage.getItem('cards'));
  }
  if (localStorage.getItem('recentImages')) {
    recentImages.value = JSON.parse(localStorage.getItem('recentImages'));
  }
});

onUnmounted(() => {
  if (cropper.value) {
    cropper.value.destroy();
  }
});

watch([cards, recentImages], saveToLocalStorage, { deep: true });

const modalStyle = computed(() => ({
  position: 'absolute',
  top: '50%',
  left: '50%',
  transform: 'translate(-50%, -50%)',
  width: '70%',
  height: '80%',
  backgroundColor: 'white',
  border: '2px solid #000',
  boxShadow: 20,
  padding: '20px',
}));
</script>


<style lang="sass" scoped>
.home
  display: flex
  flex-direction: column
  .canvas
    border-radius: 10px
    .controls-dash
      display: flex
      background-color: #c4ae96
      justify-content: center
      align-items: center
      padding: 0
      .refresh
        color: green
        border: none
        text-align: center
        text-decoration: none
        font-size: 16px
      .preview
        color: red
        border: none
        text-align: center
        text-decoration: none
        font-size: 16px
    .grid-container
      display: grid
      grid-template-columns: repeat(auto-fill, minmax(300px, 1fr))
      gap: 16px
      background-color: #c4ae96
      padding: 15px
    .grid-item 
      display: inline-block
      .card
        max-width: 100%
        max-height: 240px
        height: 240px
        overflow: hidden
        box-shadow: 0 2px 5px rgba(0, 0, 0, 0.5)
        position: relative
        cursor: pointer
        border-radius: 10px
      
        .card-media img 
          width: 100%
          height: 240px
      
        .image-list-item-bar 
          position: absolute
          bottom: 0
          right: 0
          background: rgba(0, 0, 0, 0.3)
          padding: 4px

          .icon-button 
            background: none
            border: none
            color: rgba(255, 255, 255, 0.54)
            cursor: pointer
            font-size: 24px
    .spinner-overlay
      position: fixed
      top: 0
      left: 0
      width: 100%
      height: 100%
      background: rgba(255, 255, 255, 0.8)
      display: flex
      justify-content: center
      align-items: center
      z-index: 9999
          
      .spinner
        border: 16px solid #f3f3f3
        border-top: 16px solid #3498db
        border-radius: 50%
        width: 120px
        height: 120px
        animation: spin 2s linear infinite
      
      @keyframes spin
        0%
          transform: rotate(0deg)
        100%
          transform: rotate(360deg)

    .image-cropper 
      max-width: 800px
      #cropper-image
        max-width: 600px
        max-height: 200px
      .cropper-container
        margin-bottom: 20px
        .controls 
          display: block
</style>
