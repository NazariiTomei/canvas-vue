<template>
  <div class="home">
    <div class="canvas">
      <div v-if="loading" class="spinner-overlay">
        <div class="spinner"></div>
      </div>
      <div>
        <div v-if="editing" class="editing">
          <div class="image-cropper">
            <div class="cropper-container">
              <div class="actions">
                <div class="rotate-action">
                  <img src="/rotate.png" style="width: 50px;" @click="rotateImage(90)" />
                </div>
                <div class="colorfilter-action">
                  <select class="colorFilterComboBox" v-model="colorfilter" @change="applyColorFilter"
                    style="width: 200px;height: 100px; ">
                    <option value="" style="font-size: 20px;">Original</option>
                    <option value="grayscale(100%)" style="font-size: 20px;">Grayscale</option>
                    <option value="sepia(100%)" style="font-size: 20px;">Sepia</option>
                    <option value="blur(5px)">Blur</option>
                    <option value="brightness(150%)">Brightness</option>
                    <option value="contrast(200%)">Contrast</option>
                    <option value="hue-rotate(90deg)">Hue Rotate</option>
                    <option value="invert(100%)">Invert</option>
                    <option value="saturate(200%)">Saturate</option>
                  </select>
                </div>
              </div>
              <div v-if="cropperImage" class="cropimage">
                <img ref="image" id="cropper-image" :src="cropperImage" alt="cropper-image" />
              </div>
              <div class="controls">
                <div>
                  <img src="/rotate.png" style="width: 50px;" @click="rotateImage(90)" />
                </div>
                <div>
                  <button class="done" @click="saveCroppedImage">Done</button>
                  <button class="cancel" @click="setEditing(false)">Cancel</button>
                </div>
              </div>
            </div>

          </div>
        </div>
        <div v-else class="grid-container">
          <div v-for="(card, index) in cards" :key="card.id" :class="{ 'grid-item': true }">
            <template class="ItemCard" v-if="index !== 4">
              <div class="card" @click="handleOpen(card.id)">
                <div class="card-media">
                  <img :src="card.image === '' ? '/sample.jpeg' : card.image" alt="Card image" />
                </div>
                <div v-if="card.image !== ''" class="image-list-item-bar">
                  <button class="icon-button" @click.stop="handleEdit(card.image)">ℹ️</button>
                </div>
              </div>
            </template>
            <template class="ItemCard" v-else>
              <div class="card-center">
                <div class="card-media-center">
                  <img :src="card.image === '' ? '/love1.png' : card.image" alt="love png" />
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
        <div :style="modalStyle" class="modal-content">
          <img src="/close-icon.png" @click="handleClose" class="close-icon" />
          <h1 class="select-file-content">Select Image</h1>
          <div class="upload-file">
            <input type="file" @change="onFileChange" accept="image/*" />
            <button class="btn-blue" @click="uploadImage" :disabled="!selectedFile">Upload</button>
            <div v-if="imageUrl">
              <img :src="imageUrl" class="preview-image" alt="Image Preview" />
            </div>
            <div v-if="recentImages.length">
              <h2 class="select-file-content2">Or Choose from your recent images</h2>
              <div class="recent-grid-container">
                <div v-for="(recentimage, index) in recentImages" :key="index" :class="{ 'grid-item': true }">
                  <img class="recent-image" @click="imageUploaded(recentimage)" :src="recentimage"
                    alt="Image Preview" />
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
const colorfilter = ref('');
const filterStyles = {
  '': '',
  'grayscale(100%)': 'grayscale(100%)',
  'sepia(100%)': 'sepia(100%)',
  'blur(5px)': 'blur(5px)',
  'brightness(150%)': 'brightness(150%)',
  'contrast(200%)': 'contrast(200%)',
  'hue-rotate(90deg)': 'hue-rotate(90deg)',
  'invert(100%)': 'invert(100%)',
  'saturate(200%)': 'saturate(200%)',
};
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

