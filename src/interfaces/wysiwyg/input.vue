<template>
  <div class="interface-wysiwyg">
    <Editor
      ref="editorElement"
      :init="initOptions"
      :value="value"
      @onKeyUp="updateValue"
      @onExecCommand="updateValue"
      @onBlur="updateValue"
      @onPaste="updateValue"
      @onUndo="updateValue"
      @onRedo="updateValue"
    />
    <v-modal
      v-if="newInlineFile"
      :title="$t('file_upload')"
      :buttons="{
        done: {
          text: $t('done')
        }
      }"
      @close="newInlineFile = false"
      @done="selectCallback"
    >
      <div class="body">
        <v-ext-input
          id="file"
          name="file"
          :required="false"
          :readonly="false"
          :options="fileInputOptions"
          type="file"
          datatype="INT"
          :value="selectedFile"
          :relation="relation"
          :fields="null"
          collection="directus_files"
          :values="null"
          :length="10"
          :new-item="newItem"
          @input="selectedFile = $event"
        />
      </div>
    </v-modal>
  </div>
</template>

<script>
import mixin from "@directus/extension-toolkit/mixins/interface";

import "tinymce/tinymce";
import "tinymce/themes/silver";
import "tinymce/plugins/media/plugin";
import "tinymce/plugins/table/plugin";
import "tinymce/plugins/hr/plugin";
import "tinymce/plugins/lists/plugin";
import "tinymce/plugins/image/plugin";
import "tinymce/plugins/imagetools/plugin";
import "tinymce/plugins/link/plugin";
import "tinymce/plugins/pagebreak/plugin";
import "tinymce/plugins/code/plugin";
import "tinymce/plugins/insertdatetime/plugin";
import "tinymce/plugins/autoresize/plugin";
import "tinymce/plugins/paste/plugin";
import "tinymce/plugins/preview/plugin";

import Editor from "@tinymce/tinymce-vue";

function cssVar(name) {
  return getComputedStyle(document.body).getPropertyValue(name);
}

