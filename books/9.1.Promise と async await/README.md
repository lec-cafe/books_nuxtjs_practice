# Promise と async await

JavaScript で非同期処理を行う際に重要になる Promise と Async Await の仕組みについて理解しておきましょう。

## Promise

Promise は 非同期な処理を JavaScript で表現するための共通のフォーマットです。

Promise は Promise クラスを利用して作成する事ができます。
下記は一秒後に処理が完了する Promise です。

```js 
const promise = new Promise( (resolve)=>{
  setTimeout( ()=>{
    resolve(true)
  },1000)
})
```

通常 Promise を自分で作成するケースは稀かもしれません。
多くの JavaScript ライブラリで、 非同期の処理は Promise ベースで実装されており 
通常はこうしたライブラリから 生成された promise のオブジェクトを受け取るケースがほとんどです。

生成された promise オブジェクトは、`then` を呼び出すことで、処理完了時の動作を定義することができます。

```js
promise.then((result)=>{
  // 処理が完了したタイミングで呼び出される
  console.log(result)
})
```

`then` で定義する関数の第一引数には、 resolve が呼び出された際の引数が格納されます。
また、 then は チェインすることも可能で、チェインされたthen では直前の then で return した値が
次の then の関数の第一引数に渡されます。

```js
const promise = new Promise( (resolve)=>{
  setTimeout( ()=>{
    resolve("first message")
  },1000)
})
promise.then((result)=>{
  console.log(result) // first message
  return "second message" 
}).then((result)=>{
    console.log(result) // second message
})
```

Promise 内で throw された エラーは、 catch を利用して受け取ることができます。

```js
promise.catch((e)=>{
  console.log(e)
})
```

## async await 

async await は ES2017 で追加された、 非同期処理の新しい書き方です。

async を付与した関数(async function) は 自動的に Promise を返却するよう変換されます。

```js
const myFunc = async () => {
  return "hello world"
}
// 上の関数は下の関数と同じ
const myFunc = () => {
  return new Promise( (resolve)=>{
    resolve("hello world")
  })
}
```

また async function の内部では await キーワードが利用可能です。

`await promise` の形式でコードを記述することで、 then を記述することなく、 
promise の処理完了を待つことができます。

```js
const promise = new Promise( (resolve)=>{
  setTimeout( ()=>{
    resolve("hello world")
  },1000)
})

const myFunc = async () => {
  const message = await promise
  console.log(message) // "hello world"
}
```
