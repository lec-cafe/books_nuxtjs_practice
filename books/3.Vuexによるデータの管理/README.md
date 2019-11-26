# Vuex によるデータの管理

REST APIを利用して Github からデータを取得する事ができましたが、
API 通信には若干の時間がかかります。

例えば Issue の一覧ページから個別ページに遷移する場面を考えてみると、
Issue の一覧ページでデータを取得して、また個別ページでもデータを取得して、
再度一覧に戻ったタイミングでもまたデータの取得が必要になります。

アプリケーションの仕様がゆるす限り、こうした API 通信結果のデータは
可能な限りで再利用していくのが UI 設計的にも効率的です。

Vue.js では アプリケーション内でデータを共有するための仕組みとして Vuex と呼ばれる仕組みが採用されています。
Vuex を利用することで、複雑なアプリケーション内のデータをシンプルに共有することができるようになります。

ここでは、先程取得した Issue 一覧のデータを Vuex に格納して、他のページで再利用する方法を考えてみましょう。

## データをVuexに格納する

Vuex Store は Vue.js で利用可能な データストアの構造です。

複数のページや vue コンポーネントで、データを共有する際に非常に便利なデータモデルとなっています。

Vuex でデータを管理するためには、以下の３つの仕組みを利用する必要があります。

- state : Vuex 上で管理されるデータ
- mutation : state を操作する関数。同期的に動作する。
- action : mutation を操作しながら実際のデータ操作ロジックを記述する非同期な関数

Vuex では これらの state mutation action の ３つを組み合わせながら、
アプリケーション全体で共有するデータを管理していきます。

![](/images/3/vuex.png)

https://vuex.vuejs.org/ja/

### Vuex Store の作成

issue 一覧のデータをページ間で共有できるように、Vuex Store を構築してみましょう。

Nuxt.js では、`store` フォルダに コードを配置するだけで、
自動的にアプリケーションで利用可能な Vuex Store が構築されます。

まずは、`store/index.js` を作成して以下のようなコードを記述します。

```js
export const state = () => {
  return {
    issues: []
  }
}

export const mutations = {
  SET_ISSUES (state, issues) {
    state.issues = issues
  }
}

export const actions = {
  async fetchIssues (ctx) {
    const url = "/repos/lec-cafe/book_nuxt_api_state/issues"
    const {data} = await this.$axios.get(url, {
      headers: {
        Authorization: "token YOUR_GITHUB_PRESONAL_ACCESS_TOKEN"
      }
    })
    console.log(data)
    ctx.commit("SET_ISSUES", data)
  }
}
```

::: tip 
初めて `store` フォルダ内に JS ファイルを作成した場合、
既存の `npm run dev` プロセスを再起動する必要があります。
:::

Vuex では、上記のような形で `exports` を使って state mutations actions をそれぞれ記述していきます。

それぞれのコードを一つづつ確認していきましょう。

```js
export const state = () => {
  return {
    issues: []
  }
}
```

state は vue コンポーネントから data 部分を抜き出したような構造です。 
Vuex ステートの初期値を記述していきます。

```js
export const mutations = {
  SET_ISSUES (state, issues) {
    state.issues = issues
  }
}
```

mutationsは state を変更するための関数です。
第一引数に `state`を、第二引数に任意の値を受け取る事ができます。

state の変更はすべてこの mutations 経由で行われるため、
mutations の定義を確認すれば、state にどの様な操作が行われるのかひと目で確認することができるようになります。

```js
export const actions = {
  async fetchIssues (ctx) {
    const url = "/repos/lec-cafe/book_nuxt_api_state/issues"
    const {data} = await this.$axios.get(url, {
      headers: {
        Authorization: "token YOUR_GITHUB_PRESONAL_ACCESS_TOKEN"
      }
    })
    console.log(data)
    ctx.commit("SET_ISSUES", data)
  }
}
```

actions は実際のアプリケーションロジックを含む、 state の操作関数です。

actions は 第一引数に `ctx` を受け取り、中から `ctx.commit` `ctx.dispatch` などの関数を受け取る事ができます。

`commit` は mutations を呼び出すための関数で、第一引数に mutations 名を、
第二引数は、mutations 関数の 第二引数に渡されます。

`dispatch` は actions を呼び出すための関数で、他のactionをaction 内で呼び出す際に利用されます。
第一引数は actions 名で、第二引数は、 actions 関数の第二引数に渡されます。

## Vuex Store の利用

実際に Vuex Store が作成できたら 作成した actions を vue コンポーネントから利用していきましょう。

Issue 一覧画面で、この Vuex Store を利用する場合、`pages/index.vue` は以下のような形になります。

```js
export default {
  data(){
    return {}
  },
  computed: {
    issues(){
      return this.$store.state.issues
    }
  },
  async mounted(){
    this.$store.dispatch("fetchIssues")
  }
}
```

data で定義していた `issues` は Vuex Store からの参照で computed プロパティに置き換えられます。

データの取得は、 `this.$store.dispatch("fetchIssues")` として action をvue コンポーネント内部でコールします。

同様に mutations もまた `this.$store.commit` を呼び出してコールすることができます。

Issue の個別のページ `pages/issue/_id/index.vue` では以下のようなコードになります。

```vue
<template>
  <section v-if="issue">
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
    return {}
  },
  computed: {
    issue(){
      for(let issue of this.$store.state.issues){
        if(this.$route.params.id == issue.number){
          return issue
        }
      }
      return null
    }
  },
  created(){
    if(!this.issue){
      // データがないケースでは、一覧画面に戻る
      this.$router.push("/")
    }
  }
}
</script>
```

またこの場合、直接 詳細画面にアクセスした場合、一覧画面での mounted 処理は走らないため、
Vuex のデータは空っぽです。

データがないケースでエラーにならないよう created で一覧への遷移処理を追加しています。

::: tip 
上記のような state データの操作処理は、
getter を用いて記述することでよりシンプルに記述することも可能です。
:::
