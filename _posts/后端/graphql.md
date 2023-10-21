## 基本介绍
一个 GraphQL 服务是通过定义类型和类型上的字段来创建的，然后给每个类型上的每个字段提供解析函数。在这些操作中可以用自定义的类型来描述参数
或响应的结构，GraphQl 会保证数据结构的一致。

GraphQl 是灵活的，定义了数据类型后，可以自己编写类似 sql 的语句，去查询自己想要的数据。

在 GraphQl 中，操作类型有 `query`, `mutation` 和 `subscription`。分别是查询，操作数据库和订阅数据变化。

GraphQl 服务启动后，可以访问本地的 Graphiql 网站，相当于 swagger，可以查看定义了的类型和接口。

## 查询
假如定义了一个类型 hero，并在后端提供了对应的解析函数。

```
# 类型定义
# 这里省去了解析函数
type hero {
  name
}
```

可以通过 `query` 关键字查询 hero 数据。当查询不需要参数时，`query` 也可以省略，仅用大括号。

```
query {
 hero {
   name
 }
}
```

### 定义带参数的查询
```
query HeroNameAndFriends {
  human(id: "1000") {
    name
    height(unit: FOOT)
  }
}

# 可以忽略简写操作类型 query 和操作名称 HeroNameAndFriends
{
  # 传参
  human(id: "1000") {
    name
    # 将 height 转为英尺作为单位
    height(unit: FOOT)
  }
}
```

### 别名
在一次查询中，需要对一个字段做多次查询就需要用到别名。

```
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
{
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}
```

## 变量
使用变量实现动态参数，而不是固定参数。

使用前需要做：
1. 使用 $variableName 替代查询中的静态值。
2. 声明 $variableName 为查询接受的变量之一。
3. 将变量以 {variableName: value} 放入 variables 中传入。

变量定义必须是 $ 为前缀，后跟其类型。下面例子的类型是 Episode。

所有声明的变量都必须是标量、枚举型或者输入对象类型。

变量定义可以是可选的或者必要的。如 Episode 后没有 !，因此就是可选的。

```
# 传入的对象 { "graphiql": true, "variables": { "episode": JEDI } }
query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

### 默认变量
可以通过在查询中的类型定义后面附带默认值的方式，将默认值赋给变量。

```
query HeroNameAndFriends($episode: Episode = "JEDI") {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

## 片段
片段方便复用。片段的语法是以 `fragment` 开头，紧跟 fragment 的名字，接着是 `on` 关键字，然后是 fragment 关联的类型。接着是花括号，花括号内
是关联的类型里我们需要查询的字段。

```
{
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  appearsIn
  friends {
    name
  }
}
```

### 片段内使用变量

```
query HeroComparison($first: Int = 3) {
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  friendsConnection(first: $first) {
    totalCount
    edges {
      node {
        name
      }
    }
  }
}
```

### 内联片段
下面第一个片段标注为 `... on Droid`，primaryFunction 仅在 hero 返回的 Character 为 Droid 类型时才会执行。同理适用于 Human 类型的 height 字段。

```
{
  "ep": "JEDI"
}

query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    ... on Droid {
      primaryFunction
    }
    ... on Human {
      height
    }
  }
}

{
  "data": {
    "hero": {
      "name": "R2-D2",
      "primaryFunction": "Astromech"
    }
  }
}
```

## 指令
使用变量动态地改变我们查询的结构。

GraphQL 的核心规范包含两个指令，其必须被任何规范兼容的 GraphQL 服务器实现所支持：
* @include(if: Boolean) 仅在参数为 true 时，包含此字段。
* @skip(if: Boolean) 如果参数为 true，跳过此字段。

服务端实现也可以定义新的指令来添加新的特性。

```
# 变量
{
  "episode": "JEDI",
  "withFriends": false
}

query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}

# withFriends = false
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}

# withFriends = true
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

## 变更
用于更新数据。

一个请求中的多个变更是按顺序执行。

```
# 变量
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}

mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

## 元字段（Meta fields）
GraphQL 允许你在查询的任何位置请求 __typename，一个元字段，以获得那个位置的对象类型名称。

下面的查询中，search 返回了一个联合类型，其可能是三种选项之一。没有 __typename 字段的情况下，几乎不可能在客户端分辨开这三个不同的类型。

GraphQL 服务提供了不少元字段，剩下的部分用于描述 introspection 系统。
```
{
  search(text: "an") {
    __typename
    ... on Human {
      name
    }
    ... on Droid {
      name
    }
    ... on Starship {
      name
    }
  }
}

{
  "data": {
    "search": [
      {
        "__typename": "Human",
        "name": "Han Solo"
      },
      {
        "__typename": "Human",
        "name": "Leia Organa"
      },
      {
        "__typename": "Starship",
        "name": "TIE Advanced x1"
      }
    ]
  }
}
```

## schema 和类型

```
type Character {
  name: String!
  appearsIn: [Episode!]!
}
```

