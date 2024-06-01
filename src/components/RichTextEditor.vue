<template>
  <v-input
    ref="root"
    class="atis-wysiwyg v-textarea v-text-field v-text-field--is-booted v-text-field--enclosed v-text-field--outlined v-text-field--placeholder"
    :class="{
      'error--text': isError,
      'primary--text': isFocused,
      'v-input--is-focused': isFocused || isError,
      'v-input--is-dirty': isDirty
    }"
    :error-messages="errorMessages"
    :hide-details="hideDetails && errorMessages.length === 0"
    :disabled="disabled"
    :readonly="readonly"
    :counter="counter"
    v-bind="$attrs"
    v-on="$listeners"
  >
    <fieldset aria-hidden="true" @click="$refs.input.focus()">
      <legend ref="labelBackground" :style="{ width: `${labelWidth}px` }">
        <span class="notranslate">&ZeroWidthSpace;</span>
      </legend>
    </fieldset>

    <div
        class="v-text-field__slot"
        style="padding: 4px"
    >
      <label
        ref="label"
        class="v-label theme--light"
        :class="{
          'v-label--active': isFocused || isDirty,
          'v-label--is-disabled': disabled,
          'primary--text': isFocused,
          'error--text': isError
        }"
        style="left: 0; right: auto; position: absolute"
      >
        {{ label }}
      </label>

      <div
        v-once
        style="
          padding: 12px 0;
          margin-top: 31px;
          min-width: 100%;
          max-width: 100px;
          overflow-y: auto;
          resize: vertical;
        "
        :style="{ height: `${height}px` }"
        ref="input"
        :contenteditable="editable"
        @input="onInput"
        @paste="onPaste"
        @focus="onFocus"
        @blur="onBlur"
        @keyup="updateButtonsState"
        @mousedown="updateButtonsState"
        @mouseup="updateButtonsState"
      />
    </div>

    <div
      @mousedown.prevent
      @mouseup="$refs.input.focus()"
      class="atis-wysiwyg__toolbar"
      style="
        position: absolute;
        left: 1px;
        cursor: default;
        padding: 2px;
        top: 5px;
        width: calc(100% - 2px);
        border-bottom: 1px solid rgba(0, 0, 0, 0.1);
      "
    >
      <div style="width: 5px; display: inline-block;">&nbsp;</div>
      <v-tooltip top :disabled="disabled">
        <template #activator="{ on, bind }">
          <v-btn
            small
            text
            icon
            :disabled="disabled"
            v-on="on"
            v-bind="bind"
            @click="applyBold"
            :class="{ 'v-btn--active': isBold }"
          >
            <v-icon small>mdi-format-bold</v-icon>
          </v-btn>
        </template>
        Жирный
      </v-tooltip>
      <v-tooltip top :disabled="disabled">
        <template #activator="{ on, bind }">
          <v-btn
            small
            text
            icon
            :disabled="disabled"
            v-on="on"
            v-bind="bind"
            @click="applyItalic"
            :class="{ 'v-btn--active': isItalic }"
          >
            <v-icon small>mdi-format-italic</v-icon>
          </v-btn>
        </template>
        Курсив
      </v-tooltip>
      <v-tooltip top :disabled="disabled">
        <template #activator="{ on, bind }">
          <v-btn
            small
            text
            icon
            :disabled="disabled"
            v-on="on"
            v-bind="bind"
            @click="applyUnderline"
            :class="{ 'v-btn--active': isUnderline }"
          >
            <v-icon small>mdi-format-underline</v-icon>
          </v-btn>
        </template>
        Подчеркивание
      </v-tooltip>
      <v-divider
        vertical
        style="height: 16px; padding-left: 4px; margin-right: 4px"
      />
      <v-tooltip top :disabled="disabled">
        <template #activator="{ on, bind }">
          <v-btn
            small
            text
            icon
            :disabled="disabled"
            v-on="on"
            v-bind="bind"
            @click="applyUl"
            :class="{ 'v-btn--active': isUl }"
          >
            <v-icon small>mdi-format-list-bulleted</v-icon>
          </v-btn>
        </template>
        Маркированный список
      </v-tooltip>
      <v-tooltip top :disabled="disabled">
        <template #activator="{ on, bind }">
          <v-btn
            small
            text
            icon
            :disabled="disabled"
            v-on="on"
            v-bind="bind"
            @click="applyOl"
            :class="{ 'v-btn--active': isOl }"
          >
            <v-icon small>mdi-format-list-numbered</v-icon>
          </v-btn>
        </template>
        Нумерованный список
      </v-tooltip>
      <v-divider
        vertical
        style="height: 16px; padding-left: 4px; margin-right: 4px"
      />
      <v-menu open-on-hover down offset-y :disabled="disabled">
        <template v-slot:activator="{ on, attrs }">
          <v-btn
              small
              text icon
              :disabled="disabled"
              v-on="on"
              v-bind="attrs"
          >
            <v-icon small>mdi-format-color-text</v-icon>
            <div class="rainbow-under-text-color-icon" />
          </v-btn>
        </template>
        <div
          class="atis-wysiwyg__color-picker white"
          @mousedown.prevent
          @mouseup="$refs.input.focus()"
        >
          <v-color-picker
            hide-canvas
            hide-inputs
            hide-sliders
            show-swatches
            @update:color="(color) => applyColor(color.hex)"
            :swatches="[
              ['#000000', '#4CAF50'],
              ['#82b1ff', '#FB8C00'],
              ['#1b7dc9', '#B00020'],
            ]"
          />
        </div>
      </v-menu>
    </div>
  </v-input>
