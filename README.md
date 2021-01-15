# share_board_frondend

## 介绍

基于 Websocket 的 同步 Web 手写板

## Todo

- 翻页

- 将技术栈换为 VUE

- 颜色改变

## 传输数据格式

```json
{
    "type":"add",
    "data":{}
}
```

type 取值

- add: data 为新添加的笔迹数据

- replace: data 为全部笔迹数据

- get: data 为空,服务器返回全部笔迹数据

## 致谢

前端基于 <https://github.com/vipstone/drawingboard>
