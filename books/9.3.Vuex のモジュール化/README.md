# Vuex Store の モジュール化

Vuex Store 上のデータは、アプリケーションの規模に比例意して肥大化していきます。

単一の `store/index.js` ですべての state を管理するのは、
コードの可読性的にも、依存管理上でも問題があるため、
一定の規模で Vuex Store の分割を行う必要があります。

## Vuex Store のモジュール化

例えば、Github API のデータを管理する Vuex Store として、
`store/github.js` を以下のような形で作成します。

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

`store/github.js` という名前で作成した Vuex Store は 
モジュール名 `github` の Vuex Store として機能します。

このような形で モジュール化された Vuex Store を vue コンポーネントから参照する際には、
以下のような方法を取ります。

```js
export default {
  data(){
    return {}
  },
  computed: {
    issues(){
      return this.$store.state.github.issues
    }
  },
  async mounted(){
    this.$store.dispatch("github/fetchIssues")
  }
}
```

state は モジュール名によりネストされた状態で展開されます。

mutations や actions の呼び出しは、 モジュール名 `github/` のプレフィックスを利用した呼び出しが必要になります。

vue コンポーネントからの呼び出しとは対象的に、
モジュール内部で モジュール内の mutations や actions を呼び出す際には、
`commit("SET_ISSUES", data)` の様な形でモジュール名を意識せず記述できる点には注意してください。

## モジュール間の連携

あるモジュールから別のモジュールにアクセスする際には、注意が必要です。

```js
export const actions = {
    someAction ({ dispatch, commit, rootState }) {
      dispatch('someOtherAction', null, { root: true }) 

      commit('someMutation', null, { root: true })
    },
}
```

モジュール外の グローバルな state には `rootState` を利用してアクセスすることが可能です。

また、`dispatch('someOtherAction')` のように actions をコールしても、
モジュール内の呼び出しになってしまうため、モジュール外の actions をコールする際には、
第三引数に `{ root: true }` を付与します。
(第二引数は actions のパラメータ)

## Vuex Store のバインディング

Vuex で提供される バインディング ヘルパーを利用して、`this.$store` の記述を経由することなく、
簡単に Vuex のオブジェクトを利用することができます。

### state のマッピング

vue コンポーネント内で mapState を利用することで、state のマッピングを簡単に行うことができます。

```js
import { mapState } from 'vuex'

export default {
  computed: {
    ...mapState({
      count: state => state.count,
    })    
  },
}
```

モジュールの state をマップする場合には、第一引数にモジュール名を指定します。

```js
import { mapState } from 'vuex'

export default {
  computed: {
    ...mapState({
      count: state => state.count,
    }),
    ...mapState('some/nested/module', {
      a: state => state.a,
      b: state => state.b
    })        
  },
}
```

### actions/mutations のマッピング

vue コンポーネント内で mapActions や mapMutations を利用することで、actions や mutations  のマッピングを簡単に行うことができます。

```js
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', 
      'incrementBy' 
    ]),
    ...mapActions({
      add: 'increment' // 別名での登録
    })
  }
}
```

state の際と同様に、第一引数に モジュール名を指定して、モジュールの actions や mutations を登録することもできます。

```js
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', 
      'incrementBy' 
    ]),
    ...mapActions('some/nested/module', [
      'foo',
      'bar' 
    ])
  }
}
```
