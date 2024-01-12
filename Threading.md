# Threading

スレッドに関しては、osu!framework ゲームは 2 つのモードで実行できます。

* マルチスレッド実行モードでは、ゲームは4つのメインスレッドで実行されます:

    1. 入力スレッド:
        - 通常は1000Hzで動作
        - 使用されるウィンドウツールキットとのインターフェイス (デフォルトでは SDL2、osuTK も利用可能)
        - 入力イベントの処理とコンポーネント/ドローアブルへの転送を担当。
    2. オーディオスレッド:
        - 通常は1000Hzで動作
        - BASSオーディオライブラリを介してOSのオーディオサブシステムを持つインターフェイス。
    3. 更新スレッド:
        - 通常、フレームレート制限の2倍で実行される (これは、`FrameSync`フレームワーク設定または <kbd>Ctrl</kbd>+<kbd>F7</kbd>ショートカットを使用して切り替えることができる)
        - ゲームのドローアブル階層内のすべてのコンポーネントの更新コード (`updateSubTree()`、`Update()`) を実行する責任がある。
    4. 描画スレッド:
        - 指定されたフレームレート制限で実行される
        - 描画呼び出しをGPUにディスパッチする (現在はOpenGLを使用しているが、将来的にはさらに多くのバックエンドが予定されている)

* シングルスレッド実行モードでは、上記の4つのスレッドが1つのOSスレッド上でインターリーブ方式で実行されます。

スレッドモードは、`ExecutionMode`フレームワーク設定を使用するか、<kbd>Ctrl</kbd>+<kbd>Alt</kbd>+<kbd>F7</kbd>ショートカットを使用して設定できます。

どちらのモードでも、`CompositeDrawable.LoadComponentAsync()`によるドローアブルの非同期読み込み ([非同期読み込みに関するドキュメント](./Asynchronous-loading.md)を参照) や.NET ランタイムのスレッド プールで実行されている`Task`などの他のタスクが追加の OS スレッドを使用できる可能性があることに注意してください。

# Scheduling

Relatively often, there is a need to change the state of a `Drawable` or `Component` in response from a callback which is not executing on the game's update thread.
An example of this would be updating the text and transforms to a `SpriteText` in response to a web request.
To ensure that such an operation can be executed in a safe manner, each `Drawable` or `Component` has an internal `Scheduler`, that can collect such pending operations and execute them at the correct time:

```csharp
request = CreateRequest();
request.Failure += exception => Schedule(() =>
{
    errorMessageSpriteText.Text = exception.Message;
    errorMessageSpriteText.FadeTo(Colour4.Red).Then().FadeTo(Colour4.White, 1000);
});
```

While the above use case is to be considered the primary one for schedulers, they can also be used to handle a variety of other tasks, as well:

* If a drawable - for whatever reason - is not being actively updated (due to being not present, masked away, not loaded yet), scheduling a task onto it ensures that the task may execute on the nearest possible occasion when the drawable is back in the active scene graph.
* The return value of a `Schedule()` call is a `ScheduledDelegate`, which can be later cancelled. This can be used to enqueue operations to be executed at some point in the future, but still be able to early-cancel them if some other condition is met.
* Aside from `Schedule()`, the `Scheduler` itself is also exposed as protected on each drawable, and provides additional capabilities, such as:

    - `Scheduler.AddOnce(task)` can be used to ensure that an operation runs at most once per frame.
      This is most often useful when called from an input event handler (whether mouse, keyboard, or any other peripheral), to avoid performing needless updates.

    - `Scheduler.AddDelayed(task, timeUntilRun)` can be used to run an operation at a set time in the future, relative to now.
      That method also has a `repeat` argument that can be used for recurring operations (they will execute regularly every `timeUntilRun` milliseconds).

* As a last-ditch attempt, schedulers can sometimes also be used to fix various one-frame issues, such as auto-size containers flickering when their contents are being changed.
  This sort of usage is however discouraged, as it can cause headaches down the line due to complicating interactions between components.

In general schedulers are a powerful tool, but ones that should be used carefully, as they change the usual flow of operations and therefore delay their effects.
Nesting schedules, or having chains of scheduled operations will complicate debugging significantly.

## Audio components

Note that when using `AudioComponent` and its derived classes (`SampleChannel`, `Track`) there is no inherent need to schedule operations like `Play()` or `Stop()`. The components themselves internally ensure that the operations to be executed are correctly enqueued onto the audio thread.

# Asynchronous disposal

All drawables/components in a game, upon being removed from their parent `CompositeDrawable`s, are enqueued onto a game-global async disposal queue.
The queue uses TPL threadpool threads to run each of the drawables' `Dispose()` methods.

To suppress this behaviour, methods such as `CompositeDrawable.Remove()` or `CompositeDrawable.Clear(false)` can be used to remove a drawable from the hierarchy without calling its `Dispose()` implicitly.

# Setting thread culture

To change all of the main game threads to a new culture, you can set the `Locale` framework setting to the name of the desired locale, which should correspond to the value of `CultureInfo.Name` of the desired culture.
Doing this will change culture on all of the main threads, as well as set the default culture for any new threads.
Some existing threads not managed by the framework may still continue operating using the old culture, however.