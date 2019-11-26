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
まだ store フォルダに何も格納していない場合は、 `store/index.js` を作成して Vuex の機能を有効化してください。
:::

## Auth Modulesの設定

Auth Modules を利用する上で必要な設定は以下の２つです。

- ログイン関連処理時にリダイレクトするルート
- ログイン関連処理時に発行するAPI URL (`Strategies`)

### ルートの設定

Auth Module で認証ルートを作成する場合、ミドルウェアを用います。

ルートごとに、以下のような形でミドルウェアを指定して認証ルートを設定する事ができます。

```js
export default {
  middleware: 'auth'
}
```

また `nuxt.config.js` に以下のような指定を行い、アプリケーション全体を認証ルートとする事ができます。

```js
router: {
  middleware: ['auth']
}
```

グローバルに認証ルートを設定した場合、非認証ルートでは `auth: false` を設定することで、
特定のルートを 認証ルートから除外することもできます。

```js
export default {
  auth: false
}
```

認証ルートを中心に、 Auth Module のリダイレクト挙動を設定できます。

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

### `Strategies` の設定

ログイン関連処理時に発行する API URLの設定 等を取りまとめたものを Auth Modules では 
`Strategies` と呼びます。

Github や Facebook / Google などの Oauth を使った認証では、
デフォルトで `Strategies` の設定パターンが用意されているため、
サービスから提供される client_id 等の情報のみで設定は完了します。

Githubの例:

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

Github の client_id や client_secret は以下のURLから取得可能です。

https://github.com/settings/developers

これらの認証情報は JSのソースとしてアプリケーションに組み込まれる点には注意が必要です。

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
const { data } = await this.$auth.loginWith('local', {...opt})
```

`$auth.loginWith()` は 第一引数で指定した strategies を利用して,ログイン処理を実行します。
それぞれの ログイン処理は strategies の実装に依存しますが、
`local` の場合、`$axios` を利用した API コールが行われ Promise が return されます。
フォーム等経由で取得されるログイン情報は axios の Config 形式で第二引数に渡すことが可能です。

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

