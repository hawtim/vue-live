<template>
  <component :is="layout ? layout : VueLiveDefaultLayout" v-bind="layoutProps" :code="stableCode" :language="lang"
    :prism-lang="prismLang" :requires="requires" :data-scope="dataScope" :components="components">
    <template v-slot:editor>
      <Editor :code="stableCode" :delay="delay" :prism-lang="prismLang" :editor-props="editorProps" :error="error"
        :jsx="jsx" :squiggles="squiggles" @change="updatePreview" />
    </template>
    <template v-slot:preview>
      <!--
        * Emitted every time the component rendered throws an error
        * Catches runtime and compilation errors
        * @event error
        * @property { Error } - the error thrown
      -->
      <Preview :key="codeKey" :code="model" @detect-language="switchLanguage" @error="handleError"
        @success="handleSuccess" :components="components" :directives="directives" :requires="requires" :jsx="jsx" :data-scope="dataScope"
        :check-variable-availability="checkVariableAvailability" />
    </template>
  </component>
</template>
<script lang="ts">
import { defineComponent, markRaw } from "vue";
import hash from "hash-sum";

import Preview from "./Preview.vue";
import Editor from "./Editor.vue";
import VueLiveDefaultLayout from "./VueLiveDefaultLayout.vue";

const LANG_TO_PRISM = {
  vue: "html",
  vsg: "vsg",
};

const UPDATE_DELAY = 300;

export default defineComponent({
  name: "VueLive",
  components: { Preview, Editor },
  props: {
    /**
     * code rendered in the preview and the editor
     */
    code: {
      type: String,
      required: true,
    },
    /**
     * Layout vue component with 2 slots named `editor` & `preview`
     */
    layout: {
      type: Object,
      default: undefined,
    },
    /**
     * Hashtable of auto-registered components
     * @example { DatePicker: VueDatePicker }
     * @example { VueDatePicker }
     */
    components: {
      type: Object,
      default: () => { },
    },
    /**
     * Hashtable of auto-registered directives
     * @example { Tooltip: VueTooltip }
     * @example { VueTooltip }
     */
    directives: {
      type: Object,
      default: () => { },
    },
    /**
     * Hashtable of modules available in require and import statements
     * in the Preview component
     * @example { lodash: require("lodash") }
     * @example { "foo/bar.js": import("foo/bar.js") }
     */
    requires: {
      type: Object,
      default: () => { },
    },
    /**
     * Time in ms debouncing updates to the preview
     */
    delay: {
      type: Number,
      default: UPDATE_DELAY,
    },
    /**
     * Do the code contain JSX rendered functions
     */
    jsx: {
      type: Boolean,
      default: false,
    },
    /**
     * These props will be passed as a spreat to your layout
     * They can be used to change the style
     */
    layoutProps: {
      type: Object,
      default: undefined,
    },
    /**
     * Props of vue-prism-editor
     * @example { lineNumbers: true }
     * @see https://github.com/koca/vue-prism-editor
     */
    editorProps: {
      type: Object,
      default: () => ({}),
    },
    /**
     * Outside data to the preview
     * @example { count: 1 }
     */
    dataScope: {
      type: Object,
      default: () => { },
    },
    /**
     * Set if checking variables for availability
     * when used in template
     * NOTE: if this is not checked, undefined vars will yield a blank output
     */
    checkVariableAvailability: {
      type: Boolean,
      default: true,
    },
    /**
     * Show the red markings
     * where the compiler found errors
     */
    squiggles: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      model: this.code,
      lang: "vue",
      prismLang: "html",
      VueLiveDefaultLayout: markRaw(VueLiveDefaultLayout),
      /**
       * this data only gets changed when changing language.
       * it allows for copy and pasting without having the code
       * editor repainted every keystroke
       */
      stableCode: this.code,
      error: undefined,
    };
  },
  computed: {
    codeKey() {
      return hash(this.model);
    },
  },
  watch: {
    code(newCode) {
      this.stableCode = newCode;
      this.model = newCode;
    },
  },
  methods: {
    updatePreview(code: string) {
      this.stableCode = code;
      this.model = code;
      this.$emit("change", code);
    },
    switchLanguage(newLang: "vue" | "vsg") {
      this.lang = newLang;
      const newPrismLang = LANG_TO_PRISM[newLang];
      if (this.prismLang !== newPrismLang) {
        this.prismLang = newPrismLang;
        this.stableCode = this.model;
      }
    },
    handleSuccess() {
      this.error = undefined;
      this.$emit("success");
    },
    handleError(e: any) {
      this.error = e;
      this.$emit("error", e);
    },
  },
});
</script>
