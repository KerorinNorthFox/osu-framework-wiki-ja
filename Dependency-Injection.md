# 依存性の注入

osu!framework では、`Drawable`レベルでの依存性の注入をサポートしています。これは内部的には、`DependencyContainers`を介して行われます。`DependencyContainers`は階層の下に渡され、子によるさらなるカスタマイズ (または置換) のためにいつでもオーバーライドできます。

これの一般的な使用法は、親 (使用時点よりもさらに上のレベルにある可能性があります) から得られる依存性を満たすことです。読み進める前に、[依存性の挿入](https://ja.wikipedia.org/wiki/%E4%BE%9D%E5%AD%98%E6%80%A7%E3%81%AE%E6%B3%A8%E5%85%A5)の一般的な概念を理解することが重要です。

# 実装

osu!framework の依存性注入メカニズムは、C#属性、つまり`[Cached]`、`[Resolved]`、および`[BackgroundDependencyLoader]`に大きく依存しています。
依存関係の設定は、ソースジェネレーションとリフレクションという2つの経路のいずれかを介して行われます。
ソースジェネレーションパスウェイはコンパイル時の最適化から恩恵を受けますが、利用者はそれに応じてコードを調整する必要があるため、これを理解することが重要です。

## ソースジェネレーター

2022.1126.0 リリース以降、主にサポートされている依存性注入の実装は[source generators](https://learn.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview)に依存しています。これがフレームワーク利用者に与える主な影響は次のとおりです:

- ソースジェネレーターベースの依存性注入が機能するには、ソースジェネレーターが DI機構をクラスに注入できるように、`Drawable`クラスが`partial`である必要があります。非準拠のドローアブルでは、[OFSG001](https://github.com/ppy/osu-framework/blob/e5f1aabc22f2f149f48c4fd9ca7c6f8381b00ec0/osu.Framework.SourceGeneration/Analysers/DiagnosticRules.cs#L14-L21)コード インスペクションが発生します。
- より複雑なカスタム DI の使用法で、依存関係をカスタム非描画クラスに`.Inject()`する必要がある場合は、マーカー[`IDependencyInjectionCandidate`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Allocation/IDependencyInjectionCandidate.cs)インターフェイスを実装する必要があります。


ソースジェネレーターの実装は[ここ](https://github.com/ppy/osu-framework/blob/master/osu.Framework.SourceGeneration/Generators/Dependencies/DependencyInjectionSourceGenerator.cs)で見ることができます。

コンパイルと実行のサイクルを高速化するために、ソースジェネレーターはデフォルトでリリースビルドでのみ実行されます。デバッグビルドでは、フォールバックとしてリフレクションパスウェイが使用されます。

## リフレクション

依存性注入の元のレガシー実装では、リフレクションが多用されています。ソースジェネレーターはそのようなドローアブルに独自のコードをアタッチできないため、ユーザーのドローアブルが`partial`としてマークされていない場合に使用されます。

ソースジェネレーターパスウェイが導入されて以来、この実装は下位互換性のためにサポートされていますが、通常は新しいプロジェクトには推奨されません。

# 依存性の保存と取得

依存関係を**キャッシュ** (保存) し、**解決** (取得) する方法はいくつかあります。

## ドローアブルのメンバーでの`[Cache]`

これは最も単純な実装です。

```csharp
/// <summary>
/// 子が使用するために何かをキャッシュするクラス
/// </summary>
public partial class MyGame : Game
{
    [Cached]
    protected readonly MyStore Store = new MyStore();

    public MyGame()
    {
        Child = new Container
        {
            RelativeSizeAxes = Axes.Both,
            Child = new Container
            {
                RelativeSizeAxes = Axes.Both,
                Child = new Container
                {
                    RelativeSizeAxes = Axes.Both,
                    Child = new MyComponent()
                }
            },
        };
    }
}

/// <summary>
/// キャッシュされたクラスを消費したコンポーネント
/// </summary>
public partial class MyComponent : CompositeDrawable
{
    [Resolved]
    protected MyStore FetchedStore { get; private set; } = null!;

    protected override void LoadComplete()
    {
        base.LoadComplete();

        InternalChild = new SpriteText
        {
            Text = FetchedStore.GetAwesomeThing()
        };
    }
}

/// <summary>
/// DI経由でキャッシュされるクラス
/// </summary>
public class MyStore
{
    public string GetAwesomeThing() => "awesome thing!";
}
```

これらの属性のいずれかでマークされたメンバーは、`[BackgroundDependencyLoader]`アノテーション付きメソッドが実行される前に、それぞれのクラスでキャッシュまたは解決されます。

## 解決に`[BackgroundDependencyLoader]`を使用する

これは、すべてが (非同期の可能性がある) `load()`メソッドで確実に行われるようにしたい場合に便利です。

```csharp
/// <summary>
/// 子が使用するために何かをキャッシュするクラス
/// </summary>
public partial class MyGame : Game
{
    [Cached]
    protected readonly MyStore Store = new MyStore();

    public MyGame()
    {
        Child = new Container
        {
            RelativeSizeAxes = Axes.Both,
            Child = new Container
            {
                RelativeSizeAxes = Axes.Both,
                Child = new Container
                {
                    RelativeSizeAxes = Axes.Both,
                    Child = new MyComponent()
                }
            },
        };
    }
}

/// <summary>
/// キャッシュされたクラスを消費したコンポーネント
/// </summary>
public partial class MyComponent : CompositeDrawable
{
    [BackgroundDependencyLoader]
    private void load(MyStore store)
    {
        InternalChild = new SpriteText
        {
            Text = store.GetAwesomeThing()
        };
    }
}

/// <summary>
/// DI経由でキャッシュされるクラス
/// </summary>
public class MyStore
{
    public string GetAwesomeThing() => "awesome thing!";
}
```

## キャッシュに`CreateChildDependencies()`を使用する

キャッシュ可能なオブジェクトの遅延初期化が必要な場合など、より高度なシナリオでは、`[Cached]`属性の代わりにこのメソッドの使用が必要になる場合があります。

```csharp
/// <summary>
/// 子が使用するために何かをキャッシュするクラス
/// </summary>
public partial class MyGame : Game
{
    protected MyStore Store;

    public MyGame()
    {
        Child = new Container
        {
            RelativeSizeAxes = Axes.Both,
            Child = new Container
            {
                RelativeSizeAxes = Axes.Both,
                Child = new Container
                {
                    RelativeSizeAxes = Axes.Both,
                    Child = new MyComponent()
                }
            },
        };
    }

    protected override IReadOnlyDependencyContainer CreateChildDependencies(IReadOnlyDependencyContainer parent)
    {
        var dependencies = new DependencyContainer(base.CreateChildDependencies(parent));
        dependencies.Cache(Store = new MyStore());
        return dependencies;
    }
}
```

`DependencyContainer`クラスは、依存関係をキャッシュするための 2 つのメソッドを公開していることに注意してください:

- `.Cache()`は常に、ランタイムの最も派生した型を使用して依存関係をキャッシュします。これが意味することは、次の例で示されています:

    ```csharp
    public abstract class BaseDependency { }
    public class DerivedDependency : BaseDependency { }

    public partial class DependencyProvider
    {
        private BaseDependency dependency;

        protected override IReadOnlyDependencyContainer CreateChildDependencies(IReadOnlyDependencyContainer parent)
        {
            var dependencies = new DependencyContainer(base.CreateChildDependencies(parent));
            dependencies.Cache(dependency = new DerivedDependency());
            return dependencies;
        }
    }

    public partial class DependencyConsumer
    {
        [Resolved]
        private BaseDependency baseDependency { get; set; }       // WRONG - will fail at runtime

        [Resolved]
        private DerivedDependency derivedDependency { get; set; } // OK
    }
    ```

- `.CacheAs<T>()`は、次の例で示すように、宣言された型を使用して依存関係をキャッシュします:

    ```csharp
    public abstract class BaseDependency { }
    public class DerivedDependency : BaseDependency { }

    public partial class DependencyProvider
    {
        private BaseDependency dependency;

        protected override IReadOnlyDependencyContainer CreateChildDependencies(IReadOnlyDependencyContainer parent)
        {
            var dependencies = new DependencyContainer(base.CreateChildDependencies(parent));
            dependencies.CacheAs(dependency = new DerivedDependency());
            return dependencies;
        }
    }

    public partial class DependencyConsumer
    {
        [Resolved]
        private BaseDependency baseDependency { get; set; }       // OK

        [Resolved]
        private DerivedDependency derivedDependency { get; set; } // WRONG - will fail at runtime
    }
    ```

## ドローアブルクラスでの`[Cache]`

Drawable クラス自体に`[Cached]`属性の注釈を付けることができます。その場合、属性は、ドローアブルクラスのすべてのインスタンスがそのすべての子にキャッシュされるように解釈されます。

キャッシュでは宣言時点の型が使用されます。たとえば、次の構造があるとします:

```csharp
[Cached]
public partial class A : Drawable { }
                      
public partial class B : A { }
```

次のようなことが起こります:

- `A`のインスタンスは、タイプ`A`を使用して自身を子にキャッシュします。
- `B`のインスタンスは、タイプ`A`を使用して自身を子にキャッシュします。
- `B`のインスタンスは、タイプ`B`を使用して自身を子にキャッシュしません。そのためには、`[Cached]`属性をタイプ`B`で繰り返す必要があります。

## ドローアブルクラスによって実装されたインターフェイスの`[Cache]`

上記のタイプベースのキャッシュの変形は、インターフェイス経由で利用できます。インターフェイスには`[Cached]`の注釈を付けることができます。すべての`Drawable`は、実装する`[Cached]`の注釈が付けられたすべてのインターフェイスタイプを使用して、自身をその子にキャッシュします。例として:

```csharp
[Cached]
public interface IFirstInterface { }

[Cached]
public interface ISecondInterface { }

public interface IThirdInterface : ISecondInterface { }

public partial class Dependency : Drawable, IFirstInterface, IThirdInterface { }
```

`Dependency`のすべてのインスタンスは:

- 自分自身を`IFirstInterface`として子にキャッシュします。
- 自分自身を`ISecondInterface`として子にキャッシュします (`IThirdInterface`を介して推移的に)。
- 自分自身を`IThirdInterface`として子にキャッシュ**しません** (クラスと同様に、`[Cached]`は明示的に配置された型でのみ有効です)