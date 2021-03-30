<template>
  <el-container style="height: 100%; border: 1px solid #eee">
    <el-aside width="55px" style="background-color: rgb(238, 241, 246)">
      <el-radio-group v-model="tabPosition">
        <el-radio-button label="pen">pen</el-radio-button>
        <el-radio-button label="eraser">eraser</el-radio-button>
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
let canvas;

export default {
  mounted() {
    canvas = new fabric.Canvas("canvas", {
      isDrawingMode: true,
      skipTargetFind: true,
      selectable: false,
      selection: false,
    });

    canvas.on("selection:created", function (e) {
      // 选中事件
      // 清除所有选中的笔迹

      if (e.target._objects) {
        //多选删除
        e.target._objects.forEach((element) => {
          canvas.remove(element);
        });
      } else {
        //单选删除
        canvas.remove(e.target);
      }
      canvas.discardActiveObject(); //清除选中框
    });

    // https://www.npmjs.com/package/@arch-inc/fabricjs-psbrush 压感笔刷
    let brush = new fabric.PSBrush(canvas);
    brush.width = 3;
    brush.color = "#000";
    canvas.freeDrawingBrush = brush;
  },

  data() {
    return {
      tabPosition: "pen",
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
        if (newValue == "pen") {
          canvas.isDrawingMode = true;
        } else if (newValue == "eraser") {
          canvas.isDrawingMode = false;
          canvas.selection = true;
          canvas.skipTargetFind = false;
          canvas.selectable = true;
        }
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
