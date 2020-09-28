<template>
  <div :class="{ 'input-group': bootstrapStyling }">
    <!-- Calendar Button -->
    <span
      v-if="calendarButton"
      class="vdp-datepicker__calendar-button"
      :class="{ 'input-group-prepend': bootstrapStyling }"
      @click="showCalendar"
      v-bind:style="{ 'cursor:not-allowed;': disabled }"
    >
      <span :class="{ 'input-group-text': bootstrapStyling }">
        <i :class="calendarButtonIcon">
          {{ calendarButtonIconContent }}
          <span v-if="!calendarButtonIcon">&hellip;</span>
        </i>
      </span>
    </span>
    <!-- Input -->
    <input
      :type="inline ? 'hidden' : 'text'"
      :class="computedInputClass"
      :name="name"
      :ref="refName"
      :id="id"
      :value="formattedValue"
      :open-date="openDate"
      :placeholder="placeholder"
      :clear-button="clearButton"
      :disabled="disabled"
      :required="required"
      :readonly="!typeable"
      @click="showCalendar"
      @keyup="parseTypedDate"
      @blur="inputBlurred"
      autocomplete="off"
    />
    <!-- Clear Button -->
    <span
      v-if="clearButton && selectedDate"
      class="vdp-datepicker__clear-button"
      :class="{ 'input-group-append': bootstrapStyling }"
      @click="clearDate()"
    >
      <span :class="{ 'input-group-text': bootstrapStyling }">
        <i :class="clearButtonIcon">
          <span v-if="!clearButtonIcon">&times;</span>
        </i>
      </span>
    </span>
    <slot name="afterDateInput"></slot>
  </div>
</template>
<script>
import { makeDateUtils } from "../utils/DateUtils";
export default {
  props: {
    selectedDate: Date,
    resetTypedDate: [Date],
    format: [String, Function],
    translation: Object,
    inline: Boolean,
    id: String,
    name: String,
    refName: String,
    openDate: Date,
    placeholder: String,
    inputClass: [String, Object, Array],
    clearButton: Boolean,
    clearButtonIcon: String,
    calendarButton: Boolean,
    calendarButtonIcon: String,
    calendarButtonIconContent: String,
    disabled: Boolean,
    required: Boolean,
    typeable: Boolean,
    bootstrapStyling: Boolean,
    useUtc: Boolean,
  },
  data() {
    const constructedDateUtils = makeDateUtils(this.useUtc);
    return {
      input: null,
      typedDate: false,
      utils: constructedDateUtils,
    };
  },
  computed: {
    formattedValue() {
      console.log("format 1");
      if (!this.selectedDate) {
        console.log("format 2");
        return null;
      }
      if (this.typedDate) {
        console.log("format 3");
        return this.typedDate;
      }
      console.log("format 4");
      let per = this.format(this.selectedDate);
      console.log(per);

      let hans = this.utils.formatDate(
        new Date(this.selectedDate),
        this.format,
        this.translation
      );
      console.log(hans);
      return typeof this.format === "function"
        ? this.format(this.selectedDate)
        : this.utils.formatDate(
            new Date(this.selectedDate),
            this.format,
            this.translation
          );
    },

    computedInputClass() {
      if (this.bootstrapStyling) {
        if (typeof this.inputClass === "string") {
          return [this.inputClass, "form-control"].join(" ");
        }
        return { "form-control": true, ...this.inputClass };
      }
      return this.inputClass;
    },
  },
  watch: {
    resetTypedDate() {
      console.log("reset");
      this.typedDate = false;
    },
  },
  methods: {
    showCalendar() {
      this.$emit("showCalendar");
    },
    /**
     * Attempt to parse a typed date
     * @param {Event} event
     */
    parseTypedDate(event) {
      // close calendar if escape or enter are pressed
      console.log("1");
      if (
        [
          27, // escape
          13, // enter
        ].includes(event.keyCode)
      ) {
        this.input.blur();
      }

      if (this.typeable) {
        console.log("2");
        const typedDate = Date.parse(this.input.value);
        if (!isNaN(typedDate)) {
          console.log("3");
          this.typedDate = this.input.value;
          this.$emit("typedDate", new Date(this.typedDate));
        }
      }
    },
    /**
     * nullify the typed date to defer to regular formatting
     * called once the input is blurred
     */
    inputBlurred() {
      //if (this.typeable && isNaN(Date.parse(this.input.value))) {
      /**
       *removed by DKLRAS because isNAN returns true on danish locale
       * this.clearDate()
       * this.input.value = null
       * this.typedDate = null
       */
      //}

      this.$emit("closeCalendar");
    },
    /**
     * emit a clearDate event
     */
    clearDate() {
      console.log("clearing");
      this.$emit("clearDate");
    },
  },
  mounted() {
    this.input = this.$el.querySelector("input");
  },
};
// eslint-disable-next-line
</script>
