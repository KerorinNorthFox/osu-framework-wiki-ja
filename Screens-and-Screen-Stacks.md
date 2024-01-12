# 画面と画面スタック

osu!framework は、[`Screen`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Screens/IScreen.cs)の概念を利用して実装します。スクリーンを使用すると、他のスクリーンと重ね合わせたり、終了して下に重ねられたスクリーンを表示したりできる単一の「ビュー」を指定できます。

視覚的には、1 枚の紙を別の紙の上に重ねることを想像してください。前向きのシートがユーザーに表示されますが、他の画面はその下に残り、いつでも再開できるようになります。

単一の画面スタック内で一度にアクティブにできるのは 1 つの画面だけです。ただし、複数の異なる画面スタックが存在して、画面を互いに重ね合わせることができます。

* [# 画面スタックの作成](https://github.com/ppy/osu-framework/wiki/Screens-and-Screen-Stacks#creating-a-new-screen-stack)
* [新しい画面の作成](https://github.com/ppy/osu-framework/wiki/Screens-and-Screen-Stacks#creating-a-new-screen)
* [画面遷移の処理](https://github.com/ppy/osu-framework/wiki/Screens-and-Screen-Stacks#handling-screen-transitions)
  * [例](https://github.com/ppy/osu-framework/wiki/Screens-and-Screen-Stacks#example)

# 画面スタックの作成

画面スタックの新しいインスタンスを描画階層に追加するだけです。

```csharp
private ScreenStack awesomeScreenStack = null!;

[BackgroundDependencyLoader]
private void load()
{
    Add(awesomeScreenStack = new ScreenStack());
}
```

# 新しい画面の作成

画面スタックが最初に初期化されるとき、それは空です。ユーザーに表示できるものを含める前に、画面を作成する必要があります。

他の`CompositeDrawable`の場合と同様に、`Screen`に`Drawable`を追加することで、コンテンツを追加できます。

```csharp
public partial class AwesomeScreen : Screen
{
    public AwesomeScreen()
    {
        AddInternal(new Box
        {
            Colour = Color4.Tomato,
            Origin = Anchor.Centre,
            Anchor = Anchor.Centre
        });
    }
}
```

この画面がスタックにプッシュする最初の画面である場合は、`ScreenStack.Push()`を使用してスタックに直接プッシュする必要があります。スタックにすでに画面がある場合は、`Screen.Push()`を使用して画面にプッシュすることもできます。

# 画面遷移の処理

画面には、それに伴う画面変更が発生するたびに呼び出される次のメソッドが含まれています:

* `OnEntering(ScreenTransitionEvent)` - `Screen.Push()`を使用して画面がスタックの先頭に追加されるときに呼び出されます。
* `OnExiting(ScreenExitEvent)` - `Screen.Exit()`を使用して、その下の画面の画面が終了するときに呼び出されます。戻り値を使用して終了プロセスをキャンセルできます。これは、ユーザーに確認ダイアログを表示するようなフローで役立ちます。
* `OnResuming(ScreenTransitionEvent)` - 現在の画面が終了した結果、画面がアクティブ画面になったときに呼び出されます。
* `OnSuspending(ScreenTransitionEvent)` - 別の画面がスタックの最上位にプッシュされ、この画面と置き換わるときに呼び出されます。

## 例

`OnEntering()`メソッドと`OnResuming()`メソッドを使用して、遷移先の画面内で内側への画面関連の遷移を処理できます。`FadeInFromZero()`メソッドは新しい画面のフェードインに最適ですが、`FadeIn()`メソッドを使用すると、画面がすでにあるアルファからフェードを調整できます。

```csharp
public override void OnEntering(ScreenTransitionEvent e)
{
    this.FadeInFromZero(500, Easing.OutQuint);
}

public override void OnSuspending(ScreenTransitionEvent e)
{
    this.FadeIn(500, Easing.OutQuint);
}
```