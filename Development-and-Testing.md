# 開発とテスト

プロジェクト内のコンポーネントに対するすべての初期開発 (および適用されるさらなる改善) は、VisualTests 環境で開発する必要があります。これを行うことにはいくつかの利点があります。

# ホットリロード

テスト シーンは、[.NET ホットリロード](https://devblogs.microsoft.com/dotnet/introducing-net-hot-reload/)メカニズムと統合されています。これが実際にどのように見えるかの例を次に示します。

https://github.com/ppy/osu-framework/assets/20418176/a3598d28-5347-45a2-9ef7-43fab5895a83

ホットリロード機能にはいくつかの制限があり、変更によってはアプリケーションを完全に再起動する必要があることに注意してください。詳細については、[Edit & Continue (EnC) でサポートされる編集](https://github.com/dotnet/roslyn/blob/2a9938ad11e432fdd274353f57083d4f6f83edf8/docs/wiki/EnC-Supported-Edits.md)を参照してください。

# ステップと自動テスト

`AddStep`関数を使用して、実行する必要がある自動ステップを追加できます。アサートをステップとして追加することもできます。これらのステップは、ユーザーが対話型環境で実行できますが、ヘッドレス実行でも実行できるため、CI プロセスのテストが可能になります。

追加できるステップの種類は以下のとおりです:

* [`AddStep()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L290) メソッドを実行するステップを作成します。例外がキャッチされなければ正常に完了します。
* [`AddRepeatStep()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L322) メソッドを指定された回数実行するステップを作成します。例外がキャッチされなければ正常に完了します。
* [`AddToggleStep()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L330) フラグを切り替えます。
* [`AddWaitStep()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L346) 次のステップに進む前に、指定された時間待機するステップを追加します。
* [`AddSliderStep()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L354) 設定値を調整するスライダーバーを作成するステップを追加します。
* [`AddAssert()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L362) 指定された値が true を返さない場合に失敗するステップを作成します。
* [`AddUntilStep()`](https://github.com/ppy/osu-framework/blob/b4c2a61f7e40d288611d587294a78fbcf68342fd/osu.Framework/Testing/TestScene.cs#L338) 条件が true になるまで、またはタイムアウトすると失敗するまで実行を試みるステップを追加します。

`AddAssert()`と`AddUntilStep()`は両方とも、[NUnitアサーション制約モデル](https://docs.nunit.org/articles/nunit/writing-tests/assertions/assertion-models/constraint.html)をサポートしています。つまり、次のとおりです:

```csharp
AddAssert("no text is selected", () => textBox.SelectedText, () => Is.Empty);
```

NUnit マッチャーは、単純な true/false アサーションと比較して、失敗した場合に大幅に改善されたデバッグ出力を提供するため、このスタイルの使用を強くお勧めします。

# 注意事項

* デバッガが接続された状態でアプリケーションが起動された場合、ホットリロードは効果がありません。起動後にデバッガを接続し、ホットリロードで変更を適用しようとすると、エラー`0x80131c69`(`CORDBG_E_ASSEMBLY_UPDATES_APPLIED`) が発生して失敗します。
* デバッガーが接続されている場合、`UntilSteps`がタイムアウトになることはありません。これは、until ステップでの失敗条件のデバッグを改善するための便利な機能です。