<template>
  <div class="essp-input-number"
       :class="[
          {
            'el-input-group': $slots.prepend || $slots.append,
            'el-input-group--append': $slots.append,
            'el-input-group--prepend': $slots.prepend
          }
        ]"
       @mouseenter="hovering = true"
       @mouseleave="hovering = false">
    <!-- 前置元素 -->
    <div class="el-input-group__prepend"
         v-if="$slots.prepend">
      <slot name="prepend"></slot>
    </div>
    <!-- 单数字框 -->
    <div v-if="!ranged">
      <el-input ref="input"
                :value="displayValue"
                :placeholder="placeholder"
                :disabled="inputNumberDisabled"
                :max="max"
                :min="min"
                :name="name"
                :label="label"
                :clearable="clearable"
                @blur="handleBlur"
                @focus="handleFocus"
                @change="handleInputChange">
      </el-input>
    </div>
    <!-- 数字区间 -->
    <div v-else
         class="el-date-editor el-range-editor el-input__inner"
         :class="[
            {
              'is-disabled': inputNumberDisabled,
              'is-active': focused,
              'el-input--prefix': $slots.prefix || prefixIcon,
              'el-input--suffix': $slots.suffix || suffixIcon || clearable
            }
          ]">
      <input :value="displayValue && displayValue[0]"
             :placeholder="startPlaceholder"
             :disabled="inputNumberDisabled"
             :max="max"
             :min="min"
             @blur="handleBlur"
             @focus="handleFocus"
             @change="handleStartChange"
             class="el-range-input">
      <slot name="range-separator">
        <span class="el-range-separator">{{ rangeSeparator }}</span>
      </slot>
      <input :value="displayValue && displayValue[1]"
             :placeholder="endPlaceholder"
             :disabled="inputNumberDisabled"
             @blur="handleBlur"
             @focus="handleFocus"
             @change="handleEndChange"
             class="el-range-input">
      <i v-if="showClear"
         class="el-input__icon el-icon-circle-close el-range__close-icon"
         @click="clear">
      </i>
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
      default: "number" // number、numberrange
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
    controls: {
      type: Boolean,
      default: true
    },
    controlsPosition: {
      type: String,
      default: ""
    },
    name: String,
    suffixIcon: String,
    prefixIcon: String,
    label: String,
    clearable: {
      type: Boolean,
      default: true
    },
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
      userInput: undefined,
      hovering: false,
      focused: false
    };
  },
  watch: {
    value: {
      immediate: true,
      handler(newVal) {
        if (newVal !== undefined) {
          if (Array.isArray(newVal)) {
            let [startVal, endVal] = newVal;
            let allIsNum = newVal.every(o => !isNaN(Number(o)));
            if (!allIsNum || startVal > endVal) {
              return;
            }
          } else {
            newVal = Number(newVal);
            if (isNaN(newVal)) {
              return;
            }
            newVal = this.parseValue(newVal);
          }
        }
        this.userInput = newVal;
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
          console.warn(
            "[Element Warn][InputNumber]precision should not be less than the decimal places of step"
          );
        }
        return precision;
      } else {
        return Math.max(getPrecision(value), stepPrecision);
      }
    },
    inputNumberDisabled() {
      return this.disabled || (this.elForm || {}).disabled;
    },
    showClear() {
      return (
        this.clearable &&
        !this.inputNumberDisabled &&
        !this.readonly &&
        this.userInput !== undefined &&
        (this.focused || this.hovering)
      );
    },
    displayValue() {
      const { userInput, precision, toPrecision } = this;
      if (Array.isArray(userInput)) {
        let [startVal, endVal] = userInput;
        if (precision !== undefined) {
          if (typeof startVal === "number") {
            startVal = Number(startVal).toFixed(precision);
          }
          if (typeof endVal === "number") {
            endVal = Number(endVal).toFixed(precision);
          }
        }
        return [startVal, endVal];
      } else {
        if (typeof userInput === "number" && precision !== undefined) {
          return userInput.toFixed(precision);
        } else {
          return userInput;
        }
      }
      return undefined;
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
      this.focused = false;
      this.setCurrentValue(this.displayValue);
    },
    handleFocus(event) {
      this.$emit("focus", event);
      this.focused = true;
    },
    valueEquals(a, b) {
      const aIsArray = a instanceof Array;
      const bIsArray = b instanceof Array;
      if (aIsArray && bIsArray) {
        if (a.length !== b.length) {
          return false;
        }
        return a.every((item, index) => item === b[index]);
      }
      if (!aIsArray && !bIsArray) {
        return a === b;
      }
      return false;
    },
    parseValue(value) {
      if (value !== undefined) {
        value = Number(value);
      }
      if (typeof value === "number" && this.precision !== undefined) {
        value = this.toPrecision(value, this.precision);
      }
      if (value >= this.max) value = this.max;
      if (value <= this.min) value = this.min;
      return value;
    },
    setCurrentValue(newVal) {
      const oldVal = this.userInput;
      const parseValue = this.parseValue;
      if (Array.isArray(newVal)) {
        let [startVal, endVal] = newVal;
        startVal = parseValue(startVal);
        endVal = parseValue(endVal);
        if (
          startVal !== undefined &&
          endVal !== undefined &&
          startVal > endVal
        ) {
          newVal = oldVal;
        } else {
          newVal = [startVal, endVal];
        }
      } else {
        newVal = parseValue(newVal);
      }

      if (!this.valueEquals(newVal, oldVal)) {
        this.$emit("input", newVal);
        this.$emit("change", newVal, oldVal);
        this.userInput = newVal;
      }
    },
    handleInputChange(value) {
      const newVal = value === "" ? undefined : Number(value);
      if (!isNaN(newVal) || value === "") {
        this.setCurrentValue(newVal);
      }
    },
    handleStartChange(event) {
      let value = event.target.value;
      const newVal = value === "" ? undefined : Number(value);
      if (!isNaN(newVal) || value === "") {
        let endVal = this.userInput && this.userInput[1];
        this.setCurrentValue([newVal, endVal]);
      }
    },
    handleEndChange(event) {
      let value = event.target.value;
      const newVal = value === "" ? undefined : Number(value);
      if (!isNaN(newVal) || value === "") {
        let startVal = this.userInput && this.userInput[0];
        this.setCurrentValue([startVal, newVal]);
      }
    },
    clear() {
      let newVal = this.ranged ? undefined : "";
      this.$emit("input", newVal);
      this.$emit("change", newVal, this.userInput);
      this.$emit("clear");
      this.setCurrentValue(newVal);
    }
  }
};
</script>

<style lang="less" scoped>
.essp-input-number {
  .el-date-editor {
    &.el-range-editor {
      width: 100%;
      display: inline-flex;
    }
    &:not(.el-input--suffix) {
      .el-range-input {
        width: calc(50% - 10px);
      }
      .el-range-separator {
        width: 20px;
      }
    }
  }
}
</style>