export default {
  components: {
    Editor
  },
  mixins: [mixin],
  data() {
    return {
      newInlineFile: false,
      selectedFile: null,
      selectCallback: () => {}
    };
  },
  computed: {
    fileInputOptions() {
      return {
        viewOptions: {
          content: "description",
          src: "data",
          subtitle: "type",
          title: "title"
        },
        viewType: "cards"
      };
    },
    initOptions() {
      const styleFormats = this.getStyleFormats();
      let toolbarString = this.options.toolbar.join(" ");

      if (styleFormats) {
        toolbarString += " styleselect";
      }

      return {
        skin: false,
        skin_url: false,
        content_css: false,
        content_style: this.contentStyle,
        plugins:
          "media table hr lists image link pagebreak code insertdatetime autoresize paste preview",
        branding: false,
        max_height: 1000,
        elementpath: false,
        statusbar: false,
        menubar: false,
        convert_urls: false,
        readonly: this.readonly,
        extended_valid_elements: "audio[loop],source",
        toolbar: toolbarString,
        style_formats: styleFormats,
        file_picker_callback: this.selectFile
      };
    },
    contentStyle() {
      return `
        body {
          color: ${cssVar("--input-text-color")};
          background-color: ${cssVar("--input-background-color")};
          margin: 20px;
          font-family: 'Roboto', sans-serif;
          -webkit-font-smoothing: antialiased;
          text-rendering: optimizeLegibility;
          -moz-osx-font-smoothing: grayscale;
        }
        h1 {
          font-family: 'Merriweather', serif;
          font-size: 44px;
          line-height: 52px;
          font-weight: 300;
          margin-bottom: 0;
        }
        h2 {
          font-size: 34px;
          line-height: 38px;
          font-weight: 600;
          margin-top: 60px;
          margin-bottom: 0;
        }
        h3 {
          font-size: 26px;
          line-height: 31px;
          font-weight: 600;
          margin-top: 40px;
          margin-bottom: 0;
        }
        h4 {
          font-size: 22px;
          line-height: 28px;
          font-weight: 600;
          margin-top: 40px;
          margin-bottom: 0;
        }
        h5 {
          font-size: 18px;
          line-height: 26px;
          font-weight: 600;
          margin-top: 40px;
          margin-bottom: 0;
        }
        h6 {
          font-size: 16px;
          line-height: 24px;
          font-weight: 600;
          margin-top: 40px;
          margin-bottom: 0;
        }
        p {
          font-family: 'Merriweather', serif;
          font-size: 16px;
          line-height: 32px;
          margin-top: 20px;
          margin-bottom: 20px;
        }
        a {
          color: #546e7a;
        }
        ul,ol {
          font-family: 'Merriweather', serif;
          font-size: 18px;
          line-height: 34px;
          margin: 24px 0;
        }
        ul ul,
        ol ol,
        ul ol,
        ol ul {
          margin: 0;
        }
        b,strong {
          font-weight: 600;
        }
        code {
          font-size: 18px;
          line-height: 34px;
          padding: 2px 4px;
          font-family: 'Roboto Mono', monospace;
          background-color: #eceff1;
          border-radius: 3px;
          overflow-wrap: break-word;
        }
        pre {
          font-size: 18px;
          line-height: 24px;
          padding: 20px;
          font-family: 'Roboto Mono', monospace;
          background-color: #eceff1;
          border-radius: 3px;
          overflow: auto;
        }
        blockquote {
          font-family: 'Merriweather', serif;
          font-size: 18px;
          line-height: 34px;
          border-left: 2px solid #546e7a;
          padding-left: 10px;
          margin-left: -10px;
          font-style: italic;
        }
        video,
        iframe,
        img {
          max-width: 100%;
          border-radius: 3px;
          height: auto;
        }
        hr {
          border: 0;
          margin-top: 52px;
          margin-bottom: 56px;
          text-align: center;
        }
        hr:after {
          content: "...";
          font-size: 28px;
          letter-spacing: 16px;
          line-height: 0;
        }
        table {
          border-collapse: collapse;
        }
        table th,
        table td {
          border: 1px solid #cfd8dc;
          padding: 0.4rem;
        }
        figure {
          display: table;
          margin: 1rem auto;
        }
        figure figcaption {
          color: #999;
          display: block;
          margin-top: 0.25rem;
          text-align: center;
        }
      `;
    }
  },
  created() {
    this.updateValue = _.debounce(this.updateValue, 200);
  },
  methods: {
    updateValue() {
      const editor = this.$refs.editorElement.editor;
      const newValue = editor.getContent();
      this.$emit("input", newValue);
    },
    getStyleFormats() {
      if (Array.isArray(this.options.custom_formats) && this.options.custom_formats.length > 0) {
        return this.options.custom_formats;
      }

      return null;
    },
    selectFile(callback) {
      this.newInlineFile = true;
      this.selectCallback = async () => {
        const { data: file } = await this.$api.getItem("directus_files", this.selectedFile.id);
        this.newInlineFile = false;

        // TODO: Make sure it returns the correct keys for non-image type files. See
        // https://www.tiny.cloud/docs/configure/file-image-upload/#example for an example
        callback(file.data.full_url, { alt: file.title });

        // Empty the selectedfile so the file isn't selected again when you add an additional file
        this.selectedFile = null;
      };
    }
  }
};
</script>

<style lang="scss" scoped>
.body {
  padding: 20px;
}

// The content CSS is not scoped, but is also not needed.
// The CSS below can be referenced for other core editor
// styles we might want to include in our override.

// @import "~tinymce/skins/ui/oxide/content.css";
// @import "~tinymce/skins/content/default/content.css";

@import "~tinymce/skins/ui/oxide/skin.css";
@import "./tinymce-overrides.css";
</style>
