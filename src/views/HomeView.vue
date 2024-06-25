<template>
  <div class="home">
    <div class="canvas">
      <Spinner v-if="loading" />
      <template v-else>
        <ImgEditor v-if="editing" :ImgSrc="'/saplme.jpeg'" :setEditing="setEditing" />
        <div v-else class="grid-container">
          <div v-for="(card, index) in cards" :key="card.id"
            :class="{ 'grid-item': true, 'display-none': index === 4 }">
            <template v-if="index !== 4">
              <div class="card" @click="handleOpen(card.id)">
                <div class="card-media">
                  <img :src="card.image === '' ? '/sample.jpeg' : card.image" alt="Image" />
                </div>
                <div v-if="card.image !== ''" class="image-list-item-bar">
                  <button class="icon-button" @click.stop="handleEdit(card.image)">
                    ℹ️
                  </button>
                </div>
              </div>
            </template>
          </div>
        </div>
      </template>
      <Modal :visible="openDialog" @update:visible="openDialog = $event">
        <div :style="modalStyle">
          <button @click="handleClose" style="float: right;" variant="contained" color="success">Close</button>
          <h1 style="color: black;">Select Image</h1>
          <div class="item">
            <div>
              <ImageUpload :uploadedFileName="uploadedFileName" :handleClose :imageUploaded />
            </div>
          </div>
        </div>
      </Modal>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import Spinner from '@/components/layout/Spinner.vue';
import ImgEditor from '@/components/layout/ImgEditor.vue';
import Modal from '@/components/ui/Modal.vue';
import ImageUpload from '@/components/ui/ImageUpload.vue';

const selectedFile = ref(null);
const openDialog = ref(false);
const selectId = ref(0);
const uploadStatus = ref('');
const uploadedFileName = ref({});
const loading = ref(false);
const editing = ref(false);
const editImage = ref('');
const cards = ref([
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
  editing.value = true;
};

const handleClose = () => {
  openDialog.value = false;
};

const imageUploaded = (filePath) => {
  uploadStatus.value = 'Upload successful';
  cards.value.forEach((item) => {
    if (item.id === selectId.value) {
      item.image = filePath.url;
    }
  });
  console.log(cards);
  handleClose();
};

const modalStyle = computed(() => ({
  position: 'absolute',
  top: '50%',
  left: '50%',
  transform: 'translate(-50%, -50%)',
  width: '840px',
  height: '800px',
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
    </style>
