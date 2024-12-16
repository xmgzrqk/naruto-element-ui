<template>
  <div style="margin: 20px;">
    <el-input v-model="input" placeholder="请输入内容"></el-input>
    <el-date-picker v-model="date" type="daterange" placeholder="date" :picker-options="{disabledDate}" format="yyyy年Qq季度" value-format="timestamp"></el-date-picker>
    <el-date-picker v-model="date1" format="yyyy年qQ季度" type="quarterrange" start-placeholder="date" :picker-options="{disabledDate}" @change="handleChange1"></el-date-picker>
    <el-date-picker
      v-model="value"
      type="week"
      :picker-options="{ disabledDate: disabledDate, firstDayOfWeek: 4 }"
      placeholder="选择周"
      @change="handleChange"
    >
    </el-date-picker>
    <el-date-picker
      v-model="value2"
      type="weekrange"
      :picker-options="{ disabledDate: disabledDate, firstDayOfWeek: 4 }"
      placeholder="选择周"
      @change="handleChange"
    >
    </el-date-picker>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        input: 'Hello Element UIxxxxxxxxxxxx!',
        date: '',
        date1: '',
        value: '',
        value2: '',
      };
    },
    computed: {
      disabledDate: () => {
        return (date) => {
          return date > new Date('2025-08-08');
        }
      }
    },
    methods: {
      handleChange(value) {
        console.log(value);
      },
      handleChange1(value) {
        console.log(value);
        console.log(value.map(date => date.toLocaleDateString()));
      },
    },
  };
</script>
