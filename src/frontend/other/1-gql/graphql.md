---
description: A query language for your API.
---

# GraphQL

## 查询和变更

GraphQL是一个用于API的查询语言, 是一个使用基于类型系统来执行查询的服务端运行时(类型系统由你的数据定义)。

```javascript
# 查询mutation
query HeroRes($count: Int=3) {  // 查询名(传入参数: 类型=默认值), 无参数时: HeroRes
    hero(id: "10") {  // 字段(参数: 值)
        name  // 次级字段
        friends(count: $count) {  // 使用变量
            name(age: "10")  // 每个字段都可以有参数
        }
    }
    myHero: hero(id: '4') {  // 别名, 返回的结果中使用myHero替换hero
        ...commFields  // 引用片段
    }
    youHero: hero(id: '5') {
        __typename  // 获取元字段
        ... on HeroObj {  // 内联片段, 在片段中添加新字段
            age
        }
    }
}

fragment commFields on HeroObj {  // 片段, 提取公共字段, HeroObj是一个GraphQL对象
    name
    frineds(count: $count) {
        name
    }
}

# 变更mutation
mutation AddCount($name: String!, $isAdmin: Boolean ) {  // !表示非空
    hero(name: $name, other: 'xx') {  // 变更内容
        name  // 变更后要查询的内容
        open @include(if: $isAdmin) {  // 指令: @include(if: Boolean) 仅在参数为true时, 包含此字段
            light @skip(if: $isAdmin)  // 指令: @skip(if: Boolean) 如果参数为true, 跳过此字段
        }
    }
}

# 订阅subscription
subscription Test {...}
```

## Schema和类型

GraphQL schema它定义了字段的类型、数据的结构，描述了接口数据请求的规则。 其使用一个简单的强类型模式语法，称为模式描述语言(Schema Definition Language, SDL)。

```javascript
// GraphQL对象类型
type Starship {  // 定义一个对象
    id: ID!  // 字段, ! 表示必须字段
    name: String!
    appearsIn: [Float!]!  // 数组类型
    length(unit: LenUnit = METER): Float  // 带参数字段 
}

// 默认的标量类型
Int: 有符号 32 位整数.
Float: 有符号双精度浮点值.
String: UTF‐8字符序列.
Boolean：true或false.
ID: 一个唯一标识符, 通常用以重新获取对象或者作为缓存中的键.

// 枚举类型
enum Episode {
    NEWHOPE
    EMPIRE
    JEDI
}

// 接口
interface People {
    id: ID!
    name: String!
}
type Student implement People {  // 实现接口
    id: ID!
    name: String!
    friends: [People]
}

// 联合类型
union SearchResult = Human | Droid | Starship

// 输入类型, 用于mutation中
input ReviewInput {
    stars: Int!
    commentary: String
}
mutation CreateReview($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {  // 修改数据, 并按需返回结果
    commentary
  }
}

// schema定义查询入口
schema {
    query: Starship  // 必须有一个query类型
    mutation: Register  // mutation类型按需提供
}
```
