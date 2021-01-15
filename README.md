# share_board_frondend

## 介绍

基于 Websocket 的 同步 Web 手写板

## Todo

- 翻页

- OSS 部署

- Websocket 连接指示,自动重连

- 将技术栈换为 VUE

- 颜色改变

## 传输数据格式

```json
{
    "type":"add", // add -> 增加   replace -> 替换
    "data":{}
}
```

## 致谢

前端基于 <https://github.com/vipstone/drawingboard>
