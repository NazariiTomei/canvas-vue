<template>
  <div class="card" @click="handleOpen(id)">
    <div class="card-media">
      <img :src="imageSrc === '' ? '/sample.jpeg' : imageSrc" alt="Image" />
    </div>
    <div v-if="imageSrc !== ''" class="image-list-item-bar">
      <button class="icon-button" @click.stop="handleEdit(imageSrc)">
        ℹ️
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ItemCard',
  props: {
    id: {
      type: [String, Number],
      required: true,
    },
    imageSrc: {
      type: String,
      default: '',
    },
    handleOpen: {
      type: Function,
      required: true,
    },
    handleEdit: {
      type: Function,
      required: true,
    },
  },
};
</script>

<style scoped>
.card {
  /* border: 1px solid #ddd; */
  max-width: 100%;
  max-height: 240px;
  height: 240px;
  overflow: hidden;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.5);
  position: relative;
  cursor: pointer;
  border-radius: 10px;
}

.card-media img {
  width: 100%;
  height: 240px;
}

.image-list-item-bar {
  position: absolute;
  bottom: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.3);
  padding: 4px;
}

.icon-button {
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.54);
  cursor: pointer;
  font-size: 24px;
}
</style>