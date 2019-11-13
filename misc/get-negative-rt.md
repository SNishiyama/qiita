# はじめに

今回の記事では，タイトルの通り，`psychopy.event.getKeys()`を使う際に，**条件が揃うと**負の反応時間を取得できることを紹介します。基本的に負の反応時間は実験作成者が求めているものではないので，それが得られてしまって「バグかな。困ったな。」というときに参考になれば幸いです（自分が困ったことがある）。ポイントは，負の反応時間がほしくなかったら`event.clearEvents()`を忘れるなということです。

# 方法

キーを押してから反応時間計測用の`core.Clock`オブジェクト（例えば`stopwatch`という変数名）に対して`.reset()`を実行し，`psychopy.event.getKeys(timeStamped=stopwatch)`すると負の反応時間が得られます。実際に負の反応時間が得られるコードを見ていきましょう。

## コード例 1

```python:get-neg-rt_1.py
from psychopy import visual, core, event

win = visual.Window()
stopwatch = core.Clock()
stim = visual.TextStim(win)
l_letter = ['a','b','c']

for letter in l_letter:
    stim.setText(letter)
    stim.draw()
    win.flip()
    core.wait(2) # 刺激の提示中にキーを入力する
    stopwatch.reset() # キーの入力後に時計をリセット
    resp = event.getKeys(timeStamped=stopwatch) # キー入力の処理
    print(resp)

win.close()

# 出力例
# [['space', -0.17376430198783055], ['space', -0.16929624899057671]]
# [['space', -0.19804733199998736]]
# [['space', -0.19725568499416113]]
```

`core.wait()`と`stopwatch.reset()`を逆にすれば負の反応時間は得られなくなります（正の値だとしても依然として正しく反応を取得できていませんが，これについては別記事で紹介する予定です）。ただ，このようなコードを書く人はいないと思います。刺激を提示する前（試行の最初）か直後（つまり`core.wait()`よりも前）に時計をリセットするはずです。しかし，次の例は恥ずかしながらも実際に自分が犯してしまったミスになります。

## コード例 2

```python:get-neg-rt_2.py
from psychopy import visual, core, event

win = visual.Window()
stopwatch = core.Clock()
stim = visual.TextStim(win)
l_letter = ['a','b','c']

for letter in l_letter:
    stim.setText(letter)
    resp = []
    # event.clearEvents()
    stopwatch.reset()
    for n_frame in range(120): # 120f(リフレッシュレートが60hzなら2秒)
        stim.draw()
        win.flip()
        if not resp:
            resp = event.getKeys(timeStamped=stopwatch) # キー入力の処理

    print(resp)

win.close()

# 出力例
# [['space', 0.8996025759843178]]
# [['space', -0.34911786299198866]]
# [['space', -1.4165731929824688], ['space', -0.6160959279804956]]
```

このコードは，上のコードと同じく刺激を2秒間提示している間の反応を取得しています。違いとしては，各試行の一番初めの反応だけ取得することを意図しています。刺激の提示前に`resp = []`とし，`resp`が空リストのままだと，`not resp`は`True`になるので，`event.getKeys()`が実行されます。キー入力があれば，`resp`に`[[key, rt]]`が代入されて，以降のループでは`not resp`は`False`になり，キー入力は処理されなくなります。
さらに，例1とは異なり提示のブロックの前に`stopwatch.reset()`を実行しているので，一見何の問題もなさそうです（自分はないと思っていました）。しかし，ある刺激の提示中にキーを2回以上入力すると出力例のように負の反応時間が得られます。これを解決するためには，コード中でコメントアウトしてある`event.clearEvents()`を有効にすればいいです。

# なぜこんなことが起こるのか

最近ようやく気づいたことなのですが，（少なくとも）psychopyにはキー（やマウス，ジョイスティック）の入力それぞれに対応する`event buffer`というものがあり，そこで入力が保持されているようです。これは`core.wait()`を実行しているときにも行われています。
`event.getKeys()`を実行すると，そのとき**keyboard** buffferに保持されている入力をすべて取り出します（[公式リファレンス](https://www.psychopy.org/api/event.html#psychopy.event.getKeys)）。そして，`timeStamped`を指定していると，bufferで記録されている時間（unix時間）と，指定した`core.Clock`オブジェクト（コード例の`stopwatch`）が示す時間との差分について，`.reset()`したタイミングを0として計算しなおした値を返してくれます。こうして反応時間が得られます。
これらの理由から，例1のようにキー入力から`getKeys()`までの間に`.reset()`が挟まっていると負の反応時間が得られますし，例2のように1つ目の反応しか取り出していないかのように見えても，同じ試行の2つ目以降の反応はbufferに保持されているので，それらは次のループの`getKeys()`で処理されることになります。キー入力自体は`.reset()`よりも前なので返される時間は負になってしまうということです。
例2の解決策として提示した`clearEvents()`はbufferに保持されている入力を取り除きます。これを各試行の開始直前に実行させることで前の試行で発生した2つ目以降の反応が次の試行に影響することはなくなります。なお，`getKeys()`が実行されても，それまでのキー入力はbufferからなくなります。

例1では`core.wait()`と`.reset()`並び替えれば済む話でしたが，だからといって`core.wait()`を使う場合が常に安全というわけではありません。以下のコードのように，試行間のブランクを設けた際に，そのブランクでキーを押すと負の反応時間が得られます。この場合も，`event.clearEvents()`を`core.wait()`（か`win.flip()`）の前に実行することで負の反応時間を防げます。いずれにせよ，`event.clearEvents()`を忘れないことが重要です。

```python:get-neg-rt_3.py
from psychopy import visual, core, event

win = visual.Window()
stopwatch = core.Clock()
stim = visual.TextStim(win)
l_letter = ['a','b','c']

for letter in l_letter:
    # event.clearEvents()
    stim.setText(letter)
    stim.draw()
    win.flip()
    stopwatch.reset() # キーの入力後に時計をリセット
    core.wait(2) # 刺激の提示中にキーを入力する
    resp = event.getKeys(timeStamped=stopwatch) # キー入力の処理
    print(resp)
    # ITI
    win.flip()
    core.wait(1) # ここでキーを押すと負の値が得られる

win.close()
```

ちなみに，`event.waitKeys()`は実行時にデフォルトで`event.clearEvents()`が内部で実行されるようになっているので，わざわざ明記する必要はないです（[公式リファレンス](https://www.psychopy.org/api/event.html#psychopy.event.waitKeys)）。

# おわりに

本記事では，`psychopy.event.getKeys()`が負の反応時間を返す場合について紹介しました。負の反応時間は`if not resp:`の条件式で最初の反応しか取得しないようにしてるから大丈夫だろうという自分の驕り・慢心から生まれたバグだったようです。event bufferについてはどこかでも解説されていたような気がしますが（十河先生の本？），このようなバグり方の観点から説明した記事はなかったのではないでしょうか。実験によっては負の反応時間も役に立つことがあるかもしれません。
`event.clearEvents()`は反応の取得における基本の「き」です。みなさんも慣れてきた頃に忘れないようにしてください。
