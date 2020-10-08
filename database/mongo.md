# mongodb

## 手册阅读笔记

概念：

- database
- collection，对应 rdbms 的 table
- document，对应 rdbms 的 row
- views，支持只读视图
- On-Demand Materialized Views

```json
{
    "_id" : <value>,
    "name" : { "first" : <string>, "last" : <string> },       // embedded document
    "birth" : <ISODate>,
    "death" : <ISODate>,
    "contribs" : [ <string>, ... ],                           // Array of Strings
    "awards" : [
        { "award" : <string>, year: <number>, by: <string> }  // Array of embedded documents
    ]
}
```

```bash
use myDB
# 会自动创建 collection和上下文的 db
db.myNewCollection1.insertOne( { x: 1 } )

```

查询

```bash
# query 查询操作符
# projection 投影，选择指定的字段
db.collection.find(query, projection)

# 查询 _id=5 的 doc
db.bios.find( { _id: 5 } )
# 查询 last 嵌入文档 值为 Hopper
db.bios.find( { "name.last": "Hopper" } )

# 使用操作符查询
# 查询 gty>4 的 doc
db.collection.find( { qty: { $gt: 4 } } )
# 查询 _id 在数组里面的
db.bios.find(
   { _id: { $in: [ 5, ObjectId("507c35dd8fada716c89d0013") ] } }
)
# 查询结果按照 name 进行排序
db.bios.find().sort( { name: 1 } )
# 限制查询出的记录数
db.bios.find().limit( 5 )
db.bios.find().skip( 5 )

# cursor handling 游标，默认查询 20 条数据
# session
# transaction 事务
```

更新

```bash
# filter/update/options
# 找到第一个 item=paper 的 doc，然后更新 size.uom 为 cm，status 为 p
# 更新 lastModified 字段为当前日期，如果该字段不存在，则创建字段
db.inventory.updateOne(
   { item: "paper" },
   {
     $set: { "size.uom": "cm", status: "P" },
     $currentDate: { lastModified: true }
   }
)

## 替换
# _id 是不可变(immutable)的，不会被替换
db.inventory.replaceOne(
   { item: "paper" },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 40 } ] }
)
#  删除字段
db.getCollection('answer_001001').updateMany({}, {$unset: {lastUpdate:""}})

```

索引

```bash
# 使用通配符对多个字段创建 text 索引
# 创建通配符索引
# This index allows for text search on all fields with string content
db.collection.createIndex( { "$**": "text" } )

# 对 userMetadata 的 subField 创建通配符索引
db.userData.createIndex( { "userMetadata.$**" : 1 } )
# 对 subField 进行 like 查询
db.userData.find({ "userMetadata.likes" : "dogs" })
db.userData.find({ "userMetadata.dislikes" : "pickles" })
db.userData.find({ "userMetadata.age" : { $gt : 30 } })
db.userData.find({ "userMetadata" : "inactive" })
# 对多个字段创建通配符索引
# 对所有字段创建通配符索引排除指定字段


db.answer_001001.find( { $text: { $search: "java coffee shop" } } )
# Wildcard Text Indexes are distinct from Wildcard Indexes
# 通配符文本索引和通配符索引是不同的，
# 通配符文本索引，可以使用 $text 操作符，只能这种方式创建通配符文本索引
# 我对 17000 条左右答案创建索引，大概花了 3.87 s
db.answer_001001.createIndex( { "$**": "text" } )
# 查询包含 java 或 coffee 或 shop 的
db.answer_001001.find( { $text: { $search: "java coffee shop" } } )
# 查询包含 “coffee shop” 的
db.answer_001001.find( { $text: { $search: "\"coffee shop\"" } } )
# 查询包含 java 或 shop 但不包含 coffee 的
db.answer_001001.find( { $text: { $search: "java shop -coffee" } } )
# 可以按照相关度排序 .sort( { score: { $meta: "textScore" } } )


# 通配符索引
db.collection.createIndex( { "$**": 1 } )
## 可以对子字段创建通配符索引
db.answer_001001.createIndex({"answer.$**": 1"})
```

## FAQ

1. text 索引不支持中文分词

目前社区版的[search language](https://docs.mongodb.com/manual/reference/text-search-languages/#text-search-languages)是不支持中文分词的，所以我考虑是不是可以先将[分词](https://github.com/yanyiwu/gojieba)分好之后，然后再放到其中的一个 field 里面，然后再对这个 field 创建 text index

## 随手记

mongo [可以从 ObjectId 来获取 timestamp](https://docs.mongodb.com/manual/reference/method/ObjectId.getTimestamp/#example)，所以没必要再创建一个 create_at Field

## 参考

[mongo manual](https://docs.mongodb.com/manual/introduction/)
