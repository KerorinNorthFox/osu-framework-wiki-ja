# 座標およびレイアウトシステム(Coordinate and Layout System)

一度親の`Container`階層に追加されると、`Drawable`オブジェクトは、そのコンテナのピクセル座標空間を採用します。デフォルトでは、コンテナのサイズに関係なく、すべての位置およびサイズ変更は絶対ピクセル値によって静的です。そうは言っても、さまざまなウィンドウ サイズやデバイス画面に適切にスケーリングするために、相対的なサイズ変更を可能にするさまざまなメカニズムが存在します。

共通の位置への要素のレイアウトを自動化するために、`Drawable`は初期位置の決定に役立つ 2 つのプロパティを実装しています。

### `Anchor`
`Anchor`は、親コンテナに対する原点の位置を決定します。`Centre`は要素をコンテナの中央に配置しますが、`TopLeft`などの他のオプションは要素をコンテナの外側の端に沿って配置します。

### `Origin` 
`Origin`は、要素自体の原点からの要素のコンテンツのオフセットを決定します。例として`Centre`は、要素の原点がそのコンテンツの中央に位置することを意味します。`TopLeft`は、コンテンツの左上隅を原点にします。

<!-- <p align="center">
<a href="https://puu.sh/F6BMJ/23b52dcfcf.jpg"><img src="https://puu.sh/F6BMJ/23b52dcfcf.jpg" height="250" alt="Drawable" /><a/><a href="https://puu.sh/nnjI6/6a7db36ade.png"><img src="https://puu.sh/nnjI6/6a7db36ade.png" height="250" alt="osu! Drawable" /></a>
</p> -->
これら 2 つのプロパティを設定したら、追加のプロパティを使用してさらにカスタマイズできます。

* `Drawable`の`Position`を設定すると、`Drawable`が初期位置からオフセットされます。これは、要素を任意の方向にオフセットするために正と負の両方の値をサポートします。
* `Size`を設定すると、要素の幅と高さのサイズがピクセル単位で変更されます。要素の位置は、新しいサイズを考慮して、Anchor値とOrigin値を中心に更新されます。

```csharp
box.Position = new Vector2(100.0f, 50.0f);
box.Size = new Vector2(400.0f, 400.0f);
```

# 相対的なレイアウトとサイズ設定(Relative Layout and Sizing)

コンテナのサイズがフレキシブルな場合、`Drawable`の動作をオーバーライドして、絶対値ではなく相対値に依存することが可能です。

この動作を有効にするには、`RelativePositionAxes`および`RelativeSizeAxes`列挙プロパティにアクセスし、どの軸を相対にするかを設定します。

有効にすると、絶対ピクセル値の代わりに、相対値を示すために 0.0f から 1.0f までの値を設定できます。この場合、0.0f は親コンテナの 0% を表し、1.0f は 100% を表します。

```csharp
box.RelativePositionAxes = Axes.Both;
box.Position = new Vector2(0.2f, 0.5f);
```

# 絶対値を画面サイズに合わせて拡大縮小(Scaling Absolute Values to Screen Size)

ほとんどの場合、相対値を使用した作業は、絶対値に比べてはるかに曖昧で、作業が困難です。

柔軟な画面サイズを許可しながら絶対位置決めを可能にするために、osu-framework は `DrawSizePreservingFillContainer`と呼ばれるタイプの`Container`を提供します。

```csharp
new DrawSizePreservingFillContainer
{
    Child = box = new Box
    {
        Anchor = Anchor.TopLeft,
        Origin = Anchor.TopLeft,
        Colour = Color4.Orange,
        Size = new Vector2(200),
    }
};
```

このタイプのコンテナは、キャンバスのサイズを表す`TargetDrawSize`プロパティ (デフォルト値は`Vector2(1024, 768)`) を実装します。すべての子のサイズは、このコンテナのサイズに応じて設定できます。

親コンテナのサイズが変更されると、`DrawSizePreservingFillContainer`はその親を満たすようにサイズ変更しますが、同じスペースを満たすようにすべての子`Drawable`要素を適切に拡大または縮小します。

デフォルトでは、`DrawSizePreservingFillContainer`は、"scale to fit"戦略を使用して、コンテンツのアスペクト比を維持しながら、すべての子コンテンツを拡大縮小します。

* `Minimum` - アスペクト比スケールは、コンテナの現在のサイズの小さい方の寸法値によって決まります。これにより、すべてのコンテンツがオーバーフローせずにコンテナーに収まるように縮小されます。これはデフォルト値です。
* `Maximum` - アスペクト比スケールは、コンテナの現在のサイズの大きい方の寸法値に基づいています。これにより、利用可能な境界を満たすようにコンテンツが拡大され、一部のコンテンツが切り取られる可能性があります。
* `Average` - 最小スケールと最大スケールの間の平均値が計算され、適用されます。これは、前の両方の値の間の適切な妥協点です。
* `Separate` - アスペクト比はまったく適用されません。すべての要素はコンテナの寸法に応じて伸びたり変形したりします。