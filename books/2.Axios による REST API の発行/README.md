# Axios による REST API の発行

Nuxt.js などの フロントアプリケーションでは、
サーバとのデータの受け渡しは、REST API を利用して行うのが一般的です。

JavaScript では REST API の通信に際し、 axios というライブラリを用いる事が多く、
Nuxt.js でも これを利用するための axios モジュールが提供されています。

まずは Nuxt.js 上で axios モジュールを利用するために、以下のコマンドでモジュールのインストールをを粉います。

```
$ npm i @nuxtjs/axios
```

モジュールのインストールが終わったら、axios モジュールを有効化するために`nuxt.config.js`を修正します。

```js
module.exports = {
  modules: [
    '@nuxtjs/axios',
  ],
  axios: {
    baseURL: "https://api.github.com"
  }
}
```

`modules` に `@nuxtjs/axios` を追加するのに加え、
`axios` セクションを追加して、今回主に通信する Github API の baseURL を設定しておきます。

## APIを発行する

準備ができたら実際に API を発行してみましょう。

Github の Issue 一覧ページを作成するために、 Github の REST API を実行して、
Issue 一覧 のデータを取得します。

Github の REST API の一覧は以下の公式ドキュメントからその使いかたを確認する事ができます。

https://developer.github.com/v3/

API の実行には Github の Personal Access Token が必要になるため、以下のURLから事前に準備しておいてください。

https://github.com/settings/tokens

::: tip
Github の Personal Access Token はアカウントのパスワード以上のセキュリティ的な重要性があります。

Personal Access Token が書かれたコードを GitHub の公開リポジトリにUPするなどしないよう十分に注意して取り扱いを行ってください。
:::

### Issue の一覧を取得する

Github から Issue の 一覧を取得する場合 `GET /repos/:owner/:repo/issues` の API を使用します。

[Issues \| GitHub Developer Guide](https://developer.github.com/v3/issues/#list-issues-for-a-repository)

Issue 一覧画面読み込み時に、 Issue 一覧取得 API をコールしてデータを取得することで、
GitHub 上から取得した データを画面に表示する事ができます。

画面読み込み時の処理は `mounted` セクションで定義できるため、
`/pages/index.vue` にて、以下のような形で mounted 内で APIの発行処理を記述します。

```js
export default {
  data(){
    return {
      issues: []
    }
  },
  async mounted(){
    const url = "/repos/lec-cafe/book_nuxt_api_state/issues"
    const response = await this.$axios.get(url, {
      headers: {
        Authorization: "token YOUR_GITHUB_PRESONAL_ACCESS_TOKEN"
      }
    })
    console.log(response)
    this.issues = response.data
  }
}
```

`YOUR_GITHUB_PRESONAL_ACCESS_TOKEN` の箇所は Github から取得した Personal Access Token の値で置き換えてください。

response のデータは `response.data` に格納されているため、以下のように記述するケースも多いでしょう。

```js
export default {
  async mounted(){
    const url = "/repos/lec-cafe/books_nuxtjs_practice/issues"
    const {data} = await this.$axios.get(url, {
      headers: {
        Authorization: "token YOUR_GITHUB_PRESONAL_ACCESS_TOKEN"
      }
    })
    console.log(data)
    this.issues = data
  }
}
```

::: tip 
async await の使いかたについては [こちら](/9.1.Promise%20と%20async%20await/) のドキュメントを確認してください。
:::

::: tip 
axios モジュールの詳細な使いかたについては [こちら](/9.2.axios%20モジュールの使いかた/) のドキュメントを確認してください。
:::

一覧のデータが取得が REST API 経由で実施できたら、リポジトリ名を変更するなどして、
様々なリポジトリ上の Issue 一覧を画面に表示してみましょう。

## その他の API 発行

残りの Isseu 詳細画面や、Issue 作成画面でも同様に API を発行して機能を実装することができます。


[Issue の詳細API](https://developer.github.com/v3/issues/#get-a-single-issue)

[Issue の作成API](https://developer.github.com/v3/issues/#create-an-issue)


例えば、 Issue の詳細 API は、`GET` メソドで、`/repos/:owner/:repo/issues/:issue_number` のURL形式を取るため、
Issue 詳細画面の `pages/issue/_id/index.vue` に以下のような形で mounted を記述します。

```js
export default {
  async mounted(){
    const number = this.$route.params.id
    const url = `/repos/lec-cafe/books_nuxtjs_practice/issues/${number}`
    const {data} = await this.$axios.get(url, {
      headers: {
        Authorization: "token YOUR_GITHUB_PRESONAL_ACCESS_TOKEN"
      }
    })
    console.log(data)
    this.issue = data
  }
}
```

また、 Issue の作成 API は、 `POST` メソドで、`/repos/:owner/:repo/issues`の形式を取ります。
ドキュメントの記載にある通り、作成する API は JSON 形式でパラメータとして付与するため、
Issue 作成画面の作成ボタンでは以下のようなメソドをコールすると Issue 作成処理を実装できるでしょう。

```js
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
    async submit(){
      const url = `/repos/lec-cafe/books_nuxtjs_practice/issues/`
      const requestBody = {
          title: this.form.title,
          body: this.form.body,
      }
      await this.$axios.post(url, requestBody, {
        headers: {
          Authorization: "token YOUR_GITHUB_PRESONAL_ACCESS_TOKEN"
        }
      })
      this.$router.push("/")
    }
  }
}
```

API を利用して Issue アプリケーションの実装ができたら、
次は Vuex などを利用して、アプリケーションのデータフローを改善してみましょう。
