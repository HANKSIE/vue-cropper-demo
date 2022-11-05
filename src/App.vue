<template>
  <div class="d-flex justify-content-center">
    <cropper-modal-component @crop="crop">
      <template v-if="src">
        <img class="avatar" :src="src" />
      </template>
      <template v-else>
        <div
          class="d-flex justify-content-center align-items-center bg-secondary avatar"
        >
          <span class="text-white">點擊上傳圖片</span>
        </div>
      </template>
    </cropper-modal-component>
  </div>
</template>
<style scoped>
.avatar {
  width: 200px;
  height: 200px;
  border-radius: 50%;
}
</style>
<script>
import CropperModalComponent from "./components/CropperModalComponent.vue";
export default {
  components: {
    CropperModalComponent,
  },
  data() {
    return {
      src: null,
      imageBlob: null,
    };
  },
  methods: {
    crop(blob) {
      if (this.imageBlob) URL.revokeObjectURL(this.imageBlob);
      this.imageBlob = blob;
      this.src = URL.createObjectURL(this.imageBlob);
    },
  },
};
</script>
