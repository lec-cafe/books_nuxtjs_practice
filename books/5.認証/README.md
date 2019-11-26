# 認証

本格的なアプリケーションで必要となる認証もまた、データの永続化が必要となる場面です。

認証は、認証トークンの永続化やユーザ情報の更新など、
永続化と再取得のフローが複雑で、自前で実装するには、やや複雑なところがあります。

こうした 認証周りの永続化に関しては、
`vuex-persistedstate` を用いて自前で実装を行うよりも、
Nuxt.js の Auth Module を利用するほうが高度に認証情報の永続化を実現できるかもしれません。

[Introduction - Auth Module](https://auth.nuxtjs.org/)

## Auth Modules の導入

Nuxt.js の Auth Modulesを利用するには npm 経由でモジュールをインストールしてください。

```bash
$ npm i @nuxtjs/auth @nuxtjs/axios
```

モジュールのインストールができたら `nuxt.config.js` にてモジュールを有効化します。

```js
modules.export = {
  //...
  modules: [
    '@nuxtjs/axios',
    '@nuxtjs/auth'
  ],
  auth: {
    // Options
  }  
}
```

::: tip
`auth-module` 利用時には  Vuex Store を有効化する必要があります。
まだ store フォルダに何も格納していない場合は、 `store/index.js` などを作成して Vuex の機能を有効化してください。
:::

Auth Modules を利用する上で必要な設定は以下の２つです。

- 認証関連のルート設定
- 認証処理を行うための通信設定

順にその内容を確認していきましょう。

## 認証関連のルート設定

SPA におけるそれぞれのルートについて、ユーザの認証状態に応じた表示非表示の制御を行うことができます。

Auth Module で認証ユーザ向けのルートを作成する場合、ミドルウェアを用いるのが便利です。

それぞれの ページコンポーネントで以下のようなにミドルウェアの指定を行い、
認証ルートで有ることを明示することができます。

```js
export default {
  middleware: 'auth',
  data(){
      return {
          //...
      }
  },
  mounted(){
      //...
  }  
}
```

それぞれのページコンポーネントにミドルウェアを設定するのが面倒で、
アプリケーション全体を 認証ユーザ向けルートとする場合は、
`nuxt.config.js` に以下のような指定を行い、
アプリケーション全体にミドルウェアを適用します。

```js
router: {
  middleware: ['auth']
}
```

上記の設定を行った場合、個別のページコンポーネントで `auth: false` を設定し、
非認証ユーザ向けのルートを作成することも可能です。

```js
export default {
  auth: false,
  data(){
      return {
          //...
      }
  },
  mounted(){
      //...
  }  
}
```

また、認証ユーザ向けルートに、非認証ユーザがアクセスした際の挙動なども、
`nuxt.config.js` に`auth` を追加して定義することが可能です。

```js
modules.export = {
  auth: {
    redirect: {
      login: '/login',
      logout: '/',
      callback: '/login',
      home: '/'
    }    
  }
}
```

- `login` は 未ログイン時に認証ルートへアクセスした際のリダイレクトURLです。
- `logout` はログアウト時のリダイレクトURLです。
- `callback` はOauth認証等で必要となる コールバックルートです。
- `home` はログイン後のリダイレクトURLです。

## 認証処理を行うための通信設定

実際の API コール等を用いてユーザのログイン認証を行う処理を、
Auth Modules では `Strategies`  と呼びます。

Auth Modules では、いくつかの組み込み `Strategies` が用意されており、
Github や Facebook / Google などの Oauth を使った認証を簡単に実装することが可能な用になっています。

例えば Github認証のアプリケーションを用意する場合、
`nuxt.config.js` の `auth` セクションに以下のような設定を追記します。

```js
modules.export = {
  auth: {
    strategies: {
        github: {
          client_id: '...',
          client_secret: '...'
        },
    }
  }  
}
```

以下のURL から取得した、
Github の client_id や client_secret を埋め込み、認証処理の設定は完了です。

https://github.com/settings/developers

### local storategy

任意のスクラッチで用意した認証APIを利用する場合には `local` の strategies が利用可能です。

```js
modules.export = {
  auth: {
    strategies: {
      local: {
        endpoints: {
          login: { 
            url: '/api/auth/login', 
            method: 'post', 
            propertyName: 'token' 
          },
          logout: { 
            url: '/api/auth/logout', 
            method: 'post' 
          },
          user: { 
            url: '/api/auth/user', 
            method: 'get', 
            propertyName: 'user' 
          }
        },
      }
    }
  }
}
```

`endpoints` 内の各セクションに有効なAPIの URL や method, ヘッダー情報等を記述します。
構造は `$axios` のオプション構造となっています。

- `login` ログイン時に発行されるAPI
- `logout` ログアウト時に発行されるAPI
- `user` ユーザ情報取得時に発行されるAPI

Auth Module は内部でAPIを発行する際に、自動的にレスポンスを解釈しようとします。
`login` API の場合は `token` キーからトークンを
`user` API の場合は `user` キーからユーザ情報を取得するようになっていますが、
任意のキーでレスポンスを設定したい場合には `propertyName` にそのキー名を指定してください。

ユーザ情報の取得など 認証が必要なAPI は `login` API 経由で取得したトークンを用いて
認証付きのリクエストが実施されます。
トークンは `Bearer` 方式で各API のヘッダに自動的に付与されるため、
アプリケーション内部の `$axios` 全てで、認証を意識せずAPIリクエストを送出することができます。

## Auth Modulesを用いたログイン処理

Auth Module の設定が終わったら実際にログインの処理を実行してみましょう。

Auth Module では各種ログイン関連処理を`$auth` 経由で実行することができ、
ログインについては、 `$auth.loginWith()` 関数が用意されています。

```js
const { data } = await this.$auth.loginWith('github')
```

`$auth.loginWith()` は 第一引数で指定した strategies を利用して、ログイン処理を実行します。

それぞれの ログイン処理は strategies の実装に依存するため、
`local` storategy の場合には以下のような形で第二引数に認証情報を 渡します。

```js
const { data } = await this.$auth.loginWith('local', {...opt})
```

`local` の場合、`$axios` を利用した API コールが行われ Promise が return されます。
フォーム等経由で取得されるログイン情報は axios の Config 形式で第二引数に渡すことが可能です。

### ログイン後のユーザ情報取得

ログイン後のユーザ情報は `this.$auth.user` 経由で取得可能です。

これは実際には Vuex に格納されたグローバルなデータとなっているため、
Vue.js の開発ツール上で確認したり、 Vuex の形式でアクセスすることも可能です。

```js
// Access using $auth
this.$auth.user
​
// Access using vuex
this.$store.state.auth.user
```

