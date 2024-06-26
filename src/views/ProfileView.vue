<template>
  <div>
    <input type="file" @change="onFileChange" />
    <div v-if="imageUrl">
      <img ref="image" :src="imageUrl" alt="Source Image" />
    </div>
    <button @click="cropImage">Crop</button>
    <div v-if="croppedImage">
      <h3>Cropped Image:</h3>
      <img :src="croppedImage" alt="Cropped Image" />
    </div>
  </div>
</template>

<script>
import Cropper from 'cropperjs';
import 'cropperjs/dist/cropper.css';

export default {
  data() {
    return {
      cropper: null,
      imageUrl: null,
      croppedImage: null,
    };
  },
  methods: {
    onFileChange(e) {
      const file = e.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (event) => {
          this.imageUrl = event.target.result;
          console.log(this.imageUrl);
          this.$nextTick(() => {
            if (this.cropper) {
              this.cropper.destroy();
            }
            this.cropper = new Cropper(this.$refs.image, {
              aspectRatio: 1,
              viewMode: 1,
            });
          });
        };
        reader.readAsDataURL(file);
      }
    },
    cropImage() {
      if (this.cropper) {
        const canvas = this.cropper.getCroppedCanvas();
        this.croppedImage = canvas.toDataURL('image/png');
      }
    },
  },
  beforeDestroy() {
    if (this.cropper) {
      this.cropper.destroy();
    }
  },
};
</script>

<style scoped>
img {
  max-width: 100%;
}
</style>
