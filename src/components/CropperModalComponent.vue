<template>
  <div>
    <div data-bs-toggle="modal" data-bs-target="#cropperModal" role="button">
      <slot></slot>
    </div>
    <div class="modal fade" id="cropperModal" tabindex="-1" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <div class="row justify-content-center">
              <template v-if="src">
                <div
                  class="col-6 mb-3 mb-sm-0 d-flex justify-content-center align-items-center"
                >
                  <cropper
                    ref="cropper"
                    class="cropper"
                    :src="src"
                    :stencil-component="$options.components.CircleStencil"
                    :debounce="false"
                    :stencil-props="{
                      aspectRatio: 1,
                    }"
                    @change="updatePreview"
                  />
                </div>
                <div
                  class="col d-flex justify-content-center align-items-center"
                >
                  <preview
                    class="preview__circle"
                    :width="width"
                    :height="height"
                    :image="preview.image"
                    :coordinates="preview.coordinates"
                  />
                </div>
              </template>
              <template v-else>
                <h6 class="text-center">沒有圖片</h6>
                <template v-if="errorMessage">
                  <div class="text-center text-danger mt-2">
                    {{ errorMessage }}
                  </div>
                </template>
              </template>
            </div>
          </div>
          <div class="modal-footer">
            <button
              type="button"
              class="btn btn-outline-secondary"
              @click="clear"
            >
              清空
            </button>
            <button
              type="button"
              class="btn btn-outline-primary"
              @click="loadImage"
            >
              上傳圖片
            </button>
            <template v-if="src">
              <button
                type="button"
                data-bs-dismiss="modal"
                class="btn btn-warning"
                @click="crop"
              >
                裁切圖片
              </button>
            </template>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped>
.preview__circle {
  border-radius: 50%;
}
</style>
<script>
import { CircleStencil, Preview, Cropper } from "vue-advanced-cropper";
import "vue-advanced-cropper/dist/style.css";

const createPreviewState = () => ({
  coordinates: null,
  image: null,
});
export default {
  props: {
    width: {
      // preview width
      type: Number,
      default() {
        return 200;
      },
    },
    height: {
      // preview height
      type: Number,
      default() {
        return 200;
      },
    },
    initSrc: {
      // init src
      type: String,
      default() {
        return null;
      },
    },
  },
  components: {
    Cropper,
    Preview,
    // eslint-disable-next-line vue/no-unused-components
    CircleStencil,
  },
  data() {
    return {
      src: this.initSrc,
      cropSrc: null,
      mimeType: null,
      errorMessage: null,
      preview: createPreviewState(),
    };
  },
  methods: {
    crop() {
      const { canvas } = this.$refs.cropper.getResult();
      if (!canvas) return;
      canvas.toBlob((blob) => {
        this.$emit("crop", blob);
      }, this.mimeType);
    },
    // upload image from local
    loadImage() {
      const tempInput = document.createElement("input");
      tempInput.type = "file";
      tempInput.onchange = (e) => {
        const { files } = e.target;
        if (!files || files.length == 0) return;

        this.attemptRevokeSrcObjectURL();
        const file = files[0];
        this.mimeType = file.type;

        if (!RegExp(/^image\//).test(file.type)) {
          this.errorMessage = "只能上傳圖片";
          return;
        }

        this.errorMessage = null;
        this.src = URL.createObjectURL(file);
      };
      tempInput.click();
    },

    updatePreview({ coordinates, image }) {
      this.preview = {
        coordinates,
        image,
      };
    },
    // revoke objectUrl and init state
    clear() {
      this.attemptRevokeSrcObjectURL();
      this.errorMessage = null;
      this.mimeType = null;
      this.src = null;
      this.preview = createPreviewState();
    },
    attemptRevokeSrcObjectURL() {
      if (this.src) URL.revokeObjectURL(this.src);
      this.src = null;
    },
  },
  destroyed() {
    this.attemptRevokeSrcObjectURL();
  },
};
</script>
