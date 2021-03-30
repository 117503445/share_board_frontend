<template>
  <el-container style="height: 100%; border: 1px solid #eee">
    <el-aside style="width: 130px; background-color: rgb(238, 241, 246)">
      <el-radio-group
        style="display: flex; flex-direction: column"
        v-model="drawMode"
      >
        <el-radio-button label="pen"></el-radio-button>
        <el-radio-button label="eraser"></el-radio-button>
      </el-radio-group>

      <el-input-number v-model="pageIndex" :min="1" style="width: 130px;"></el-input-number>
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
var ws;

function connectWS() {
  let HOST_WS = "wss://shareboard.goldhome.117503445.top:18888";
  let HOST_HTTP = "https://shareboard.goldhome.117503445.top:18888";

  // HOST_WS = "ws://localhost";
  // HOST_HTTP = "http://localhost";

  ws = new WebSocket(HOST_WS + "/api/ws");
  ws.onmessage = function (msg) {
    console.log("ws onmessage");
    console.log(msg.data.length);
    console.log(msg.data);
    let receivedJson = JSON.parse(msg.data);

    let updateType = receivedJson["type"];
    let receivedData = receivedJson["data"];

    if (updateType == "replace") {
      console.log("replace");
      canvas.loadFromJSON(JSON.stringify(receivedData));
    } else if (updateType == "add") {
      console.log("add");
      let oldCanvasJson = canvas.toJSON();
      oldCanvasJson["objects"].push(receivedData);
      canvas.loadFromJSON(JSON.stringify(oldCanvasJson));
    } else if (updateType == "status") {
      console.log("status");
      canvas.loadFromJSON(JSON.stringify(receivedData));
    } else {
      console.log(updateType + " ???");
    }
  };

  ws.onopen = function (event) {
    console.log("WebSocket is ready now.");

    // const Http = new XMLHttpRequest();
    // Http.open("GET", HOST_HTTP + "/api/v1/board/1");
    // Http.send();

    // Http.onreadystatechange = (e) => {
    //     let js = Http.responseText;
    //     //console.log(js);
    //     canvas.loadFromJSON(js);
    // }
    var json = JSON.stringify({
      type: "status",
      data: { boardid: "1", pagenumber: 1 },
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
    function canvas_up(options) {
      if (this.drawMode === "eraser") {
        var json = JSON.stringify({ type: "replace", data: canvas.toJSON() });
        ws.send(json);
      } else {
        let objects = canvas.toJSON()["objects"];
        let lastObject = objects[objects.length - 1];
        var json = JSON.stringify({ type: "add", data: lastObject });
        console.log(json);
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
</style>
