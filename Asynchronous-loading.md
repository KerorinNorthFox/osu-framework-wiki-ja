# 非同期読み込み

osu!frameworkはノンブロッキング読み込みに重点を置き、いかなる時点でもインターフェイススレッドをブロックしないように努めています。したがって、コンポーネントをロードするときに非同期パターンを使用することが推奨されます (場合によっては強制されます)。

# コンポーネントの事前読み込み

大きなコンポーネントは事前にプリロードすることをお勧めします。たとえば、ユーザーがメニュー画面に到達した後、表示される次の画面で読み込みが開始される可能性があります。

バックグラウンドでコンポーネントを読み込む最も簡単な方法は、`Drawable`内から `LoadComponentAsync()`を呼び出すことです。このメソッドは、読み込み完了時にアクションを実行するために使用できるコールバックも公開します。これは、準備ができたらすぐに新しく読み込むコンポーネントを表示するのに役立ちます。

コンポーネントを即座に使用:

```csharp
LoadComponentAsync(new MyComponent(), Add);
```

将来の使用のためにプリロード:

```csharp
var myComponent = new MyComponent();

LoadComponentAsync(myComponent);

// ...

Add(myComponent); // 非同期ロードがまだ完了していない場合、これによりロードのブロッキングが発生することに注意してください。
```

このメソッドは、カスタムのブロック動作が必要な場合に`Task.Wait()`でブロックできる`Task`を返します。

# `[BackgroundDependencyLoader]`による非同期チェーン

コンポーネントがロードされると、`ShouldBeAlive == true`を持つネストされた子をすべてロードしようとします。このチェーンを利用するには、すべての読み込みコードが`ctor`または`[BackgroundDependencyLoader]`属性でマークされたプライベート メソッドのいずれかにあることを確認してください。後者は、非同期ロードにネストされていないコンポーネントの新しいインスタンスを構築する際の潜在的なオーバーヘッドを回避するため、推奨されます。

```csharp
public class MainComponent : CompositeDrawable
{
    [BackgroundDependencyLoader]
    private void load()
    {
        Child = new NestedComponent();
        // 必要に応じてこれをctor内に含めることもできますが、通常は効率を最大化するために可能な限りBDLを使用します。
    }
}

public class NestedComponent : Drawable
{
    [BackgroundDependencyLoader]
    private void load()
    {
        // long running load
    }
}
```

次に、次の呼び出しを使用して`MainComponent`と`NestedComponent`を非同期的にロードし、ネストされたツリーが完全にロードされたときに`MainCompoent`のみを追加します。

```csharp
LoadComponentAsync(new MainComponent(), Add);
```

# 事前にパッケージ化された非同期動作のオーバーライド

一部のクラスには、事前に構築された非同期動作が付属しています。一例として`Screen`があります。これは、まだロードされた状態でない場合に`Push`された子画面を自動的にロードします。このような非同期動作をオーバーライドするには、前述のように、プリロード呼び出しを手動でブロックすることで実現できます。

```csharp
var blocking = new BlockingScreen();

LoadComponentAsync(blocking).Wait();
screenStack.Push(blocking);
```

# 長時間読み込まれるコンポーネント (オンラインでの検索など)

特別な属性`[LongRunningLoad]`は、長時間 (通常は 100 ミリ秒を超える) かかるロードを必要とするコンポーネントをマークするために存在します。これは通常、ネットワークソースからテクスチャ (またはその他のコンテンツ) を取得するドローアブルです。これをクラスにアタッチすると、そのクラスのすべての使用法が非同期コンテキストで正しく読み込まれることを保証できます。

この属性については、次の 2 つの重要な点に注意してください。

## マークされたコンポーネントは、独自の分離されたスレッド プールで実行される

これにより、ネットワーク要求などによるスレッドプールの飽和の可能性が回避されます。`LoadComponentAsync()`経由でロードされた通常のコンポーネントは、`LongRunningLoad`コンポーネントの影響を受けません。

## マークされたコンポーネントはトップレベルの非同期ロードである必要がある

`[LongRunningLoad]`コンポーネントが誤って標準の非同期読み込みスレッドプールに置かれることを避けるために、コンポーネントは常に`LoadComponentAsync()`経由で直接実行する必要があります。そうでない場合は、例外がスローされます。