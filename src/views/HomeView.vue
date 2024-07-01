<template>
  <div class="home">
    <div v-if="loading" class="spinner-overlay">
      <div class="spinner"></div>
      <p calss="bar" style="position: absolute; font-size: 150%; font-weight: bold; color: red">
        {{ progress }} %
      </p>
    </div>
    <div class="canvas">
      <div v-if="editing" class="editing">
        <div class="container">
          <div class="row">
            <div class="col-sm-2">
              <button @click="rotateImage(90)" class="rotate-btn">
                <i class="fa fa-rotate-right"></i>
              </button>
            </div>
            <div class="col-sm-7"></div>
            <div class="col-sm-3 form-group" style="width: 25%">
              <select class="form-control" v-model="colorfilter" id="sel1">
                <option value="">Original</option>
                <option value="grayscale(100%)">Grayscale</option>
                <option value="sepia(100%)">Sepia</option>
                <option value="blur(5px)">Blur</option>
                <option value="brightness(150%)">Brightness</option>
                <option value="contrast(200%)">Contrast</option>
                <option value="hue-rotate(90deg)">Hue Rotate</option>
                <option value="invert(100%)">Invert</option>
                <option value="saturate(200%)">Saturate</option>
              </select>
            </div>
          </div>
          <div class="row">
            <cropper
              ref="cropperRef"
              class="cropper"
              :aspect-ratio="16 / 9"
              background-classname="cropper-background"
              style="height: 500px; width: 100%"
              :src="editImage"
              :stencil-props="{
                aspectRatio: 933 / 633,
                movable: true,
                resizable: true
              }"
              :defaultPosition="cropperPosition"
              :style="{ filter: colorfilter }"
            />
          </div>
          <!-- <div class="row p-3">
            <div class="col-sm-6">
              <button @click="$refs.zoomer.zoomIn()">-</button>
            </div>
            <div class="col-sm-6">
              <button>+</button>
            </div>
          </div> -->
          <div class="row pt-3">
            <div class="col-sm-6">
              <button style="float: right" class="btn btn-danger" @click="saveCroppedImage">
                Done
              </button>
            </div>
            <div class="col-sm-6">
              <button class="btn btn-secondary" @click="setEditing(false)">Cancel</button>
            </div>
          </div>
        </div>
      </div>
      <div v-else class="card-board">
        <div
          v-for="(card, index) in cards"
          :key="card.id"
          :style="{
            position: 'absolute',
            top: card.coordinate.y / 21 + '%',
            left: card.coordinate.x / 30 + '%',
            width: card.coordinate.w / 30 + '%',
            height: card.coordinate.h / 21 + '%'
          }"
        >
          <div v-if="card.id !== 4" @click="handleOpen(card.id)">
            <div class="card">
              <img
                v-if="card.croppedImage !== null"
                :src="card.croppedImage"
                :style="{ objectFit: 'cover', width: '100%', height: '100%' }"
              />
              <div
                v-else
                :style="{
                  backgroundColor: '#666666',
                  width: '100%',
                  height: '100%',
                  display: 'flex',
                  alignItems: 'center',
                  justifyContent: 'center'
                }"
              >
                <img :src="'/image-plus.svg'" alt="plus image" width="20%" />
              </div>
              <div v-if="card.image !== ''">
                <button class="custom-button" @click.stop="handleEdit(card)">
                  <i class="fa fa-edit"></i>
                </button>
              </div>
            </div>
          </div>
          <div v-else>
            <img
              :src="card.croppedImage === null ? '/love1.png' : card.croppedImage"
              alt="love png"
              width="100%"
            />
          </div>
        </div>
      </div>
      <div v-if="!editing" class="controls-dash">
        <a @click="onRefresh" class="refresh">Refresh</a>
        <a @click="onPreview" class="preview">Preview</a>
      </div>
      <div v-if="openDialog" @click.self="closeModal" class="custom-modal">
        <div class="modal-content">
          <img src="/close-icon.png" @click="handleClose" class="close-icon" />
          <h1 class="select-file-content">Select Image</h1>
          <div class="upload-file">
            <input type="file" @change="onFileChange" accept="image/*" />
            <button class="btn btn-success" @click="uploadImage" :disabled="!selectedFile">
              Upload
            </button>
            <div v-if="imageUrl">
              <img :src="imageUrl" class="preview-image" alt="Image Preview" />
            </div>
            <div v-if="recentImages.length">
              <h2 class="select-file-content2">Or Choose from your recent images</h2>
              <div class="recent-grid-container">
                <div
                  v-for="(recentimage, index) in recentImages"
                  :key="index"
                  :class="{ 'grid-item': true }"
                >
                  <img
                    class="recent-image"
                    @click="imageUploaded(recentimage)"
                    :src="recentimage"
                    alt="Image Preview"
                  />
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
import { ref, onMounted, watch } from 'vue'
import axios from 'axios'
import { Cropper } from 'vue-advanced-cropper'
import 'vue-advanced-cropper/dist/style.css'
import 'font-awesome/css/font-awesome.min.css'
import 'bootstrap/dist/css/bootstrap.css'
import VueZoomer from 'vue-zoomer'
const selectedFile = ref(null)
const openDialog = ref(false)
const selectId = ref(0)
const recentImages = ref(JSON.parse(localStorage.getItem('recentImages')) || [])
const imageUrl = ref('')
const loading = ref(false)
const progress = ref(0)
const editing = ref(false)
const editImage = ref({})
const colorfilter = ref('')
const cropperRef = ref(null)
const filterStyles = {
  '': '',
  'grayscale(100%)': 'grayscale(100%)',
  'sepia(100%)': 'sepia(100%)',
  'blur(5px)': 'blur(5px)',
  'brightness(150%)': 'brightness(150%)',
  'contrast(200%)': 'contrast(200%)',
  'hue-rotate(90deg)': 'hue-rotate(90deg)',
  'invert(100%)': 'invert(100%)',
  'saturate(200%)': 'saturate(200%)'
}
const cards = ref(
  JSON.parse(localStorage.getItem('cards')) || [
    {
      image: '',
      id: 0,
      rotate: 0,
      coordinate: { x: 50, y: 50, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 1,
      rotate: 0,
      coordinate: { x: 1034, y: 50, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 2,
      rotate: 0,
      coordinate: { x: 2018, y: 50, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 3,
      rotate: 0,
      coordinate: { x: 50, y: 734, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 4,
      rotate: 0,
      coordinate: { x: 1034, y: 734, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 5,
      rotate: 0,
      coordinate: { x: 2018, y: 734, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 6,
      rotate: 0,
      coordinate: { x: 50, y: 1418, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 7,
      rotate: 0,
      coordinate: { x: 1034, y: 1418, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 8,
      rotate: 0,
      coordinate: { x: 2018, y: 1418, w: 933, h: 33 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    }
  ]
)

const cropperPosition = (cropper, image, width, height, props) => {
  const { left, top } = editImage.coordinates
  if (left !== null && top !== null) {
    return { left, top }
  }
  return { left: 100, top: 100 }
}

const handleOpen = (id) => {
  selectId.value = id
  openDialog.value = true
}

const handleEdit = (card) => {
  editImage.value = card.image
  editImage.coordinates = card.coordinates
  editImage.rotate = card.rotate
  colorfilter.value = card.colorfilter
  selectId.value = card.id
  editing.value = true
}

const handleClose = () => {
  openDialog.value = false
}

const onPreview = () => {
  //Implement your preview functionality here
}

const imageUploaded = (filePath) => {
  selectedFile.value = filePath
  cards.value.forEach((item) => {
    if (item.id === selectId.value) {
      item.image = filePath
      item.croppedImage = filePath
      if (filePath.coordinates) {
        item.coordinates = filePath.coordinates
      }
    }
  })
  loading.value = false
  handleClose()
  saveToLocalStorage()
}

const onFileChange = (event) => {
  const file = event.target.files[0]
  if (file) {
    selectedFile.value = file
    const reader = new FileReader()
    reader.onload = (e) => {
      imageUrl.value = e.target.result
    }
    reader.readAsDataURL(file)
  }
}

const rotateImage = (degrees) => {
  editImage.rotate++
  editImage.rotate = editImage.rotate % 4
  cropperRef.value.rotate(degrees)
}

const saveCroppedImage = () => {
  const cropperInstance = cropperRef.value
  if (cropperInstance) {
    const { coordinates, canvas } = cropperInstance.getResult()
    const tempCanvas = document.createElement('canvas')
    tempCanvas.width = canvas.width
    tempCanvas.height = canvas.height
    const tempContext = tempCanvas.getContext('2d')

    // Apply selected color filter
    if (colorfilter.value && filterStyles[colorfilter.value]) {
      tempContext.filter = colorfilter.value
      tempContext.drawImage(canvas, 0, 0, canvas.width, canvas.height)
    } else {
      tempContext.drawImage(canvas, 0, 0)
    }
    // Get the data URL of the edited image
    const imgURL = tempCanvas.toDataURL('image/png')
    cards.value.forEach((item) => {
      if (item.id === selectId.value) {
        item.croppedImage = imgURL
        if (coordinates) {
          item.coordinates = coordinates
          item.colorfilter = colorfilter.value
        }
      }
    })
    setEditing(false) // Exit editing mode
  }
}

const setEditing = (state) => {
  editing.value = state
}

const onRefresh = () => {
  localStorage.removeItem('cards')
  cards.value = [
    {
      image: '',
      id: 0,
      rotate: 0,
      coordinate: { x: 50, y: 50, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 1,
      rotate: 0,
      coordinate: { x: 1034, y: 50, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 2,
      rotate: 0,
      coordinate: { x: 2018, y: 50, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 3,
      rotate: 0,
      coordinate: { x: 50, y: 734, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 4,
      rotate: 0,
      coordinate: { x: 1034, y: 734, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 5,
      rotate: 0,
      coordinate: { x: 2018, y: 734, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 6,
      rotate: 0,
      coordinate: { x: 50, y: 1418, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 7,
      rotate: 0,
      coordinate: { x: 1034, y: 1418, w: 933, h: 633 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    },
    {
      image: '',
      id: 8,
      rotate: 0,
      coordinate: { x: 2018, y: 1418, w: 933, h: 33 },
      colorfilter: '',
      croppedImage: null,
      coordinates: { width: null, height: null, left: null, top: null },
      zoom: 0.5
    }
  ]
}

const saveToLocalStorage = async () => {
  const cardsCopy = await JSON.stringify(cards.value)
  localStorage.setItem('cards', cardsCopy)
  localStorage.setItem('recentImages', JSON.stringify(recentImages.value))
}

const uploadImage = async () => {
  if (!selectedFile.value) {
    return
  }

  loading.value = true

  try {
    const formData = new FormData()
    formData.append('file', selectedFile.value)
    const response = await axios.post('http://195.14.123.132:5000/upload', formData, {
      headers: {
        'Content-Type': 'multipart/form-data'
      },
      onUploadProgress: (progressEvent) => {
        progress.value = Math.round((progressEvent.loaded * 100) / progressEvent.total)
      }
    })

    const filePath = response.data.filePath
    imageUploaded(filePath)
    recentImages.value.push(filePath)
    saveToLocalStorage()
  } catch (error) {
  } finally {
    loading.value = false
  }
}

watch(cards, saveToLocalStorage, { deep: true })

onMounted(() => {
  const savedCards = localStorage.getItem('cards')
  if (savedCards) {
    cards.value = JSON.parse(savedCards)
  }
})
</script>

<style lang="sass" scoped>
.home
  display: flex
  flex-direction: column
  backdrop-filter: blur(8px)
  .canvas
    border-radius: 10px
    .editing
      display: flex
      justify-content: center
      align-items: center
      .container
        width: 80%
        .row
          width: 100%
          .cropper
            background: #DDD
    .custom-modal
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
    .card-board
      max-width: 1000px
      min-width: 1000px
      max-height: 700px
      min-height: 700px
      background-color: white
      display: inline-flex
      position: relative
      .card
        float: center
        overflow: hidden
        box-shadow: -4px 7px 3px rgba(0, 0, 0, 0.7)
        cursor: pointer
        position: relative
        border-radius: 10px
        height:211px
        .custom-button
          position: absolute
          bottom: 0
          right: 0
          background: rgba(0, 0, 0, 0.5)
          padding: 5px
          border: none
          border-radius:10%
          color: rgba(255, 255, 255, 0.7)
          cursor: pointer
        @media (max-width: 600px)
          .custom-button
            position: absolute
            bottom: 0
            right: 0
            background: rgba(0, 0, 0, 0.5)
            padding: 5px
            border: none
            border-radius:10%
            color: rgba(255, 255, 255, 0.7)
            cursor: pointer
            font-size: 14px
        @media (max-width: 478px)
          .custom-button
            position: absolute
            bottom: 0
            right: 0
            background: rgba(0, 0, 0, 0.5)
            padding: 5px
            border: none
            border-radius:10%
            color: rgba(255, 255, 255, 0.7)
            cursor: pointer
            font-size: 10px
        .card-img
          width: 100%
          background-color:#ccc
    @media (max-width: 1200px)
      .card-board
        max-width: 900px
        min-width: 900px
        max-height: 630px
        min-height: 630px
        background-color: white
        display: inline-flex
        position: relative
        .card
          overflow: hidden
          box-shadow: -4px 7px 3px rgba(0, 0, 0, 0.7)
          cursor: pointer
          border: none
          position: relative
          border-radius: 10px
          height:189.9px
    @media (max-width: 1000px)
      .card-board
        max-width: 800px
        min-width: 800px
        max-height: 560px
        min-height: 560px
        background-color: white
        display: inline-flex
        position: relative
        .card
          border: none
          overflow: hidden
          box-shadow: -4px 7px 3px rgba(0, 0, 0, 0.7)
          cursor: pointer
          position: relative
          border-radius: 10px
          height:168.8px
    @media (max-width: 900px)
      .card-board
        max-width: 750px
        min-width: 750px
        max-height: 525px
        min-height: 525px
        background-color: white
        display: inline-flex
        position: relative
        .card
          overflow: hidden
          box-shadow: -4px 7px 3px rgba(0, 0, 0, 0.7)
          cursor: pointer
          position: relative
          border-radius: 10px
          height:158.25px
    @media (max-width: 768px)
      .card-board
        max-width: 600px
        min-width: 600px
        max-height: 420px
        min-height: 420px
        background-color: white
        display: inline-flex
        position: relative
        .card
          overflow: hidden
          box-shadow: -4px 7px 3px rgba(0, 0, 0, 0.7)
          cursor: pointer
          position: relative
          border-radius: 10px
          height:126.6px
    @media (max-width: 660px)
      .card-board
        max-width: 500px
        min-width: 500px
        max-height: 350px
        min-height: 350px
        background-color: white
        display: inline-flex
        position: relative
        .card
          overflow: hidden
          box-shadow: -4px 7px 3px rgba(0, 0, 0, 0.7)
          cursor: pointer
          position: relative
          border-radius: 10px
          height:105.5px
    @media (max-width: 550px)
      .card-board
        max-width: 450px
        min-width: 450px
        max-height: 315px
        min-height: 315px
        background-color: white
        display: inline-flex
        position: relative
        .card
          overflow: hidden
          box-shadow: -4px 7px 3px rgba(0, 0, 0, 0.7)
          cursor: pointer
          position: relative
          border-radius: 10px
          height:94.95px
    @media (max-width: 500px)
      .card-board
        max-width: 300px
        min-width: 300px
        max-height: 210px
        min-height: 210px
        background-color: white
        display: inline-flex
        position: relative
        .card
          overflow: hidden
          box-shadow: -4px 7px 3px rgba(0, 0, 0, 0.7)
          cursor: pointer
          position: relative
          border-radius: 10px
          height:63.3px
    @media (max-width: 350px)
      .card-board
        max-width: 200px
        min-width: 200px
        max-height: 140px
        min-height: 140px
        background-color: white
        display: inline-flex
        position: relative
        .card
          overflow: hidden
          box-shadow: -4px 7px 3px rgba(0, 0, 0, 0.7)
          cursor: pointer
          position: relative
          border-radius: 10px
          height:42.2px
    @media (max-width: 250px)
      .card-board
        max-width: 180px
        min-width: 180px
        max-height: 126px
        min-height: 126px
        background-color: white
        display: inline-flex
        position: relative
        .card
          overflow: hidden
          box-shadow: -4px 7px 3px rgba(0, 0, 0, 0.7)
          cursor: pointer
          position: relative
          border-radius: 10px
          height:37.98px
    @media (max-width: 200px)
      .card-board
        max-width: 150px
        min-width: 150px
        max-height: 105px
        min-height: 105px
        background-color: white
        display: inline-flex
        position: relative
        .card
          overflow: hidden
          box-shadow: -4px 7px 3px rgba(0, 0, 0, 0.7)
          cursor: pointer
          position: relative
          border-radius: 10px
          height:36.65px
    .modal-content
      position: relative
      background-color: white
      padding: 20px
      border-radius: 20px
      box-shadow: 0 20px 15px rgba(0, 0, 0, 0.5)
      width: 800px
      max-width: 800px
      height: 100%
      max-height: 740px
      overflow-x: hidden
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
          display: flex
          height: 220px
          gap:16px
          max-height: 220px
          overflow-y: scroll
          .recent-image

            width: 210px
            height: 168px
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


    .preview-image
      width: 100%
      max-width: 100%
      height: auto
      border-radius: 10px

    .recent-images
      margin-top: 20px
      display: grid

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
        width: 100%
        max-height: 100%
    .controls-dash
      min-width: 100%
      max-width: 100%
      display: flex
      justify-content: center
      align-items: center
      padding: 0
      .refresh
        cursor: pointer
        color: green
        border: none
        text-align: center
        text-decoration: none
        font-size: 24px
        padding: 10px
      .preview
        cursor: pointer
        color: red
        border: none
        text-align: center
        text-decoration: none
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
</style>
