# デバッグオーバーレイ: ログオーバーレイ

# ログオーバーレイ

osu!frameworkは、[`osu.Framework.Input.FrameworkActionContainer`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Input/FrameworkActionContainer.cs)で定義されたピクチャ内デバッグオーバーレイを提供します。これらのオーバーレイの 1 つはログオーバーレイです。

## 概要

<kbd>Control</kbd>+<kbd>F10</kbd>を押すとログオーバーレイが切り替わり、[`osu.Framework.Logging.Logger`](https://github.com/ppy/osu-framework/blob/master/osu.Framework/Logging/Logger.cs)によって記録されたイベントが表示されます。[`Logger.Log`](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Logging/Logger.cs#L143)を呼び出すと、このストリームに表示できるメッセージが記録されます。

![](https://cdn.discordapp.com/attachments/318886668889227266/539702590347411456/Screen_Shot_2019-01-29_at_4.02.30_PM-1.png)

デフォルトでは、メッセージはレベル[`Verbose`](https://github.com/ppy/osu-framework/blob/91ddc390d745c742a43f31cdd53d5fd25d986dc5/osu.Framework/Logging/Logger.cs#L479)のロガーを使用して記録され、`DEBUG`プリプロセッサフラグを使用してビルドしない限り、ログオーバーレイから省略されます。詳細レベルよりも高いレベルでは、メッセージがこのオーバーレイを通じて表示されます。




