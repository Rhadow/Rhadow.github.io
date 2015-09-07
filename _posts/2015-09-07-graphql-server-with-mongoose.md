---
layout     : post
title      : "如何實作 GraphQL API"
subtitle   : "使用 GraphQL 與 Mongoose 建立一個簡易的 Node 伺服器並提供基本 API 功能"
date       : 2015-09-07
author     : "Rhadow"
tags       : GraphQL NodeJS MongoDB
comments   : true
signature  : true
---

Facebook 在最近正式提出了 GraphQL 的規格與相對應的實作。GraphQL 的出現主要是想降低 RESTful API 在取得複雜資料時 query 的次數並同時保有 RESTful 那樣簡單與後端溝通的方式。舉一個在 [RisingStack 文章](https://blog.risingstack.com/graphql-overview-getting-started-with-graphql-and-nodejs/) 中提到的一個例子: 當開發者需要取得使用者與使用者的朋友們的資訊時，傳統的 RESTful 會用以下方式取得資料:

`GET /users/1 and GET /users/1/friends  `

或

`GET /users/1?include=friends.name`

換成使用 GraphQL 時，取得同樣資料的方法如下:

```js
{
    user(id: 1) {
        name
        age
        friends {
            name
        }
    }
}
```

對開發者而言，graphQL 的方法更為直觀也更具有彈性。這也是本篇主要探討的主題: 如何實作 GraphQL API。
本篇範例的原始碼: [graphQL-relay-example](https://github.com/Rhadow/graphQL-relay-example)。

## Server

這次要實作的 GraphQL API 是以 express 框架建立的 Node 伺服器。Database 是使用 MongoDB 搭配 [Mongoose](http://mongoosejs.com/)，而 GraphQL 部分則是使用 Facebook 提供的 [graphql-js](https://github.com/graphql/graphql-js)。

老實說，GraphQL 的 server 部分其實非常簡單，直接來看看程式碼:

```js
// server.js

// Express
import express from 'express';
import bodyParser from 'body-parser';

// Mongoose
import mongoose from 'mongoose';

// GraphQL
import schema from '../schema/schema.js';
import { graphql } from 'graphql';

const app = express();
const PORT = 3000;
const DB_PATH = 'mongodb://localhost:27017/test';

// Connet to DB
mongoose.connect(DB_PATH);

app.use(bodyParser.text({
	type: 'application/graphql'
}));

app.post('/graphql', (req, res) => {
	graphql(schema, req.body).then((result) => {
		res.send(JSON.stringify(result, null, 2));
	});
});

const server = app.listen(PORT, () => {
	const host = server.address().address;
	const port = server.address().port;
	console.log(`Graphql is listening at https://${host}:${port}`);
});

```

是不是非常的直接，與平常不同的地方在於 graphql 與 schema。graphql 只是把 request 的內容與 schema 做比對並將結果送回，而 schema 呢，則是這次 graphql 的重點。

## Schema

我們先從[這篇文章](https://medium.com/@clayallsopp/your-first-graphql-server-3c766ab4f0a2)中借一個簡單的例子來看如何建立 schema:

```js
// schema.js

import {
    GraphQLObjectType,
    GraphQLSchema,
    GraphQLInt
} from 'graphql/lib/type';

let count = 0;

let schema = new GraphQLSchema({
    query: new GraphQLObjectType({
        name: 'RootQueryType',
        fields: {
            count: {
                type: GraphQLInt,
                resolve: function() {
                    return count;
                }
            }
        }
    })
});

export default schema;
```

我們用了一個蠻複雜的配置建立了一個 schema 的實體，讓我們來仔細看看這複雜的配置是怎麼回事。


首先是 `query`: 在我們與 GraphQL 溝通時，都需要跟 graphQL 說我們要做什麼事(例如: 取資料，修改資料等)。而 `query` 底下的內容就是讓我們設定當取資料的動作發生時，GraphQL 可以提供那些 API。以上例來說，如果它變成 RESTful 的話，會是 `GET /count`。

我們可以透過在 `fields` 中增加其他屬性來提供更多的 API 出去。每個 `fields` 的屬性都至少要提供它會回傳的型別 (`type`) 與如何回傳實際需要的值的函式 (`resolve`)。

在 GraphQL 的世界裡，型別很重要。因此 graphql-js 提供了各式各樣的型別供我們使用。我們透過 `GraphQLObjectType` 定義了 `query` 也使用了 `GraphQLInt` 來告知 count 回傳的值將會是個 `Int`。

接著讓我們來試著使用這個 API:

如果是用 Postman 的話，需要 POST 以下內容到 `/graphql` 並將 Content-Type 設為 `application/graphql` 即可:

```
query RootQueryType {
	count
}

```

用指令的話:

```
curl -XPOST -H "Content-Type:application/graphql"  -d 'query RootQueryType { count }' http://localhost:3000/graphql
```

仔細分析內容其實不難發現與 GraphQL 的溝通模式，先說明要做什麼動作再提供需要的資料格式就大功告成囉。名字部分(`RootQueryType`)目前是有點問題，經過實驗就算隨便亂打也能正常取得資料，但不給的話會跑出語法錯誤。這部分可能要看未來官方會如何改善。


GraphQL 還有提供一個特別的設計，就是當你不指定方法時，它會自動使用 `query` 模式。因此，用以下方式也能取得 `count` 的值!

```
{
    count
}
```

## 自訂型別

剛有提到 GraphQL 的型別很重要，除了官方提供的固定型別外，我們也能夠自訂型別。下面將示範要如何自訂:

```js
// UserType.js

import {
	GraphQLObjectType,
	GraphQLList,
	GraphQLString,
	GraphQLInt,
	GraphQLID
} from 'graphql';

import ProductType from '../Product/ProductType.js';

let UserType = new GraphQLObjectType({
	name: 'User',
	description: 'A user',
	fields: () => ({
		_id: {
			type: GraphQLID,
			description: 'user id'
		},
		name: {
			type: GraphQLString,
			description: 'user name'
		},
		age: {
			type: GraphQLInt,
			description: 'user age'
		},
		shoppingList: {
			type: new GraphQLList(ProductType),
			description: 'List of products that user bought'
		}
	})
});

export default UserType;
```

在看完了剛剛複雜的 schema 後，這個就相對簡單了吧。上面的程式碼定義了一個 `UserType` 型別。
這邊有用到一些剛沒提到的新型別，很直觀的例如 `GraphQLString` 就先省略不說明了。

* GraphQLID: 專門給 ID 用的型別，可以是 `Int` 或是 `String`
* GraphQLList: 其實就是 `Array`，參數則是 array 裡值的型別。在上例中用了另一個自訂型別 ProductType。

自訂型別使開發者不再受限於基本型別並能提供回傳更複雜的資料結構 API。

## 修改資料

說了這麼多取值，那修改呢? 首先我們必須先回去看看 Schema。別忘了所有的動作都是從它開始的。

```js
// schema.js

const schema = new GraphQLSchema({
	query: new GraphQLObjectType({
		...省略
	}),
	mutation: new GraphQLObjectType({
		name: 'RootMutationType',
		fields: {
			addUser: {
				type: UserType,
				description: 'Add a new user to list',
				args: {
					name: {
						type: new GraphQLNonNull(GraphQLString),
						description: 'New user\'s name'
					},
					age: {
						type: new GraphQLNonNull(GraphQLInt),
						description: 'New user\s age'
					},
					shoppingList: {
						type: new GraphQLNonNull(new GraphQLList(GraphQLID)),
						description: 'New user\s shopping list'
					}
				},
				resolve: (root, { name, age, shoppingList }) => {
					let newUser = new User();
					newUser.name = name;
					newUser.age = age;
					newUser.shoppingList = shoppingList;
					return newUser.save();
				}
			},
		}
	})
});
```

哇，怎麼又這麼多行。別緊張，仔細研究的話會發現其實與 `query` 大同小異。不一樣的地方在於我們新增了一個 `mutation` 屬性，任何關於修改資料的 API 都請放在 `mutation` 底下。其實你如果天生反骨要放在 `query` 下面也不是不行，但何必這樣搞自己呢?

在這裡我們定義了一個 `addUser` API 用來新增用戶。接著我們快速過一下裡面的屬性:

* type: 與 `query` 介紹時相同，不解釋
* description: 關於這個 API 的描述，在做 [Introspection](https://github.com/facebook/graphql/blob/master/spec/Section%204%20--%20Introspection.md) 時會有幫助，可有可無的屬性，但為求完整性建議加上。
* args: 提供給這個 API 的參數，每個參數都需要指定型別。上例中使用到的 `GraphQLNonNull` 會將參數設為無法為 `null` 的參數，換句話說就是必填。`args` 也可以使用在 `query` 的 API 中。
* resolve: 功能與在 `query` 段落時提到的相同。需要補充的是，回傳的值可以是一個 promise。

接著來看看如何使用這支新的 API 吧!

Postman:

```js
mutation RootMutationType {
    addUser(name:"Tester", age: 999, shoppingList: []) {
        name
    }
}
```

指令:

```js
curl -XPOST -H 'Content-Type:application/graphql'  -d ‘mutation RootMutationType { addUser(name:"Tester", age: 999, shoppingList: []) { name } }' http://localhost:3000/graphql
```

使用方法上與 `query` 相同，不過這次就無法省略方法與名字了。

## 與 Database 連結

其實 GraphQL 與 Database 的連結並沒有太深，GraphQL 只會將 `resolve` 內 return 的資料根據開發者要求的格式回傳回去，至於要如何生成這些資料，其實是由 Database 或是其他的邏輯來處理，與 GraphQL 沒有什麼關係。

舉例來說，一個使用者的購物車裡有多個商品，開發者想要取得使用者的姓名與他購買商品的名稱。

使用者範例資料:

```js
// Example user
{
    _id: 'HOWARDS ID',
    name: 'Howard',
    age: 27,
    shoppingList: ['PRODUCT 1 ID']
}
```

商品範例資料:

```js
// Example product
{
    _id: 'PRODUCT 1 ID',
    name: 'MacBook Pro',
    category: ['electronics', 'apple']
}
```

當開發者用下列 `query` 取值時:

```js
{
    user(id: "HOWARDS ID") {
        name,
        shoppingList {
            name
        }
    }
}
```

user API 內的 `resolve` 要回傳的值會是:

```js
{
	_id: 'HOWARDS ID',
    name: 'Howard',
    age: 27,
    shoppingList: [
        {
		    _id: 'PRODUCT 1 ID',
		    name: 'MacBook Pro',
		    category: ['electronics', 'apple']
		}
    ]
}
```

大家可能會認為 GraphQL 這麼有彈性，從 DB 取資料後又要依照 query 的格式輸出一定很麻煩。但事實上是，我們只要把完整的資料從 DB 都取好把它們組起來，再由 GraphQL 依照 query 的格式回傳給使用者就完成了。以上例來說，雖然 query 內並沒有要求要回傳使用者的年齡也沒有要求要回傳商品的種類，但是資料還是完整的被取回來以對應未來的變動。

## 總結

GraphQL 帶給了前端更有彈性的 API，也提供開發者們除了 RESTful 之外的另一種方式來與後端溝通。個人認為 GraphQL 把一部分前端的工作量轉移到後端了，因此適當的去了解後端架構並學習才能讓自己能夠在 GraphQL 與 RESTful 中使用最適合解決問題的方法。想要看詳細程式碼內容請到: [graphQL-relay-example](https://github.com/Rhadow/graphQL-relay-example) 查看。