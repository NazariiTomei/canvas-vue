<template>
  <div class="button-wrapper">
    <span class="button" @click="$refs.file.click()">
      <input type="file" ref="file" @change="uploadImage($event)" accept="image/*">
      <slot>Upload image</slot>
    </span>
  </div>
</template>
<script>
export default {
  name: "ImageUploadButton",
  props: {
    value: {
      type: String
    }
  },
  data() {
    return {
      image: null
    };
  },
  methods: {
    uploadImage(event) {
      // Reference to the DOM input element
      var input = event.target;
      // Ensure that you have a file before attempting to read it
      if (input.files && input.files[0]) {
        // create a new FileReader to read this image and convert to base64 format
        var reader = new FileReader();
        // Define a callback function to run, when FileReader finishes its job
        reader.onload = e => {
          // Note: arrow function used here, so that "this.imageData" refers to the imageData of Vue component
          // Read image as base64 and set to imageData
          this.image = e.target.result;
          this.$emit("input", this.image);
        };
        // Start the reader job - read file as a data url (base64 format)
        reader.readAsDataURL(input.files[0]);
      }
    }
  }
};
</script>
<style scoped>
.button-wrapper {
  display: flex;
  justify-content: center;
  margin-top: 10px;
}

.button {
  color: #3fb37f;
  font-size: 16px;
  cursor: pointer;
}

.button:hover {
  font-weight: bold;
}

.button input {
  display: none;
}
</style>

