---
title: "ビルド（Build）とは？RailsとDockerの仕組みから改めて理解する"
emoji: "🛠️"
type: "tech"
topics: ["rails", "docker", "build"]
published: false
---

## ビルドとは？

ビルドとは、ソースコードや関連ファイルを、アプリケーションの実行・配信に必要な形へ準備する処理のことです。

例えば、Webアプリケーションを作る場合、開発者は次のようなファイルを用意します。

- Rubyのコード
- JavaScript
- CSS
- 画像などの素材
- 設定ファイル

これらのうち、必要なものを変換・結合・最適化するなどして、実行や配信に適した状態にします。

ただし、ビルドの具体的な内容は、言語や開発環境によって異なります。

## ビルドを行う理由

ビルドを行う理由は、**開発時に用意したソースコードや関連ファイルを、実行・配信に適した状態にするため**です。

例えば、CSSやJavaScriptをまとめたり、不要な部分を削除してファイルサイズを小さくしたりします。

これにより、ブラウザへ効率よくファイルを配信できるようになります。

ただし、すべてのアプリケーションで同じ処理が必要になるわけではありません。

## ビルド・コンパイル・デプロイの違い

| 用語 | 役割 |
|---|---|
| ビルド | 実行・配信に必要な成果物を準備する一連の処理 |
| コンパイル | ソースコードを別の形式へ変換する処理 |
| デプロイ | アプリケーションを実行環境へ配置・反映する処理 |

例えば、コンパイルはビルド工程の一部になる場合があります。

つまり、**ビルド＝コンパイルではありません。**

また、ビルドによって成果物を準備しても、それだけでアプリケーションが本番環境に公開されるとは限りません。

本番環境への反映には、デプロイという別の工程があります。

## Railsにおけるビルド

Railsを例に考えてみます。

![Railsのビルド処理フロー](/images/Railsビルドの流れを図解.png)
*▲ Railsにおけるビルドの流れを簡略化した概念図。実際の処理内容や生成先は、使用するツールや設定によって異なります。*

Railsでは、CSSやJavaScriptなどのファイルを、ブラウザへ配信できる状態に準備する処理があります。

例えば、Tailwind CSSを使っている場合、HTMLやERBなどで使用しているクラスをもとに、必要なスタイルを含むCSSを生成します。

Railsで扱うファイルの例を整理すると、次のようになります。

| 対象 | 処理の例 |
|---|---|
| CSS | Tailwind CSSなどを使用してCSSを生成・最適化する |
| JavaScript | 使用するツールによっては、ファイルを結合・変換・最適化する |
| 画像・フォント | アセット管理の仕組みに応じて、配信用のパスやファイルを準備する |

ただし、すべてのRailsアプリで同じ処理が行われるわけではありません。

例えば、JavaScriptをImportmapで管理している場合、一般的なJavaScriptバンドラーによる結合処理は必須ではありません。

また、本番環境では、アセットを事前に準備するために次のコマンドを使用する場合があります。

```bash
bin/rails assets:precompile
```

これは、使用しているアセット管理の仕組みに応じて、本番配信用のファイルを準備する処理です。

一方、Rubyのコードは通常、C言語などと同じようにアプリ全体を事前に機械語へコンパイルして実行ファイルを作る必要はありません。

## Dockerのbuildとは？

Dockerで実行する、

```bash
docker compose build
```

もビルドですが、こちらは**Dockerイメージを作成する処理**です。

例えば、Dockerfileの指示に従って、必要なパッケージのインストールやファイルの配置などを行います。

Dockerイメージは、コンテナを作成・起動するための元となるものです。

そのため、イメージをビルドしただけでは、コンテナが起動するわけではありません。

コンテナを作成・起動する場合は、例えば次のコマンドを使用します。

```bash
docker compose up
```

また、次のように実行することで、必要なイメージをビルドしてからコンテナを起動できます。

```bash
docker compose up --build
```

ここまでのビルドを整理すると、次のようになります。

| 種類 | 作成・準備するもの |
|---|---|
| CSSのビルド | ブラウザで使うCSS |
| JavaScriptのビルド | 配信用のJavaScript |
| Railsのアセット準備 | 本番環境などで配信するアセット |
| Dockerのビルド | コンテナの元になるDockerイメージ |

同じ「ビルド」でも、**何を作成するのか、どのような処理を行うのかが異なります。**

## 自作アプリ「やまッチ」でのビルドの流れ

自作アプリ「やまッチ」では、Dockerを使用して開発環境を構築しています。

今回は、実際に使用している `Dockerfile.dev` を確認しながら、Dockerイメージがどのように作成されるのかを整理します。

### 1. Rubyの環境を用意する

```dockerfile
FROM ruby:3.3.6
```

`FROM` は、Dockerイメージを作成する際の土台となるイメージを指定する命令です。

今回は、Ruby 3.3.6がインストールされたイメージを使用しています。

つまり、Rubyを一からインストールするのではなく、あらかじめ用意された環境を利用しています。

### 2. 環境変数を設定する

```dockerfile
ENV LANG C.UTF-8
ENV TZ Asia/Tokyo
```

`ENV` は、コンテナ内で使用する環境変数を設定する命令です。

| 設定 | 役割 |
|---|---|
| `LANG C.UTF-8` | 文字コードなどに関するロケールを設定する |
| `TZ Asia/Tokyo` | タイムゾーンを日本時間に設定する |

これにより、アプリケーションを動かす環境の基本設定を行っています。

### 3. Node.jsとYarnの取得元を設定する

:::details Node.jsとYarnとは？
開発環境では、Node.jsとYarnもインストールしています。

