# Nuxt.js を使ったSPA開発

Nuxt.js は Vue.js のアプリケーションフレームワークです。

フロントエンドのライブラリ Vue.js やその周辺ライブラリをまとめて、
Single Page Application (SPA) の構築を行うために必要な構成を取りまとめたのが Nuxt.js です。

Nuxt.js では、 Vue.js で本格的なアプリケーションを作成する際に必要となる VueRouter/Vuex などの各種機能の用意に加え、
複数人でのアプリケーション開発をより円滑に進めるためのディレクトリ規約を提供してくれています。

今回は、サンプルアプリケーションとして、Github APIを利用した Issue 管理ツールを作成します。
サンプルアプリケーションの制作を通じて、 Nuxt.js でのアプリケーション開発の基礎から、
Nuxt.js アプリケーションでの REST APIの発行、 Vuex Store によるデータ管理の手法、を学んでいきましょう。

## 準備

まずは、Nuxt.js の開発環境を構築しましょう。

Nuxt.js でアプリケーション開発を始めるためには、まず npx コマンドを利用して、
アプリケーションの雛形を作成します。

```
$ npx create-nuxt-app my_issue_app
```

ファイルが展開されたら、作成されたフォルダ内に移動して `npm run dev` コマンドを実行すれば、
開発用のWebサーバが立ち上がります。

::: tip 
npx コマンドの利用には npm のバーションが 5.2 以上である必要があります。
:::

## アプリケーションの構成

今回は、 GitHub の Issue 管理ツールということで以下のような機能を考えてみます。

- ユーザは指定したリポジトリの Issue 一覧を閲覧できる。
- ユーザは指定したリポジトリに Issue を追加できる。

上記の機能を提供するために、以下のような画面を構築します。

- Issue 一覧画面
- Issue 個別画面
- Issue 投稿画面 

### ページ作成の準備

今回は、CSSの記述を簡単にするために  Twitter Booststrap を利用しましょう。

`nuxt.config.js` の `head.link` セクションに Bootstrap の CDN を追加します。

```js
module.exports = {
  // ...
  head: {
    // ...
    link: [
      { rel: 'stylesheet', href: "https://stackpath.bootstrapcdn.com/bootstrap/4.2.1/css/bootstrap.min.css" }
    ]
  },
}
```

次に、レイアウトファイルを調整します。

`layouts/defautl.vue`を編集して、以下のような内容で書き換えます。

Nuxt.js において `layouts` フォルダ内の vue ファイルは、全ページでの共通のレイアウト定義となります。


```vue
<template>
  <div>
    <nav class="navbar navbar-expand-lg navbar-light bg-light mb-3">
      <div class="container">
        <a class="navbar-brand" href="/">Navbar</a>
      </div>
    </nav>
    <div class="container">
      <nuxt />
    </div>
  </div>
</template>

<style>
  .container{
    max-width: 640px;
  }
</style>
```

## Issue 一覧画面の作成

レイアウトの準備ができたら、各ページを作成します。
Nuxt.js では ページは `pages` フォルダに vue ファイルを追加して作成していきます。

例えば、Issue 一覧のページは、 `pages/index.vue` に以下のような形で作成します。

```vue
<template>
  <section>
    <h2>Issues</h2>
    <div class="text-right mb-3">
      <router-link class="btn btn-outline-primary" to="/new">Open New Issue</router-link>
    </div>
    <div>
      <div class="list-group list-group-flush">
        <router-link :to="`/issue/${issue.number}`" class="list-group-item list-group-item-action" v-for="(issue,index) in issues" :key="index">
          <div class="">#{{issue.number}} {{issue.title}}</div>
        </router-link>
      </div>
    </div>
  </section>
</template>

<script>

export default {
  data(){
    return {
      issues: [
        {number: "1", title: "Cras justo odio"},
        {number: "2", title: "Dapibus ac facilisis in"},
        {number: "3", title: "Morbi leo risus"},
        {number: "4", title: "Porta ac consectetur ac"},
        {number: "5", title: "Vestibulum at eros"},
      ]
    }
  }
}
</script>

<style>

</style>
```

画面上では Issue 一覧の表示にあたり、
変数 `issues` に定義した配列のデータを `v-for` でループして表示しています。

また Issue 作成画面 `/new` や Issue 詳細画面 `/issue/{id}` へのリンクを 
`router-link` 要素を使って定義しています。

## Issue 詳細画面の作成

Issue 詳細画面は `pages/issue/_id/index.vue` を作成して以下の内容を記述します。

```vue
<template>
  <section>
    <h2 class="mb-3">#{{issue.number}} {{issue.title}}</h2>
    <div class="text-right mb-5">
      <a :href="issue.html_url">Open Github</a>
    </div>
    <div class="mb-5">{{issue.body}}</div>
    <router-link class="btn btn-outline-primary" to="/">back</router-link>
  </section>
</template>

<script>

export default {
  data(){
    return {
      issue: {
        number: "1",
        title: "Cras justo odio",
        html_url: "https://github.com/lec-cafe/book_nuxt_api_state/issues/1",
        body: "Lorem ipsum dolor sit amet, consectetur adipisicing elit. Ipsa nihil pariatur repudiandae soluta voluptas! Aperiam asperiores atque corporis ex hic illum in incidunt iure minus non obcaecati provident quam, voluptatum?"
      },
    }
  }
}
</script>

<style>

</style>
```

変数 `issue` に定義したデータでテンプレートにデータを描画しています。

## Issue 新規作成画面の作成

Issue 新規作成のページも作成しておきましょう。

`pages/new/index.vue` を作成して、以下の内容を記述します。

変数 `form` のデータを 各フォームにバインドしています。 

```vue
<template>
  <section>
    <h2>Open New Issue</h2>
    <div>
      <form>
        <div class="form-group">
          <label>Title</label>
          <input type="text" class="form-control" v-model="form.title">
        </div>
        <div class="form-group">
          <label>Body</label>
          <textarea class="form-control" v-model="form.body"/>
        </div>
        <a class="btn btn-primary" tabindex="" @click="submit">Submit</a>
      </form>
    </div>
  </section>
</template>

<script>

export default {
  data(){
    return {
      form: {
        title: "",
        body: ""
      }
    }
  },
  methods:{
    submit(){
      console.log(this.form)
      this.$router.push("/")
    }
  }
}
</script>

<style>

</style>
```

画面を作成して一覧の動きが確認できたら準備は完了です！ 

次は、実際に REST API を発行して、アプリケーションとして動作できるように実装してみましょう。
