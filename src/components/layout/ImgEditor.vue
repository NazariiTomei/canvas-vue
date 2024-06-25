<template>
    <div>
        <ImageCropper :imageUrl="imageUrl" :onSave="handleSave" :onCancel="handleCancel" />
    </div>
</template>

<script>
import ImageCropper from '@/components/layout/ImageCropper.vue';

export default {
    components: {
        ImageCropper,
    },
    data() {
        return {
            imageUrl: '/1719108548512-360_F_831622859_OUuIZtXhI1ExyaXrIRugaX3ZFKdHFbu4.jpg',
        };
    },
    methods: {
        handleSave(croppedImage) {
            // Handle saving cropped image
            console.log('Cropped image:', croppedImage);
            // Here you can implement logic to save the cropped image, like sending it to a server or updating state
        },
        handleCancel() {
            // Handle cancel action
            console.log('Crop operation canceled');
        },
    },
};
</script>