- **Node.js**：JavaScriptをブラウザの外でも実行できるようにする実行環境
- **Yarn**：JavaScriptのパッケージを管理するツール

Railsで例えると、YarnはRubyのGemを管理するBundlerに近い役割を持っています。

Node.jsは、CSSやJavaScriptのビルドなどに使用する開発ツールを動かすために必要になる場合があります。

一方、Yarnは、それらのツールやライブラリをインストール・管理するために使用します。

今回の `Dockerfile.dev` では、これらをインストールすることで、開発に必要な環境を準備しています。
:::

```dockerfile
RUN apt-get update -qq \
&& apt-get install -y ca-certificates curl gnupg \
&& mkdir -p /etc/apt/keyrings \
&& curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
&& NODE_MAJOR=20 \
&& echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list \
&& wget --quiet -O - /tmp/pubkey.gpg https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
&& echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
```

ここでは、Node.jsとYarnをインストールするための準備をしています。

コードが長いため、主な処理を表に整理します。

| コード | 役割 |
|---|---|
| `apt-get update -qq` | パッケージの一覧情報を更新する |
| `apt-get install` | 必要なパッケージをインストールする |
| `mkdir -p` | 署名鍵を保存するディレクトリを作成する |
| `curl` | NodeSourceの署名鍵を取得する |
| `gpg --dearmor` | 取得した鍵を保存に適した形式へ変換する |
| `NODE_MAJOR=20` | Node.jsのメジャーバージョンを指定する |
| `echo ... nodesource.list` | Node.jsの取得元を登録する |
| `wget ... apt-key add` | Yarnの署名鍵を取得・登録する |
| `echo ... yarn.list` | Yarnの取得元を登録する |

`&&` は、直前のコマンドが成功した場合に次のコマンドを実行するための記述です。

なお、今回使用している `Dockerfile.dev` には、Yarnの公開鍵を登録するために `apt-key` が使用されています。

`apt-key` は現在非推奨となっており、リポジトリごとに公開鍵を管理する `signed-by` 方式が推奨されています。

現在の開発環境では動作していますが、今後の互換性や安全性を考慮し、別途修正する予定です。

### 4. 開発に必要なツールをインストールする

```dockerfile
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs yarn vim
```

ここで、実際に開発に必要なパッケージをインストールします。

| パッケージ | 役割 |
|---|---|
| `build-essential` | C言語などのコンパイルに必要なツール |
| `libpq-dev` | PostgreSQLを利用するための開発用ライブラリ |
| `nodejs` | JavaScriptを実行するための環境 |
| `yarn` | JavaScript関連のパッケージを管理するツール |
| `vim` | テキストを編集するためのエディタ |

例えば、RubyのGemには、インストール時にC言語などで書かれた拡張機能をコンパイルするものがあります。

そのため、`build-essential` などのツールが必要になります。

### 5. 作業ディレクトリを用意する

```dockerfile
RUN mkdir /myapp
WORKDIR /myapp
```

`mkdir` はディレクトリを作成するコマンドです。

`WORKDIR` は、以降の命令で使用する作業ディレクトリを指定します。

今回は `/myapp` を作業場所として設定しています。

### 6. Bundlerをインストールする

```dockerfile
RUN gem install bundler
```

Bundlerは、RubyのGemの依存関係を管理するためのツールです。

Railsアプリケーションでは、`Gemfile` に使用するGemを記述し、Bundlerを使って必要なGemをインストールします。

ただし、このコードではBundler自体をインストールしているだけで、`bundle install` は実行していません。

### 7. アプリケーションのファイルをコピーする

```dockerfile
COPY . /myapp
```

`COPY` は、ビルド時に指定されたコンテキスト内のファイルを、Dockerイメージ内へコピーする命令です。

今回は、アプリケーションのファイルを `/myapp` にコピーしています。

ただし、`.dockerignore` に指定されているファイルなどはコピー対象から除外されます。

### Dockerfile.devで行っていること

ここまでの処理を整理すると、次のようになります。

| 処理 | 目的 |
|---|---|
| Rubyイメージの指定 | Rubyを実行できる環境を用意する |
| 環境変数の設定 | 文字コードやタイムゾーンを設定する |
| Node.js・Yarnの設定 | パッケージの取得元を登録する |
| 開発ツールの導入 | GemやJavaScript関連の処理に必要な環境を用意する |
| 作業ディレクトリの設定 | アプリケーションの作業場所を決める |
| Bundlerのインストール | Gemの依存関係を管理できるようにする |
| ファイルのコピー | アプリケーションのコードをイメージに含める |

今回の `Dockerfile.dev` では、**Railsアプリケーションを開発するための実行環境を用意すること**が中心となっています。

一方で、`bundle install` や `yarn install`、CSSのビルドなどは、このDockerfile内では明示的に実行されていません。

これらの処理がどこで実行されるかは、Docker Composeや起動時の設定などを確認する必要があります。

つまり、**Dockerイメージのビルドと、Railsのアセットのビルドは別の処理**であることが分かりました。

## まとめ

これまで何となく使っていた「ビルド」という言葉について、今回改めて調べることで、少し理解を深めることができました。
特に、RailsのアセットのビルドとDockerのビルドでは、作成するものや目的が異なることが分かりました。

また、実際に使用しているDockerfileを確認することで、普段あまり意識していなかった処理の役割についても知ることができました。
まだ具体的な処理の仕組みまで十分に理解できたとは言えませんが、今後も実際のコードや動作と照らし合わせながら、少しずつ理解を深めていきたいと思います。
