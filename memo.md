# メモ

[Fullstack app with TypeScript, Next\.js, Prisma & GraphQL](https://www.prisma.io/blog/fullstack-nextjs-graphql-prisma-oklidw1rhw)

## 2022-08-22 02:08:57
Part2の途中までしか書かれていなかったときに読んだことがある。
再び見るとかなり加筆されていたのでまたやって見る。

ローカルでの開発ではDBはDockerで管理したい。

[postgres \- Official Image \| Docker Hub](https://hub.docker.com/_/postgres)

Postgresコンテナの環境変数を docker-compose.yml に

```yml
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: awesome-links
```

で指定して、`.env` に

```
DATABASE_URL="postgresql://user:password@localhost:5432/awesome-links"
```
を書くと Prisma が読み取って接続してくれる。

## 2022-08-22 02:16:13

指示に従うと以下のエラーを得る。

```
$ npx prisma db seed --preview-feature
prisma:warn Prisma "db seed" was in Preview and is now Generally Available.
You can now remove the --preview-feature flag.
prisma:warn The "ts-node" script in the package.json is not used anymore since version 3.0 and can now be removed.
Environment variables loaded from .env
prisma:warn The "ts-node" script in the package.json is not used anymore since version 3.0 and can now be removed.
Error: To configure seeding in your project you need to add a "prisma.seed" property in your package.json with the command to execute it:

1. Open the package.json of your project
2. Add the following example to it:
```
"prisma": {
  "seed": "ts-node prisma/seed.ts"
}
```
If you are using ESM (ECMAScript modules):
```
"prisma": {
  "seed": "node --loader ts-node/esm prisma/seed.ts"
}
```

3. Install the required dependencies by running:
npm i -D ts-node typescript @types/node

More information in our documentation:
https://pris.ly/d/seeding
```

- `seed` コマンドはもはや preview featureではない。
- package.json に `prisma` プロパティが必要。
- `ts-node` のコンパイラオプションは今も必要。

従って、package.jsonに

```
"prisma": {
   "seed": "ts-node --compiler-options {\"module\":\"CommonJS\"} prisma/seed.ts"
},
```

を加えると良い。

```
$ npx prisma db seed
Environment variables loaded from .env
Running seed command `ts-node --transpile-only --compiler-options {"module":"CommonJS"}  prisma/seed.ts` ...

🌱  The seed command has been executed.
```

参考：[Seeding your database \| Prisma Docs](https://www.prisma.io/docs/guides/database/seed-database)

## 2022-08-22 02:40:37
```
npx prisma studio
```

でデフォルトlocalhost:5050にGUIが開く。

タイムゾーンが設定できていない。

## 2022-08-25 20:30:24

- [graphql \- npm](https://www.npmjs.com/package/graphql)
- [apollo\-server\-micro \- npm](https://www.npmjs.com/package/apollo-server-micro)
- [micro\-cors \- npm](https://www.npmjs.com/package/micro-cors)

tagged template [Template literals \(Template strings\) \- JavaScript \| MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals#tagged_templates)

`/graphql/schema.ts` で定義した `Query` の型通りに `/graphql/resolvers.ts` に関数を定義する。

> Body parsing is disabled here since it is handled by GraphQL.

？

DBのコネクションを節約するためローカルでは `global` オブジェクトに `Prisma Client` のインスタンスをくっつける。

[Best practice for instantiating PrismaClient with Next\.js \| Prisma Docs](https://www.prisma.io/docs/guides/database/troubleshooting-orm/help-articles/nextjs-prisma-client-dev-practices)


## 2022-08-25 21:03:37

SDL (Schema Definition Langauge).

```ts
export const schema = makeSchema({
  types: [], //ここにgraphqlで扱うオブジェクトの型をリストする
  outputs: {
    typegen: join(process.cwd(), 'node_modules', '@types', 'nexus-typegen', 'index.d.ts'), //型ファイルを吐き出す場所
    schema: join(process.cwd(), 'graphql', 'schema.graphql'), //スキーマを吐き出す場所
  },
  contextType: {
    export: 'Context',
    module: join(process.cwd(), 'graphql', 'context.ts'), //ここで定義された `Context` 型を使うという宣言
  },
})
```

GraphQLで扱うオブジェクトの型を（TSの）オブジェクトとして定義する。頭文字は大文字だがTSの型ではない。
```ts
// /graphql/types/Link.ts
import { objectType, extendType } from 'nexus'
import { User } from './User'

export const Link = objectType({
  name: 'Link',
  definition(t) {
    t.string('id')
    t.string('title')
    t.string('url')
    t.string('description')
    t.string('imageUrl')
    t.string('category')
    t.list.field('users', {
      type: User,
      async resolve(_parent, _args, ctx) {
        return await ctx.prisma.link
          .findUnique({
            where: {
              id: _parent.id,
            },
          })
          .users()
      },
    })
  },
})
```

## 2022-08-25 23:19:08

[GraphQL Cursor Connections Specification](https://relay.dev/graphql/connections.htm)

## 2022-08-30 00:20:26

```ts
            orderBy: {
              index: 'asc',
            },
```

```ts
const AllLinksQuery = gql`
  query allLinksQuery($first: Int, $after: String) {
    links(first: $first, after: $after) {
      pageInfo {
        endCursor
        hasNextPage
      }
      edges {
        cursor
        node {
          index
          imageUrl
          url
          title
          category
          description
          id
        }
      }
    }
  }
`;
```

どちらも `index` となっている場所は`id` が正しい。

`/api/graphql`　に送られるメソッドは `POST` ？ デバッグ方法がよく分からない。 `POST http://localhost:3000/api/graphql 400 (Bad Request)` とエラーメッセージが出た。

apolloで見ると存在しないフィールド？に赤線が引かれるので分かる。

`schema.graphql` はコミットしていいのだろうか。