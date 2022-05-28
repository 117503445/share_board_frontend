# 系统设计

(偷懒，不调整 Markdown 格式了)

基于 Websocket 异步执行 

每个 stroke 的 id 都为 startTime, 是 13 位 的毫秒时间戳,可以认为每个 board 上的 stroke 都有不同的 startTime

Board id 为 UUID
Page id 为 正整数

Client -> Server 增加单个 stroke
{
    "route":"stroke-create",
    "data":{具体信息}
}

Client -> Server 删除多个 stroke
如果 无法找到 delete id 则忽略此 id
{
    "route":"strokes-delete",
    "id": [1617467159173,1617467159679]
}

Client -> Server 笔迹 清空
{
    "route":"strokes-clear"
}

Client -> Server 换页
{
    "route": "change-page-index",
    "boardId": "fd2a9415-d897-45b6-9672-8cca7274197a",
    "pageId": 2,
}

Server -> Client 笔迹 增 增加单个 stroke
{
    "route":"stroke-create",
    "data":{笔迹具体信息}
}

Server -> Client 笔迹 删 删除多个 stroke
{
    "route":"strokes-delete",
    "id": [1617467159173,1617467159679]
}

Server -> Client 笔迹 清空
{
    "route":"strokes-clear"
}

Server -> Client 当前页所有笔迹
成功切换到指定页
{
    "route": "strokes-set",
    "data": [{笔迹1},{笔迹2}],
    "code": 0,
    "msg": "ok"
}
如果待切换页未创建,且上一页的笔迹数为空,则切换失败
{
    "route": "strokes-set",
    "data": {},
    "code": 40000,
    "msg": "create new page failed."
}

储存方式 Redis 键值对
放在内存中 速度更快
主要操作为
    获取 某 page 所有笔迹
    增加 某 page 上 单个笔迹
    删除 某 page 上 多个笔迹
因此设计为
key: ${BoardId}-${PageId} (fd2a9415-d897-45b6-9672-8cca7274197a-1)
value: 
    {key: ${strokeId},value: ${stroke}}