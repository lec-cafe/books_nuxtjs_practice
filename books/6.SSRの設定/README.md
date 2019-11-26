# SSR の設定

Nuxt.js の 特徴の一つとして SSR の対応があります。

SSR はブラウザ以外の場所で Vue.js のコードを動かす仕組みで、
Node のサーバを用いて Nuxt.js アプリケーションを動作させる際以外にも、
`npm run generate` でページを静的に出力する際に重要になってくる機能です。

SSR の機能を利用することで、`npm run generate` で生成されるページに、
APIのデータを埋め込む事ができるようになり、 SEO や OGP の生成に役立てることができるようになります。

## fetch / asyncData 

`pages` フォルダ内の .vue コンポーネントでは `fetch` や `asyncData` といった関数を定義することが可能で、
このセクションを用いて SSR 時の挙動を制御することができます。

fetch の例

```vue
<template>
  <h1>Issues: {{ $store.state.issues.length }}</h1>
</template>

<script>
export default {
  async fetch (ctx) {
    ctx.store.dispatch('fetchIssues')
  }
}
</script>
```

asyncData の例

```js
export default {
  data () {
    return { project: 'default' }
  },
  asyncData (context) {
    return { project: 'nuxt' }
  }
}
```

fetch は SSR のフェーズで任意の動作を行うことができるため、
単純にVuex Store のアクションなどを呼び出すのに向いています。

asyncData は SSR のフェーズで動作し、return したオブジェクトを data のオブジェクトとマージすることができます。

fetch も asyncData もどちらのセクションも、ブラウザ上では動作することはありません。

## head の設定

`pages` フォルダ内の .vue コンポーネントでは `head` 関数を定義して、head要素内の値をカスタマイズすることが可能です。

```vue
<template>
  <h1>{{ title }}</h1>
</template>

<script>
export default {
  data () {
    return {
      title: 'Hello World!'
    }
  },
  head () {
    return {
      title: this.title,
      meta: [
        { hid: 'description', name: 'description', content: 'My custom description' }
      ]
    }
  }
}
</script>
```

head 関数内で 例えば　APIの情報などを取得したい場合には、
`fetch` や `asyncData` などのSSRフェーズの関数を利用してデータを準備する必要があります。 

