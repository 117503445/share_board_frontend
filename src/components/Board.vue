<template>
  <el-container style="height: 100%; border: 1px solid #eee">
    <el-aside width="55px" style="background-color: rgb(238, 241, 246)">
      <el-radio-group v-model="tabPosition">
        <el-radio-button label="top">top</el-radio-button>
        <el-radio-button label="right">right</el-radio-button>
        <el-radio-button label="bottom">bottom</el-radio-button>
        <el-radio-button label="left">left</el-radio-button>
      </el-radio-group>
    </el-aside> 

    <el-container>
      <el-main>
        <canvas id="canvas" width="1920" height="1080" />
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  mounted() {
    var canvas = new fabric.Canvas("canvas", {
      isDrawingMode: true,
      skipTargetFind: true,
      selectable: false,
      selection: false,
    });

    let brush = new fabric.PSBrush(canvas);
    brush.width = 3;
    brush.color = "#000";
    canvas.freeDrawingBrush = brush;
  },

  data() {
    const item = {
      date: "2016-05-02",
      name: "王小虎",
      address: "上海市普陀区金沙江路 114514 弄",
    };
    return {
      tableData: Array(20).fill(item),
      tabPosition: "left",
    };
  },
  methods: {
    btn_click() {
      console.log("click");
    },
  },
  watch: {
    tabPosition: {
      handler(newValue) {
        console.log("change to " + newValue);
      },
    },
  },
};
</script>

<style>
.el-button {
  border-radius: 0px;
  width: 50px;
  height: 50px;
  margin-top: 10px;
}
</style>
