<template>
    <div id="app">
        <ImageUploadButton v-model="image" />
        <button v-if="image" @click="openModal">Edit</button>
        <Modal v-show="showModal" @close="closeModal">
            <div slot="body" class="cropper-wrapper">
                <Cropper ref="cropper" class="cropper" :src="image" :defaultSize="cropperSize"
                    :defaultPosition="cropperPosition" />
            </div>
            <div slot="actions" class="actions">
                <button @click="crop">Crop</button>
            </div>
        </Modal>
        <!-- <pre>{{ image }}</pre> -->
        <img class="crop-preview" v-if="croppedImage" :src="croppedImage">
    </div>
</template>

<script>
import ImageUploadButton from "@/components/ui/ImageUpload.vue";
import Modal from "@/components/ui/Modal.vue";
import { Cropper } from "vue-advanced-cropper";

export default {
    name: "App",
    data() {
        return {
            image: null,
            croppedImage: null,
            coordinates: {
                width: null,
                height: null,
                left: null,
                top: null
            },
            showModal: false,
            zoom: 0.5
        };
    },
    watch: {
        image(src) {
            if (src) this.openModal();
        }
    },
    methods: {
        openModal() {
            this.showModal = true;
        },
        closeModal() {
            this.showModal = false;
        },
        crop() {
            const { coordinates, canvas } = this.$refs.cropper.getResult();
            this.coordinates = coordinates;
            // You able to do different manipulations at a canvas
            // but there we just get a cropped image
            this.croppedImage = canvas.toDataURL();
            this.closeModal();
        },
        cropperSize(cropper, image, props) {
            const { width, height } = this.coordinates;

            if (width !== null && height !== null) {
                return { width, height };
            }
            // TODO: move this code to function defaultSize function
            const maxWidth = (props.maxWidth / 100) * image.naturalWidth;
            const maxHeight = (props.maxHeight / 100) * image.naturalHeight;
            const minWidth = (props.minWidth / 100) * image.naturalWidth;
            const minHeight = (props.minHeight / 100) * image.naturalHeight;

            let newHeight, newWidth;
            if (maxHeight > maxWidth) {
                newHeight = Math.max(minHeight, maxHeight * 0.8);
                newWidth = Math.max(minWidth, maxWidth * 0.8);
            } else {
                newWidth = Math.max(minWidth, maxWidth * 0.8);
                newHeight = Math.max(minHeight, maxHeight * 0.8);
            }

            return {
                height: newHeight,
                width: newWidth
            };
        },
        cropperPosition(cropper, image, width, height, props) {
            const { left, top } = this.coordinates;

            if (left !== null && top !== null) {
                return { left, top };
            }
            // TODO: move this code below to defaultPosition function
            return {
                left: image.naturalWidth / 2 - width / 2,
                top: image.naturalHeight / 2 - height / 2
            };
        }
    },
    components: {
        ImageUploadButton,
        Modal,
        Cropper
    }
};
</script>
<style>
#app {
    font-family: "Avenir", Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}

.cropper {
    height: 300px;
    width: 100%;
}

/* Trash code */
.crop-preview {
    height: 200px;
}
</style>