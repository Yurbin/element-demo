<template>
  <div class="essp-input-number"
       :class="[
    {
      'el-input-group': $slots.prepend || $slots.append,
      'el-input-group--append': $slots.append,
      'el-input-group--prepend': $slots.prepend
    }
  ]">
    <!-- 前置元素 -->
    <div class="el-input-group__prepend"
         v-if="$slots.prepend">
      <slot name="prepend"></slot>
    </div>
    <!-- 单数字框 -->
    <div v-if="!ranged">
      <el-input ref="input"
                :value="currentInputValue"
                :placeholder="placeholder"
                :disabled="inputNumberDisabled"
                :max="max"
                :min="min"
                :name="name"
                :label="label"
                @blur="handleBlur"
                @focus="handleFocus"
                @change="handleInputChange">
      </el-input>
    </div>
    <!-- 数字区间 -->
    <div v-else
         class="el-date-editor el-range-editor el-input__inner">
      <input autocomplete="off"
             :placeholder="startPlaceholder"
             :value="displayValue && displayValue[0]"
             :disabled="inputNumberDisabled"
             v-bind="firstInputId"
             :readonly="!editable || readonly"
             @change="handleStartChange"
             @focus="handleFocus"
             class="el-range-input">
      <slot name="range-separator">
        <span class="el-range-separator">{{ rangeSeparator }}</span>
      </slot>
      <input autocomplete="off"
             :placeholder="endPlaceholder"
             :value="displayValue && displayValue[1]"
             :disabled="inputNumberDisabled"
             v-bind="secondInputId"
             :readonly="!editable || readonly"
             @change="handleEndChange"
             @focus="handleFocus"
             class="el-range-input">
    </div>

    <!-- 后置元素 -->
    <div class="el-input-group__append"
         v-if="$slots.append">
      <slot name="append"></slot>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import Focus from "element-ui/src/mixins/focus";
import RepeatClick from "element-ui/src/directives/repeat-click";

export default {
  name: "EsspInputNumber",
  mixins: [Focus("input")],
  inject: {
    elForm: {
      default: ""
    },
    elFormItem: {
      default: ""
    }
  },
  directives: {
    repeatClick: RepeatClick
  },
  props: {
    type: {
      type: String,
      default: "number"
    },
    max: {
      type: Number,
      default: Infinity
    },
    min: {
      type: Number,
      default: -Infinity
    },
    value: {},
    disabled: Boolean,
    readonly: Boolean,
    editable: {
      type: Boolean,
      default: true
    },
    size: String,
    controls: {
      type: Boolean,
      default: true
    },
    controlsPosition: {
      type: String,
      default: ""
    },
    name: String,
    label: String,
    placeholder: {
      type: String,
      default: "请输入数字"
    },
    startPlaceholder: {
      type: String,
      default: "开始值"
    },
    endPlaceholder: {
      type: String,
      default: "结束值"
    },
    rangeSeparator: {
      default: "至"
    },
    precision: {
      type: Number,
      validator(val) {
        return val >= 0 && val === parseInt(val, 10);
      }
    }
  },
  data() {
    return {
      currentValue: 0,
      userInput: null
    };
  },
  watch: {
    value: {
      immediate: true,
      handler(value) {
        let newVal = value === undefined ? value : Number(value);
        if (newVal !== undefined) {
          if (isNaN(newVal)) {
            return;
          }
          if (this.precision !== undefined) {
            newVal = this.toPrecision(newVal, this.precision);
          }
        }
        if (newVal >= this.max) newVal = this.max;
        if (newVal <= this.min) newVal = this.min;
        this.currentValue = newVal;
        this.$emit("input", newVal);
      }
    }
  },
  computed: {
    ranged() {
      return this.type.indexOf("range") > -1;
    },
    numPrecision() {
      const { value, step, getPrecision, precision } = this;
      const stepPrecision = getPrecision(step);
      if (precision !== undefined) {
        if (stepPrecision > precision) {
          // eslint-disable-next-line
          console.warn(
            "[Element Warn][InputNumber]precision should not be less than the decimal places of step"
          );
        }
        return precision;
      } else {
        return Math.max(getPrecision(value), stepPrecision);
      }
    },
    currentInputValue() {
      const currentValue = this.currentValue;
      if (typeof currentValue === "number" && this.precision !== undefined) {
        return currentValue.toFixed(this.precision);
      } else {
        return currentValue;
      }
    },
    inputNumberDisabled() {
      return this.disabled || (this.elForm || {}).disabled;
    },
    displayValue() {
      if (Array.isArray(this.userInput)) {
        return [this.userInput[0], this.userInput[1]];
      }
      return undefined;
    },
    firstInputId() {
      const obj = {};
      let id;
      if (this.ranged) {
        id = this.id && this.id[0];
      } else {
        id = this.id;
      }
      if (id) obj.id = id;
      return obj;
    },

    secondInputId() {
      const obj = {};
      let id;
      if (this.ranged) {
        id = this.id && this.id[1];
      }
      if (id) obj.id = id;
      return obj;
    }
  },
  methods: {
    toPrecision(num, precision) {
      if (precision === undefined) precision = this.numPrecision;
      return parseFloat(parseFloat(Number(num).toFixed(precision)));
    },
    getPrecision(value) {
      if (value === undefined) return 0;
      const valueString = value.toString();
      const dotPosition = valueString.indexOf(".");
      let precision = 0;
      if (dotPosition !== -1) {
        precision = valueString.length - dotPosition - 1;
      }
      return precision;
    },
    handleBlur(event) {
      this.$emit("blur", event);
      this.$refs.input.setCurrentValue(this.currentInputValue);
    },
    handleFocus(event) {
      this.$emit("focus", event);
    },
    setCurrentValue(newVal) {
      const oldVal = this.currentValue;
      if (typeof newVal === "number" && this.precision !== undefined) {
        newVal = this.toPrecision(newVal, this.precision);
      }
      if (newVal >= this.max) newVal = this.max;
      if (newVal <= this.min) newVal = this.min;
      if (oldVal === newVal) {
        this.$refs.input.setCurrentValue(this.currentInputValue);
        return;
      }
      this.$emit("input", newVal);
      this.$emit("change", newVal, oldVal);
      this.currentValue = newVal;
    },
    handleInputChange(value) {
      const newVal = value === "" ? undefined : Number(value);
      if (!isNaN(newVal) || value === "") {
        this.setCurrentValue(newVal);
      }
    },
    select() {
      this.$refs.input.select();
    },
    handleStartChange(event) {
      if (this.userInput) {
        this.userInput = [event.target.value, this.userInput[1]];
      } else {
        this.userInput = [event.target.value, null];
      }
    },
    handleEndChange() {
      if (this.userInput) {
        this.userInput = [this.userInput[0], event.target.value];
      } else {
        this.userInput = [null, event.target.value];
      }
    }
  }
};
</script>

<style lang="less" scoped>
.essp-input-number {
  .el-date-editor {
    &.el-input__inner {
      width: 100%;
    }
    .el-range-input {
      width: calc(50% - 10px);
    }
    .el-range-separator {
      width: 20px;
    }
  }
}
</style>