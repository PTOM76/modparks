# note
以下はメモです。<br />

## Dockerfile
必要に応じて変更してください。

```dockerfile
FROM oven/bun:latest
RUN apt update && apt install -y \
    git \
    curl \
    unzip zip \
    vim nano \
    wget \
    && apt clean

# Git

# Change the following to your own Git user and email
# Gitユーザーとメールアドレスは自分のものに変更してください
ARG GIT_USER="EXAMPLE_UESR"
ARG GIT_EMAIL="example@example.com"
RUN git config --global user.name "$GIT_USER" \
    && git config --global user.email "$GIT_EMAIL"

COPY id_rsa /root/.ssh/id_rsa
COPY id_rsa.pub /root/.ssh/id_rsa.pub
RUN chmod 600 /root/.ssh/id_rsa && chmod 644 /root/.ssh/id_rsa.pub
# ----

WORKDIR /repo

CMD ["bash"]
```

## compose.yaml
必要に応じて変更してください。<br />
デフォルトではhostnameはuserにしています。

```yaml
services:
  app:
    build: .
    hostname: user
    tty: true
    volumes:
      - .\repo:/repo
    ports:
      - 3000:3000

```

## ホスト側の操作

ビルド & 起動
```sh
docker compose up --build -d
```

シェルに入る
```sh
docker compose exec app bash
```

削除
```sh
docker compose down
```

強制削除
```sh
docker-compose down --remove-orphans
```

## コンテナ内の操作

bunxの操作は以下の通りです。(おそらくbunxのxはexecuteのことでしょうね。)
- Enterキーで続行
- Spaceで項目の選択
- 矢印で項目の移動


0. ホスト側から `docker compose exec app bash` でシェルに入る
1. `bunx sv create (project-name)` でプロジェクトを作成 (ここではsv-devにした)
    1. templateはデフォルトの `SvelteKit minimal` を選択
    2. typeはデフォルトの `TypeScript` を選択
    3. libraryはとりあえず、`prettier` と `eslint` を選択
    4. package managerはデフォルトの `bun` を選択
    5. 待機します。(結構時間がかかりました)
2. `cd (project-name)` でプロジェクトに移動
3. 必要であればgitの初期化
4. vite.config.js の修正
```js
export default defineConfig({
	plugins: [sveltekit()],
    // この行を追加
	server: {
		host: "0.0.0.0",
		port: 3000
	}
    // ここまで
});
```
5. `bun add -D svelte-adapter-bun` でsvelte-adapter-bunをインストール
    - ちな、これはSvelteKitをBunで動かすためのアダプタらしいです。(これしないと警告が出る)
6. svelte.config.jsの修正
```js
//import adapter from '@sveltejs/adapter-auto'; // コメントアウトor削除
import adapter from "svelte-adapter-bun"; // 追加
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';
```
7. `bun run dev` で開発サーバーを起動 (Docker上なので--openは使えない)
8. ブラウザで `http://localhost:3000` にアクセスして、SvelteKitの初期画面が表示されれば成功
