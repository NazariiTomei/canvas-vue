<template>
    <div class="image-cropper">
        <h1>Image Cropper</h1>
        <div class="cropper-container">
            <Cropper ref="cropper" :src="imageUrl" :options="cropperOptions" @change="handleCropChange" />
        </div>
        <div class="controls">
            <button @click="rotateLeft">Rotate Left</button>
            <button @click="rotateRight">Rotate Right</button>
            <button @click="zoomIn">Zoom In</button>
            <button @click="zoomOut">Zoom Out</button>
            <button @click="reset">Reset</button>
            <button @click="applyFilter('grayscale')">Grayscale</button>
            <button @click="applyFilter('sepia')">Sepia</button>
            <button @click="applyFilter('invert')">Invert</button>
            <button @click="applyFilter('brightness')">Brightness</button>
            <button @click="save">Save</button>
            <button @click="cancel">Cancel</button>
        </div>
    </div>
</template>

<script>
import { defineComponent, ref } from 'vue';
import { Cropper } from 'vue-advanced-cropper';
import 'vue-advanced-cropper/dist/style.css';

export default defineComponent({
    name: 'ImageCropper',
    components: {
        Cropper,
    },
    props: {
        imageUrl: {
            type: String,
            required: true,
        },
        onSave: {
            type: Function,
            required: true,
        },
        onCancel: {
            type: Function,
            required: true,
        },
    },
    setup(props) {
        const cropperRef = ref(null);
        const filter = ref('');

        const cropperOptions = {
            aspectRatio: null,
            viewMode: 1,
        };

        const handleCropChange = () => {
            const croppedCanvas = cropperRef.value.getCroppedCanvas();
            if (croppedCanvas) {
                applyCanvasFilter(croppedCanvas, filter.value);
            }
        };

        const applyCanvasFilter = (canvas, filter) => {
            const ctx = canvas.getContext('2d');
            const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
            const data = imageData.data;

            switch (filter) {
                case 'grayscale':
                    for (let i = 0; i < data.length; i += 4) {
                        const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
                        data[i] = data[i + 1] = data[i + 2] = avg;
                    }
                    break;
                case 'sepia':
                    for (let i = 0; i < data.length; i += 4) {
                        const r = data[i];
                        const g = data[i + 1];
                        const b = data[i + 2];
                        data[i] = r * 0.393 + g * 0.769 + b * 0.189;
                        data[i + 1] = r * 0.349 + g * 0.686 + b * 0.168;
                        data[i + 2] = r * 0.272 + g * 0.534 + b * 0.131;
                    }
                    break;
                case 'invert':
                    for (let i = 0; i < data.length; i += 4) {
                        data[i] = 255 - data[i];
                        data[i + 1] = 255 - data[i + 1];
                        data[i + 2] = 255 - data[i + 2];
                    }
                    break;
                case 'brightness':
                    for (let i = 0; i < data.length; i += 4) {
                        data[i] *= 1.2;
                        data[i + 1] *= 1.2;
                        data[i + 2] *= 1.2;
                    }
                    break;
                default:
                    break;
            }
            ctx.putImageData(imageData, 0, 0);
        };

        const applyFilter = (selectedFilter) => {
            filter.value = selectedFilter;
            handleCropChange();
        };

        const rotateLeft = () => {
            cropperRef.value.rotateTo(cropperRef.value.getData().rotate - 90);
            handleCropChange();
        };

        const rotateRight = () => {
            cropperRef.value.rotateTo(cropperRef.value.getData().rotate + 90);
            handleCropChange();
        };

        const zoomIn = () => {
            cropperRef.value.zoomTo(cropperRef.value.getData().scaleX + 0.1);
            handleCropChange();
        };

        const zoomOut = () => {
            cropperRef.value.zoomTo(cropperRef.value.getData().scaleX - 0.1);
            handleCropChange();
        };

        const reset = () => {
            cropperRef.value.reset();
            filter.value = '';
            handleCropChange();
        };

        const save = () => {
            const croppedCanvas = cropperRef.value.getCroppedCanvas();
            if (croppedCanvas) {
                const croppedImage = croppedCanvas.toDataURL('image/jpeg');
                props.onSave(croppedImage);
            }
        };

        const cancel = () => {
            props.onCancel();
        };

        return {
            cropperRef,
            cropperOptions,
            handleCropChange,
            applyFilter,
            rotateLeft,
            rotateRight,
            zoomIn,
            zoomOut,
            reset,
            save,
            cancel,
        };
    },
});
</script>

<style scoped>
.image-cropper {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
}

.cropper-container {
    margin-bottom: 20px;
}

.controls {
    display: flex;
    justify-content: space-between;
    flex-wrap: wrap;
}

.controls button {
    margin: 5px;
}
</style>