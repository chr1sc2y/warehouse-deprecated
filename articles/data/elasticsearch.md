### http 操作

- 查看所有资源

  `ip:port/_cat`

- 查看所有 Index

  `ip:port/_cat/indices?v`

#### Index 操作

- 新建 Index

  - `PUT ip:port/<index_name>`
  - response `{"acknowledged":true,"shards_acknowledged":true,"index":"test_index"}`
- 删除 index
  - `DELETE ip:port/<index_name>`
  - response `{"acknowledged":true}`

#### 数据操作

- 新增记录
  - `PUT ip:port/<index_name>/_doc/<_id> --header 'Content-Type: application/json' --data {...}`
- 查询记录
  - `GET ip:port/<index_name>/_doc/<_id>`

