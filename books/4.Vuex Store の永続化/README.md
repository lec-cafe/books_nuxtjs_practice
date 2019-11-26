# Vuex Store の初期化・永続化

## Vuex Store の初期化

Vuex にアプリケーションの状態を保存しても、データはリロードしたタイミングでリセットされてしまいます。
アプリケーションの様々な場面で初期化が必要になります。

アプリケーション全体で必要なデータは、プラグインを用いてページ表示前に初期化するなどして工夫する必要があります。

例えば特定の Vuex Action を ページ表示前に呼出だす場合、
次のような `plugins/init.js` を作成して、 `nuxt.config.js` に プラグインを登録します。

```js
module.exports = (ctx) => {
  return ctx.store.dispatch("fetchIssues")
}
```

```js
// nuxt.config.js
module.exports = {
  plugins: [
    {src:"~/plugins/init.js"}
  ],
}
```

## Vuex Store の永続化


毎回読み込む必要のないようなデータに関しては、
`vuex-persistedstate` を利用して localstorage に永続化することもできます。

`vuex-persistedstate` は Vuex Store のデータを
localstorage に自動的に格納させるためのライブラリです。 

`vuex-persistedstate` は npm でインストールして利用します。

```
$ npm i vuex-persistedstate
```

モジュールをインストールしたら、以下のような形で `~/plugins/localStorage.js` を作成します。

```js
import createPersistedState from 'vuex-persistedstate'

export default ({store}) => {
  window.onNuxtReady(() => {
    createPersistedState({})(store)
  })
}
```

作成したファイルを nuxt.config.js に登録して準備は完了です。

```js
modules.export = {
  // ...
  plugins: [{ src: '~/plugins/localStorage.js', ssr: false }]
  // ...
}
```

`vuex-persistedstate` を有効化すると、
すべての vuex データが自動的にローカルストレージに格納されるようになり、
リロードしてもデータは消えなくなります。

すべての Vuex データを保存するのが冗長で、特定の Vuex データのみを保持したい場合には、 
`createPersistedState` 関数のコール時に オプションでキーを渡します。

```js
import createPersistedState from 'vuex-persistedstate'

export default ({store}) => {
  window.onNuxtReady(() => {
    createPersistedState({
        paths: [ 
          "issues"
        ]
    })(store)
  })
}
```