</template>

<style>
.v-input.atis-wysiwyg [contenteditable]:focus {
  outline: 0 solid transparent;
}
.v-input.atis-wysiwyg [contenteditable] {
  color: rgba(0, 0, 0, 0.87);
}

.v-input.atis-wysiwyg .v-input__slot .v-text-field__slot .v-label {
  top: 50px;
}
.v-input.atis-wysiwyg .v-input__slot .v-text-field__slot .v-label.v-label--active {
  transform: translateY(-56px) scale(0.75);
}

.v-input.atis-wysiwyg .atis-wysiwyg__toolbar .v-btn {
  border-radius: 6px;
}
.v-input.atis-wysiwyg .rainbow-under-text-color-icon {
  width: 14px;
  height: 3px;
  position: absolute;
  background: linear-gradient(
    to right,
    rgb(101, 84, 192) 25%,
    rgb(0, 184, 217) 25%,
    rgb(0, 184, 217) 50%,
    rgb(255, 153, 31) 50%,
    rgb(255, 153, 31) 75%,
    rgb(222, 53, 11) 75%
  );
  bottom: 4px;
}
.v-input.atis-wysiwyg.v-input--is-disabled .rainbow-under-text-color-icon {
  opacity: 0.25;
}

.v-input.atis-wysiwyg.v-text-field--rounded > .v-input__control > .v-input__slot {
  padding-right: 13px;
}

.atis-wysiwyg__color-picker .v-color-picker__swatch {
  margin-bottom: 0;
}
.atis-wysiwyg__color-picker .v-color-picker__swatch .v-color-picker__color {
  width: 20px;
  height: 20px;
}
.atis-wysiwyg__color-picker .v-color-picker__swatch .v-color-picker__color:hover {
  box-shadow: 0 0 0 1px white, 0 0 0 2px rgba(0, 0, 0, 0.2);
}

.atis-wysiwyg__color-picker > .v-input__control > .v-input__slot {
  padding-left: 10px;
}
</style>

<script>
export default {

  props: {
    value: { type: String, default: "" },
    label: { type: String, default: "" },
    height: { type: String, default: "120" },
    disabled: { type: Boolean, default: false },
    readonly: { type: Boolean, default: false },
    counter: { type: Boolean, default: false },
    hideDetails: { type: Boolean, default: false },
    errorMessages: { type: Array, default: () => [] },
  },

  data() {
    return {
      text: "",
      isFocused: false,
      isBold: false,
      isItalic: false,
      isUnderline: false,
      isUl: false,
      isOl: false,
    };
  },

  computed: {
    isError() {
      return this.errorMessages.length !== 0;
    },
    isDirty() {
      return this.text !== "";
    },
    labelWidth() {
      if (this.isFocused === false && this.isDirty === false) {
        // if no focus and textfield is clear
        return 0;
      }
      if (!this.$refs.label) {
        // if no label
        return 0;
      }

      // the label will be scaled down by the CSS property scale(0.75),
      // so we'll replicate this behaviour but also add a little gap
      return this.$refs.label.clientWidth * 0.75 + 6;
    },
    editable() {
      return !this.disabled && !this.readonly;
    }
  },

  watch: {
    value(value) {
      if (value === null) {
        this.clear();
      } else {
        this.text = value;
      }
    },
  },

  async mounted() {
    await this.$nextTick();
    document.execCommand('defaultParagraphSeparator', false, 'p');

    if (typeof this.value === "string") {
      this.text = this.value;
    } else {
      this.clear(true);
    }
  },

  methods: {
    applyBold() {
      document.execCommand("bold");
      this.checkIsBold();
    },
    applyItalic() {
      document.execCommand("italic");
      this.checkIsItalic();
    },
    applyUnderline() {
      document.execCommand("underline");
      this.checkIsUnderline();
    },
    applyUl() {
      document.execCommand("insertUnorderedList");
    },
    applyColor(color) {
      if (color === "#FF0000") { return; } // lazy fix, but it works for now (#ff0000 - default v-color-picker value)
      document.execCommand("foreColor", false, color);
    },
    applyOl() {
      document.execCommand("insertOrderedList");
    },

    checkIsFocused() {
      this.isFocused = document.activeElement === this.$refs.input;
    },
    checkIsBold() {
      this.isBold = this.isFocused ? document.queryCommandState("bold") : false;
    },
    checkIsItalic() {
      this.isItalic = this.isFocused
        ? document.queryCommandState("italic")
        : false;
    },
    checkIsUnderline() {
      this.isUnderline = this.isFocused
        ? document.queryCommandState("underline")
        : false;
    },

    onFocus() {
      this.checkIsFocused();
      this.updateButtonsState();
    },
    onBlur() {
      this.checkIsFocused();
      this.updateButtonsState();
      if (this.$refs.input.innerText.trim() === "") {
        this.clear();
      }
    },

    updateButtonsState() {
      this.checkIsBold();
      this.checkIsItalic();
      this.checkIsUnderline();
    },

    clear(noEmit) {
      this.text = "";
      if (!noEmit) {
        this.$emit("input", this.text);
      }
    },

    onInput(event) {
      // update toolbar buttons
      this.updateButtonsState();
      this.$emit("input", event.target.innerHTML);
    },

    onPaste(event) {
      event.preventDefault();
      const text = event.clipboardData.getData("text/plain");
      document.execCommand("insertHTML", false, text);
    },
  },
};
</script>