* Character 是一个 GraphQL 对象类型，表示其是一个拥有一些字段的类型。你的 schema 中的大多数类型都会是对象类型。
* name 和 appearsIn 是 Character 类型上的字段。这意味着在一个操作 Character 类型的 GraphQL 查询中的任何部分，都只能出现 name 和 appearsIn 字段。
* String 是内置的标量类型之一 —— 标量类型是解析到单个标量对象的类型，无法在查询中对它进行次级选择。后面我们将细述标量类型。
* String! 表示这个字段是非空的，GraphQL 服务保证当你查询这个字段后总会给你返回一个值。在类型语言里面，我们用一个感叹号来表示这个特性。
* `[Episode!]!` 表示一个 Episode 数组。因为它也是非空的，所以当你查询 appearsIn 字段的时候，你也总能得到一个数组（零个或者多个元素）。
且由于 Episode! 也是非空的，你总是可以预期到数组中的每个项目都是一个 Episode 对象。

### 参数
GraphQL 对象类型上的每一个字段都可能有零个或者多个参数，例如下面的 length 字段：

```
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```
在 GraphQL 中，所有参数必须具名传递。本例中，length 字段定义了一个参数，unit。

参数可能是必选或者可选的，当一个参数是可选的，我们可以定义一个默认值 —— 如果 unit 参数没有传递，那么它将会被默认设置为 METER。

![img.png](/img/graph_build_schema.png)

![img_1.png](/img/graph_express.png)


## 基本参数类型
这些基本参数类型可以在 schema 声明中直接使用。

* Int：有符号 32 位整数。
* Float：有符号双精度浮点值。
* String：UTF‐8 字符序列。
* Boolean：true 或者 false。
* ID：ID 标量类型表示一个唯一标识符，通常用以重新获取对象或者作为缓存中的键。ID 类型使用和 String 一样的方式序列化；然而将其定义为 ID 意味着并不需要人类可读型。

另外，我们可以使用 \[类型\] 来表示一类数组，如：
* `[Int]` 表示整型数组；
* `[String]` 表示字符串型数组；

### 自定义返回类型

```
//...省略其他
const schema = buildSchema(`
    type Hero {
        name: String
        age: Int
        # doSomething 也可以传递参数
        doSomething(thing: String): String
    }
    type Query {
        getSuperHero(heroName: String!): Hero
    }
`)
const root = {
    getSuperHero: ({heroName}) => {
        // 这里的操作 实际开发中常常用在请求数据库
        const name = heroName
        const age = 18
        const doSomething = ({thing}) => {
            return `I'm ${name}, I'm ${thing} now`
        }
        return { name, age, doSomething }
    }
}
//...省略其他

```

使用：

```
// 查询
query {
	getSuperHero(heroName:"IronMan") {
        name
        age
        doSomething
	}
}

// 结果
{
    "data": {
        "getSuperHero": {
            "name": "IronMan",
            "age": 46,
            "doSomething": "I'm IronMan, I'm undefined now"
        }
    }
}
```

也可以给 doSomething 传递参数，就会获取到不同结果：

```
// 查询
query {
	getSuperHero(heroName:"IronMan") {
        name
        age
	    doSomething(thing:"watching TV")
	}
}

// 结果
{
    "data": {
        "getSuperHero": {
            "name": "IronMan",
            "age": 46,
            "doSomething": "I'm IronMan, I'm watching TV now"
        }
    }
}

```

## ConstructingTypes
在前面的介绍中，我们要创建一个 schema 都是使用 buildSchema 方法来定义，但我们也可以使用另外一种定义方式。
就是这里要学习使用的构造函数 graphql.GraphQLObjectType 定义，它有这么几个优点和缺点：

* 优点：报错提醒更直观，结构更清晰，更便于维护。
* 缺点：代码量上升。


定义相同的类型和实现相同参数的对比：

```javascript
const schema = buildSchema(`
  type Hero {
    name: String
    age: Int
  }
  type Query {
    getSuperHero(heroName: String!): Hero
  }
`)

const root = {
  getSuperHero: ({heroName}) => {
    const name = heroName;
    const age = 11;
    return {name, age};
  }
}
```

```
const graphql = require('graphql') // 需要引入
const QueryType = new graphql.GraphQLObjectType({
    name: 'Query', // 和 type Query 对应
    fields: {
        // 和 root 对应
        getSuperHero: {
            type: HeroType,
            args: {
                heroName: { type: graphql.GraphQLString }
            },
            // 方法实现 查询的处理函数
            resolve: function(_, { heroName }){
                const name = heroName
                const age = 18
                return { name, age }
            }
        }
    }
})
```

创建 schema，创建的时候只需实例化并且将参数传入即可：

```javascript
// step3 构造 schema
const schema = new graphql.GraphQLSchema({ query: QueryType})
```

使用：

```javascript
const app = express()

app.use('/graphql', graphqlHTTP({
    schema,
    graphiql: true
}))
app.listen(3000)
```

## 订阅
每当后台数据更改，都会推送数据过来。是基于 websocket 实现。

如：

```
# gql
subscription {
  online_users {
    id
    last_seen
    user {
      name
    }
  }
}
```

graphql-tag 解析字符串为 graphql 可识别语法。
