<template>
  <div class="card">
    <h5 class="card-header">{{ $t("card.routingUpload.title") }}</h5>
    <p class="text-gray-700 text-base mb-4">
      <i18n-t keypath="card.routingUpload.description">
        <a
          href="https://github.com/treee111/wahooMapsCreator/blob/develop/docs/HOWTO_ADD_ROUTING_TILES_MANUALLY.MD"
          class="underline"
          target="_blank"
          >docs of wahooMapsCreator</a
        >
      </i18n-t>
    </p>

    <div class="upload-routing">
      <div>
        <select-directory
          @selected="selected"
          :label="$t('card.routingUpload.action.selectDirectory.label')"
          ref="directorySelector"
        />
        <div v-if="tilesInfo" class="text-xs p-1 mt-2 rounded-md">
          {{ tilesInfo }}
        </div>
      </div>

      <div class="pt-4 flex gap-2">
        <action-button
          :label="$t('card.routingUpload.action.upload.label')"
          :loadingLabel="$t('card.routingUpload.action.upload.progress')"
          :disabled="!path"
          :disabledLabel="
            $t('card.routingUpload.action.upload.noDirectorySelected')
          "
          :action="uploadRouting"
          :progress="progress"
        />
        <button v-if="path && !progress" class="btn" @click="reset">
          {{ $t("card.routingUpload.action.reset.label") }}
        </button>
      </div>
      <div
        v-if="message"
        class="bg-green-400 text-white text-xs p-1 mt-2 rounded-md"
      >
        {{ message }}
      </div>
      <div
        v-if="error"
        class="bg-red-400 text-white text-xs p-1 mt-2 rounded-md"
      >
        ERROR: {{ error }}
      </div>
    </div>
  </div>
</template>

<script>
import ActionButton from "./action-button.vue";
import SelectDirectory from "./select-directory.vue";
import { readableBytes } from "@/ui/file-utils.js";

export default {
  props: ["deviceId"],
  inject: ["backend"],
  components: {
    ActionButton,
    SelectDirectory,
  },
  data() {
    return {
      loader: false,
      message: null,
      tilesInfo: null,
      error: null,
      path: null,
      progress: 0,
      files: [],
    };
  },
  methods: {
    selected(path) {
      this.path = path;
      this.error = null;

      if (path === null) {
        this.$refs.directorySelector.reset();
        return;
      }

      this.backend.findRoutingTiles(path).then((files) => {
        if (files.length > 0) {
          let totalSizeFormatted = readableBytes(
            files.reduce((prev, v) => prev + v.size, 0)
          );
          this.tilesInfo = this.$i18n.t(
            "card.routingUpload.action.selectDirectory.foundTiles",
            { totalFiles: files.length, totalSize: totalSizeFormatted }
          );
          this.files = files;
        } else {
          this.error = this.$i18n.t(
            "card.routingUpload.action.selectDirectory.noTilesFound"
          );
          this.$refs.directorySelector.reset();
          this.path = null;
        }
      });
    },
    uploadRouting() {
      this.error = null;
      this.message = null;

      return this.backend
        .uploadRouting(this.deviceId, this.files, (progress) => {
          this.progress = (
            (progress.uploadedFiles / progress.totalFiles) *
            100
          ).toFixed(1);
        })
        .then(() => {
          this.message = this.$i18n.t(
            "card.routingUpload.action.upload.success"
          );
          this.progress = 0;

          setTimeout(() => {
            this.message = null;
          }, 10000);
        })
        .catch((err) => {
          this.error = err;
        });
    },
    reset() {
      this.message = null;
      this.tilesInfo = null;
      this.error = null;
      this.path = null;
      this.files = [];
      this.progress = 0;
      this.$refs.directorySelector.reset();
    },
  },
};
</script>
