# \_SwiftUI の MEMO

Draft 関連であったり、書き殴り系のメモを置いておく場所

# 書く順

1. SwiftUI におけるデータの流れ
2. 状態管理
3. Identity とレンダリング
4. 深掘り SwiftUI

# WWDC で見たやつ

## Data Flow Through SwiftUI WWDC19

- SSOT 守ろうね
- BindingObject 使おうね
- View は値型で、再レンダリングされても State は保持されるよ
  くらいの内容

トークの内容全てが SSOT を強調してる感じ。

## SwiftUI Essentials WWDC19

発言 MEMO ＝ 21:00 くらい

UIKit

> UIView defines storage for common view properties like alpha and backgroundColor.

> Our Custom View would inherit the stored properties of UIView as well as adding more properties for its own custom behavior.
> ![](/images/_SwiftUIのMEMO/image20230815-233452.png)

SwiftUI

> ...~these modifiers creates their own view.

> And this means that the storage for those properties is distributed across our view hierarchy in each of these modifier views instead of being inherited by every individual view.
> ![](/images/_SwiftUIのMEMO/image20230815-233424.png)

大体の SwiftUI コードでは、struct someView: View とあり、View プロトコルを継承している。View プロトコルとはなんなのか。

```swift
struct OrderHistory: View {
  let previousOrders: [CompletedOrder]

  var body: some View {
    List(previousOrders) { order in
      VStack(alignment: .leading) {
        Text(order.summary)
        Text(order.purchaseData)
          .font(.subheadline)
          .foregroundColor(.secondary)
      }
    }
  }
}
```

SwiftUI の公式ドキュメントから引用

> The View protocol provides a set of modifiers — protocol methods with default implementations — that you use to configure views in the layout of your app.

まあ役目としては UIView と大体同じ。

> Modifiers work by wrapping the view instance on which you call them in another view with the specified characteristics, as described in Configuring views. For example, adding the opacity(\_:) modifier to a text view returns a new view with some amount of transparency:

注意すべきなのは、さっきも述べたように Modifier が新しい View を返しているということ。

View プロトコルの内部はこうなってる

```swift
public protocol View {
  associatedtype Body : View
  @ViewBuilder var body: Self.Body { get }
}
```

新しい疑問、@ViewBuilder ってなんなのか。

以下のコードを見てください。

```swift
// Sample1
@ViewBuilder
func HelloView() -> some View{
    Text("Hello")
    Text("Hello")
    Text("Hello")
}
```

このまんまです。
ViewBuilder は、「View を返すクロージャ」を引数に取る Property Wrapper。
特徴的なのは、複数の View を返すことができるということ。

話が戻りますが、View プロトコルの body が@ViewBuilder でマークされているということは...

```swift
// Sample1
struct ContentView: View {
  var body: some View {
    Text("Hello")
    Text("Hello")
  }
}
```

よく見るこのコードが書けるってことですね。
View 以外が帰ってくるのを防ぐ＆複数の View を返すことができるために ViewBuilder という Property Wrapper が必要になっています。

# Modifier について

元となる View をラップしていて、新しい View を生成しているイメージ
![](/images/_SwiftUIのMEMO/image20230815-192608.png)

Q.いやいや、View 階層が大きくなってるじゃん。パフォーマンス悪化するでしょ
A.SwiftUI のレンダリングシステムがあれこれしてくれるから大丈夫らしい。詳しく調べる。

#　 Result Builder と Property Wrapper
プロパティラッパーは知ってるのでまとめる優先度低い。ただ、個人的に書くことがありそうだから詳しく知っておきたい。
リザルトビルダーは殆ど知らないので、優先的にまとめます。
