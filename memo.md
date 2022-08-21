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