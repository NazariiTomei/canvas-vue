<template>
  <div v-if="visible" @click.self="close">
    <div class="modal">
      <slot></slot>
    </div>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits } from 'vue';

const props = defineProps({
  visible: {
    type: Boolean,
    default: false,
  },
});

const emits = defineEmits(['update:visible']);

const close = () => {
  emits('update:visible', false);
};
</script>

<style scoped></style>