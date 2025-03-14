<template>
  <component
    :is="componentName"
    :id="name"
    :name="name"
    :input-name="id"
    :value="value"
    :type="typeOrDefault"
    :length="length"
    :readonly="readonly"
    :required="required"
    :loading="loading"
    :options="optionsWithDefaults"
    :new-item="newItem"
    :relation="relation"
    :fields="fieldsFormatted"
    :collection="collection"
    :values="values"
    :width="width"
    class="v-ext-input selectable"
    @input="$emit('input', $event)"
    @setfield="$emit('setfield', $event)"
  >
    <slot />
  </component>
</template>

<script>
import Vue from "vue";
import loadExtension from "../../../../helpers/load-extension";
import componentExists from "../../../../helpers/component-exists";
import InputFallback from "./input-fallback.vue";
import InputLoading from "./input-loading.vue";
import { datatypes } from "../../../../type-map";

export default {
  name: "VExtInput",
  props: {
    id: {
      type: String,
      default: null
    },
    name: {
      type: String,
      required: true
    },
    value: {
      type: null,
      default: null
    },
    type: {
      type: String,
      default: null
    },
    collection: {
      type: String,
      default: null
    },
    datatype: {
      type: String,
      default: null
    },
    length: {
      type: [String, Number],
      default: null
    },
    readonly: {
      type: Boolean,
      default: false
    },
    required: {
      type: Boolean,
      default: false
    },
    loading: {
      type: Boolean,
      default: false
    },
    options: {
      type: Object,
      default: () => ({})
    },
    newItem: {
      type: Boolean,
      default: false
    },
    relation: {
      type: Object,
      default: null
    },
    fields: {
      type: [Array, Object],
      default: null
    },
    values: {
      type: Object,
      default: null
    },
    width: {
      type: String,
      default: null,
      validator(val) {
        return ["half", "half-left", "half-right", "full", "fill"].includes(val);
      }
    }
  },
  computed: {
    interfaces() {
      return this.$store.state.extensions.interfaces;
    },
    interface() {
      if (this.id === null) return this.interfaceFallback;
      return this.interfaces && this.interfaces[this.id];
    },
    databaseVendor() {
      return this.$store.state.serverInfo.databaseVendor;
    },
    componentName() {
      if (this.id === null) return this.componentNameFallback;
      return `input-${this.id}`;
    },
    typeOrDefault() {
      if (!this.interface) return null;
      return this.type ? this.type : this.interface && this.interface.types[0];
    },
    optionsWithDefaults() {
      if (!this.interface) return {};

      const defaults = _.mapValues(this.interface.options, settings => settings.default || null);

      return {
        ...defaults,
        ...this.options
      };
    },
    // NOTE:
    // We want to move to a setup where everything is an array instead of a keyed object. This is an
    // in-between patch that allows us to use the array style already while we're refactoring the
    // rest of the app to use it as well
    fieldsFormatted() {
      if (Array.isArray(this.fields)) {
        return _.keyBy(this.fields, "field");
      }

      return this.fields;
    },
    componentNameFallback() {
      return `input-${this.interfaceFallback.id}`;
    },
    interfaceFallback() {
      // Default to text-input if all else fails
      if (this.datatype == null) return this.interfaces["text-input"];

      // Lookup the raw db datatype based on the current vendor in the type-map
      // to extract the fallback interface to use.
      const fallback = datatypes[this.databaseVendor][this.datatype].fallbackInterface;

      return this.interfaces[fallback];
    }
  },
  watch: {
    id() {
      this.registerInterface();
    }
  },
  created() {
    this.registerInterface();
  },
  methods: {
    /**
     * Register the extension as component (if it hasn't been registered before yet)
     */
    registerInterface() {
      // If component already exists, do nothing
      if (componentExists(this.componentName)) return;

      let component;

      if (this.interface.core) {
        component = import("@/interfaces/" + this.interface.id + "/input.vue");
      } else {
        const filePath = `${this.$store.state.apiRootPath}${this.interface.path.replace(
          "meta.json",
          "input.js"
        )}`;

        component = loadExtension(filePath);
      }

      Vue.component(this.componentName, () => ({
        component: component,
        error: InputFallback,
        loading: InputLoading
      }));
    }
  }
};
</script>
