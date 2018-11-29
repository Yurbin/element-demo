<template>
  <div id="app">
    <essp-input-number class="essp-input-number"
                       v-model="number1"
                       placeholder="请输入数字">
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
                       v-model="number2"
                       placeholder="请输入数字">
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
                       type="numberrange">
      <!-- <el-select v-model="select2"
                 slot="append"
                 placeholder="请选择">
        <el-option label="餐厅名"
                   value="1"></el-option>
        <el-option label="订单号"
                   value="2"></el-option>
        <el-option label="用户电话"
                   value="3"></el-option>
      </el-select> -->
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
      number1: undefined,
      number2: undefined
    };
  },
  methods: {}
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
        width: 100px;
      }
    }
  }
}
</style>