<template>
  <div class="input-single-file">
    <v-card
      v-if="value"
      class="card"
      :title="image.title"
      :subtitle="subtitle + subtitleExtra"
      :src="src"
      :icon="icon"
      :href="href"
      text-background
      color="black"
      :options="{
        deselect: {
          text: $t('deselect'),
          icon: 'clear'
        },
        remove: {
          text: $t('delete'),
          icon: 'delete'
        }
      }"
      medium-image
      @deselect="$emit('input', null)"
      @remove="removeFile"
    ></v-card>
    <v-upload
      v-else
      small
      :disabled="readonly"
      class="uploader"
      :accept="options.accept"
      :multiple="false"
      @upload="saveUpload"
    ></v-upload>

    <div v-if="!value" class="buttons">
      <v-button type="button" :disabled="readonly" @click="existing = true">
        <v-icon name="playlist_add" />
        {{ $t("existing") }}
      </v-button>
    </div>

    <portal v-if="existing" to="modal">
      <v-modal
        :title="$t('choose_one')"
        :buttons="{
          done: {
            text: $t('done')
          }
        }"
        @cancel="existing = false"
        @close="existing = false"
        @done="existing = false"
      >
        <div class="content">
          <div class="search">
            <v-input
              type="search"
              :placeholder="$t('search_for_item')"
              class="search-input"
              @input="onSearchInput"
            />
          </div>
          <v-items
            class="items"
            collection="directus_files"
            :view-type="viewType"
            :selection="value ? [value] : []"
            :filters="filters"
            :view-query="viewQuery"
            :view-options="viewOptions"
            @options="setViewOptions"
            @query="setViewQuery"
            @select="saveSelection"
          ></v-items>
        </div>
      </v-modal>
    </portal>
  </div>
</template>

<script>
import mixin from "@directus/extension-toolkit/mixins/interface";
import formatSize from "../file-size/format-size";
import getIcon from "./get-icon";

export default {
  mixins: [mixin],
  data() {
    return {
      existing: false,
      viewOptionsOverride: {},
      viewTypeOverride: null,
      viewQueryOverride: {},
      filtersOverride: [],
      image: _.cloneDeep(this.value)
    };
  },
  computed: {
    subtitle() {
      if (!this.image) return "";

      return (
        this.image.filename
          .split(".")
          .pop()
          .toUpperCase() +
        " • " +
        this.$d(new Date(this.image.uploaded_on), "short")
      );
    },
    subtitleExtra() {
      // Image ? -> display dimensions and formatted filesize
      return this.image.type && this.image.type.startsWith("image")
        ? " • " + formatSize(this.image.filesize)
        : null;
    },
    src() {
      return this.image.type && this.image.type.startsWith("image")
        ? this.image.data.full_url
        : null;
    },
    icon() {
      return this.image.type && !this.image.type.startsWith("image")
        ? getIcon(this.image.type)
        : null;
    },
    href() {
      return this.image.type && this.image.type === "application/pdf"
        ? this.image.data.full_url
        : null;
    },
    viewOptions() {
      const viewOptions = this.options.viewOptions;
      return {
        ...viewOptions,
        ...this.viewOptionsOverride
      };
    },
    viewType() {
      if (this.viewTypeOverride) return this.viewTypeOverride;
      return this.options.viewType;
    },
    viewQuery() {
      const viewQuery = this.options.viewQuery;
      return {
        ...viewQuery,
        ...this.viewQueryOverride
      };
    },
    filters() {
      return [...this.options.filters, ...this.fileTypeFilters, ...this.filtersOverride];
    },
    fileTypeFilters() {
      if (
        !this.options.accept ||
        this.filtersOverride.length > 0 ||
        this.options.filters.some(filter => filter.field === "type")
      ) {
        return [];
      }

      return [
        {
          field: "type",
          operator: "in",
          value: (this.options.accept || "").trim().split(/,\s*/)
        }
      ];
    }
  },
  async created() {
    if (this.value && this.value.id) {
      try {
        let fileData = await this.$api.getItem("directus_files", this.value.id);
        this.image = fileData.data;
      } catch (e) {}
    }
    this.onSearchInput = _.debounce(this.onSearchInput, 200);
  },
  methods: {
    saveUpload(fileInfo) {
      this.image = fileInfo.data;
      // We know that the primary key of directus_files is called `id`
      this.$emit("input", { id: fileInfo.data.id });
    },
    setViewOptions(updates) {
      this.viewOptionsOverride = {
        ...this.viewOptionsOverride,
        ...updates
      };
    },
    setViewQuery(updates) {
      this.viewQueryOverride = {
        ...this.viewQueryOverride,
        ...updates
      };
    },
    onSearchInput(value) {
      this.setViewQuery({
        q: value
      });
    },
    saveSelection(value) {
      const file = value[value.length - 1];

      if (file) {
        this.image = file;
        this.$emit("input", { id: file.id });
      } else {
        this.image = null;
        this.$emit("input", null);
      }
    },
    async removeFile() {
      const file = this.value;
      await this.$api.deleteItem("directus_files", file.id);
      this.$notify({
        title: this.$t("item_deleted"),
        color: "green",
        iconMain: "check"
      });
      this.image = null;
      this.$emit("input", null);
    }
  }
};
</script>

<style lang="scss" scoped>
.card,
.uploader {
  width: 100%;
  max-width: var(--width-x-large);
}

.uploader {
  height: 236px;
}

.buttons {
  margin-top: 24px;
}

button {
  display: inline-block;
  margin-left: 20px;
  &:first-of-type {
    margin-left: 0;
  }
}

.body {
  padding: 20px;
}

.search-input {
  border-bottom: 2px solid var(--input-border-color);
  &::v-deep input {
    border-radius: 0;
    border: none;
    padding-left: var(--page-padding);
    height: var(--header-height);

    &::placeholder {
      color: var(--input-placeholder-color);
    }
  }
}

.content {
  &::v-deep .v-layout {
    height: auto;
    max-height: none;
    overflow: hidden;
  }
}
</style>