const applyColorFilter = () => {
  if (cropper.value && cropper.value.canvas && filterStyles[colorfilter.value]) {
    cropper.value.canvas.style.filter = filterStyles[colorfilter.value];
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
  height: '90%',
  backgroundColor: 'white',
  borderRadius: '20px',
  border: '2px solid #000',
  boxShadow: ' 20 20px 15px rgba(0, 0, 0, 0.5)',
  padding: '20px',
}));
</script>


<style lang="sass" scoped>
.home
  display: flex
  flex-direction: column
  backdrop-filter: blur(8px)
  .canvas
    border-radius: 10px
    .modal
      position: fixed
      top: 0
      left: 0
      width: 100%
      height: 100%
      background-color: rgba(0, 0, 0, 0.5)
      display: flex
      justify-content: center
      align-items: center
      z-index: 100
      backdrop-filter: blur(8px)

    .modal-content 
      position: relative
      background-color: white
      padding: 20px
      border-radius: 20px
      box-shadow: 0 20px 15px rgba(0, 0, 0, 0.5)
      width: 70%
      max-width: 800px
      height: 80%
      max-height: 740px
      overflow-y: hidden
      .select-file-content
        color: black
        font-weight: bold
      .upload-file
        .preview-image
          width: 240px
          height: 200px
          border-radius:20px
        .select-file-content2
          color: black
        .recent-grid-container
          height: 280px
          max-height: 280px
          overflow-y: scroll
          .recent-image
            width: 220px
            height: 200px
            padding: 10px
            border-radius: 20px
    .close-icon 
      position: absolute
      top: 10px
      right: 10px
      cursor: pointer
    
    h1 
      color: black
      font-weight: bold
    
    .upload-file 
      margin-top: 20px
    
    .btn-blue 
      background-color: #007bff
      color: white
      border: none
      padding: 10px 20px
      cursor: pointer
      border-radius: 4px
    
    .preview-image 
      width: 100%
      max-width: 100%
      height: auto
      border-radius: 10px
    
    .recent-images 
      margin-top: 20px
      display: flex
      flex-wrap: wrap
    
    .recent-image 
      margin-right: 10px
      margin-bottom: 10px

    .recent-image img 
      width: 100px
      height: auto
      cursor: pointer
      border-radius: 10px
    @media (max-width: 768px) 
      .modal-content 
        width: 90%
        height: 90%
        max-width: unset
        max-height: unset
    .controls-dash
      display: flex
      background-color: white
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
      background-color: white
      padding: 15px
    .grid-item 
      display: inline-block
      .card-center
        max-width: 100%
        max-height: 240px
        height: 240px
        overflow: hidden
        position: relative
        border-radius: 10px
      
        .card-media-center img 
          width: 100%
          height: 240px
      .card
        max-width: 100%
        max-height: 240px
        height: 240px
        overflow: hidden
        box-shadow: 5px 8px 5px rgba(0, 0, 0, 0.7)
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
      display: flex
      justify-content: center
      align-items: center
      z-index: 99999
      backdrop-filter: blur(8px) 
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
    .editing
      display: flex
      justify-content: center
      align-items: center
      .image-cropper 
        max-width: 800px
        .cropper-container
          max-width: 100%
          .cropimage
            max-width: 100%
            #cropper-image
              max-height: 600px
              max-width: 800px
              height: 500px
              width: 800px
          .controls 
            .rotate
              background-color: #04AA6D
              border: none
              color: black
              padding: 15px 32px
              text-align: center
              text-decoration: none
              display: inline-block
              font-size: 24px
              margin: 4px 2px
              cursor: pointer
              width: 150px

            
            .done
              background-color: #04AA6D
              border: none
              color: black
              padding: 15px 32px
              text-align: center
              text-decoration: none
              display: inline-block
              font-size: 24px
              margin: 4px 2px
              cursor: pointer
              width: 150px
            .cancel
              background-color: white
              border: none
              color: black
              padding: 15px 32px
              text-align: center
              text-decoration: none
              display: inline-block
              font-size: 24px
              margin: 4px 2px
              cursor: pointer
              width: 150px
</style>
