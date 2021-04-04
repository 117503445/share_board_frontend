<template>
  <el-container style="position: absolute; height: 100%; width: 100%">
    <el-aside style="width: 130px; background-color: rgb(238, 241, 246)">
      <el-radio-group
        style="display: flex; flex-direction: column"
        v-model="drawMode"
      >
        <el-radio-button label="pen"></el-radio-button>
        <el-radio-button label="eraser"></el-radio-button>
      </el-radio-group>

      <el-input-number
        v-model="pageIndex"
        :min="1"
        style="width: 130px"
      ></el-input-number>
    </el-aside>

    <el-container>
      <el-main style="padding: 0px">
        <canvas id="canvas" style="height: 100%; width: 100%" />
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
console.log(import.meta.env.VITE_WS_HOST);
console.log(import.meta.env.VITE_HTTP_HOST);

let canvas;
var ws;

function connectWS() {
  ws = new WebSocket(import.meta.env.VITE_WS_HOST + "/api/ws");
  ws.onmessage = function (msg) {
    console.log("ws onmessage");
    let receivedJson = JSON.parse(msg.data);

    let route = receivedJson["route"];

    if (route == "stroke-create") {
      let receivedData = receivedJson["data"];

      let oldCanvasJson = canvas.toJSON();
      oldCanvasJson["objects"].push(receivedData);

      canvas.loadFromJSON(JSON.stringify(oldCanvasJson));
    } else if (route == "strokes-delete") {
      let deleteIdArray = receivedJson["id"];

      let canvasJson = canvas.toJSON();
      // console.log(canvasJson);

      canvasJson["objects"] = canvasJson["objects"].filter(
        (obj) => deleteIdArray.indexOf(obj["startTime"]) == -1
      );

      canvas.loadFromJSON(JSON.stringify(canvasJson));

      // console.log(deleteIdArray);
      // console.log(canvasJson);
    } else if (route == "strokes-set") {
      let canvasJson = canvas.toJSON();
      canvasJson["objects"] = receivedJson["data"];
      canvas.loadFromJSON(JSON.stringify(canvasJson));
    }
  };

  ws.onopen = function (event) {
    console.log("WebSocket is ready now.");

    var json = JSON.stringify({
      route: "change-page-index",
      boardId: "1",
      pageId: 1,
    });
    ws.send(json);
  };

  ws.onclose = function (event) {
    console.log("WebSocket is closed now.");
    connectWS();
  };
}
connectWS();

export default {
  mounted() {
    let self = this;

    canvas = new fabric.Canvas("canvas", {
      isDrawingMode: true,
      skipTargetFind: true,
      selectable: false,
      selection: false,
    });

    resizeCanvas(); // resize canvas at init
    window.addEventListener("resize", resizeCanvas, false);

    function resizeCanvas() {
      canvas.setWidth(window.innerWidth - 130);
      canvas.setHeight(window.innerHeight);
    }

    canvas.on("selection:created", function (e) {
      // 选中事件
      // 清除所有选中的笔迹
      let removeIdList = new Array();
      if (e.target._objects) {
        //多选删除
        e.target._objects.forEach((element) => {
          removeIdList.push(element["startTime"]);
          canvas.remove(element);
        });
      } else {
        //单选删除
        removeIdList.push(e.target["startTime"]);
        canvas.remove(e.target);
      }
      canvas.discardActiveObject(); //清除选中框

      var json = JSON.stringify({ route: "strokes-delete", id: removeIdList });
      ws.send(json);
    });
    function canvas_up(options) {
      if (self.drawMode === "pen") {
        let objects = canvas.toJSON()["objects"];
        let lastObject = objects[objects.length - 1];
        var json = JSON.stringify({ route: "stroke-create", data: lastObject });
        // console.log(json);
        ws.send(json);
      }
    }
    canvas.on("mouse:up", canvas_up);
    canvas.on("touch:up", canvas_up);

    // https://www.npmjs.com/package/@arch-inc/fabricjs-psbrush 压感笔刷
    let brush = new fabric.PSBrush(canvas);
    brush.width = 3;
    brush.color = "#000";
    canvas.freeDrawingBrush = brush;
  },

  data() {
    return {
      drawMode: "pen",
      pageIndex: 1,
    };
  },

  watch: {
    drawMode: {
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
html,
body,
#app,
.el-container {
  padding: 0px;
  margin: 0px;
  height: 100%;
}
canvas {
  display: block;
}
</style>
