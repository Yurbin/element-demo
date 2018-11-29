<template>
  <div id="app">
    <essp-input-number class="essp-input-number"
                       v-model="number1">
      <el-select v-model="select1"
                 slot="prepend"
                 clearable
                 placeholder="请选择">
        <el-option v-for="item in options"
                   :key="item.value"
                   :label="item.label"
                   :value="item.value">
        </el-option>
      </el-select>
    </essp-input-number>
    <br /><br />
    <essp-input-number class="essp-input-number"
                       v-model="number2">
      <el-select v-model="select2"
                 slot="append"
                 clearable
                 placeholder="请选择">
        <el-option v-for="item in options"
                   :key="item.value"
                   :label="item.label"
                   :value="item.value">
        </el-option>
      </el-select>
    </essp-input-number>
    <br /><br />
    <essp-input-number class="essp-input-number"
                       type="numberrange"
                       v-model="number3"
                       :clearable="false"
                       :min="10"
                       :disabled="true"
                       @change="handleChange3">
      <el-select v-model="select3"
                 slot="append"
                 clearable
                 placeholder="请选择">
        <el-option v-for="item in options"
                   :key="item.value"
                   :label="item.label"
                   :value="item.value">
        </el-option>
      </el-select>
    </essp-input-number>
    <br /><br />
    <essp-input-number class="essp-input-number"
                       type="numberrange"
                       v-model="number4">
    </essp-input-number>
  </div>
</template>

<script>
/* eslint-disable */
import EsspInputNumber from "./components/EsspInputNumber";

export default {
  name: "app",
  components: {
    EsspInputNumber
  },
  data() {
    return {
      options: [
        {
          value: "1",
          label: "元"
        },
        {
          value: "2",
          label: "万元"
        }
      ],
      select1: "",
      select2: "",
      select3: "",
      number1: 111.2345,
      number2: undefined,
      number3: [1, 2],
      number4: undefined
    };
  },
  methods: {
    handleChange3(newVal, oldVal) {
      console.log(
        "newVal : ",
        newVal,
        "oldVal : ",
        oldVal,
        "number3 : ",
        this.number3
      );
    }
  }
};
</script>

<style lang="less">
#app {
  margin: 50px;
  .essp-input-number {
    font-size: 14px;
    width: 240px;
    .el-select {
      .el-input {
        width: 90px;
      }
    }
  }
}
</style>