# はじめに

本記事は，「PsychoPy Coderによる心理学実験作成チュートリアル」の第4回の補足記事です。[第4回](https://qiita.com/snishym/items/0b10e714363656094181)では`event.waitKeys()`を利用したキー反応の取得を紹介しました。この補足では，`event.waitKeys()`を使わないキー反応の取得方法について紹介します。合わせて`for`文を利用した刺激の提示方法についても紹介しています。

このチュートリアルシリーズの目的・概要等が気になった方はこちらの[全体のまとめ](https://qiita.com/snishym/items/8b52db0d901cf5744463)をご一読ください。

# `event.waitKeys()`では困る場面

`event.waitKeys()`は反応があるまで処理を待機する関数になります。本チュートリアルのサイモン課題のように利用すれば，参加者のキー反応を待ち，反応が得られるとともに次の試行に進みます。しかし，反応とともに次の試行に移ってしまうため，**反応の有無に関わらず刺激を一定時間提示したり，刺激の提示中に複数回の反応を取得したりしたいという場合**には`event.waitKeys()`は利用できません。
そのような場合に有用なキー反応取得方法として今回は

1. `core.wait()` + `event.getKeys()`
2. `for` + `event.getKeys()`

の2つを紹介します。1つ目のほうがシンプルですが，場合によっては2つ目のほうが便利です。

# `core.wait()` + `event.getKeys()`

この組み合わせの要点は，処理を待機している間にもこっそりキー入力の処理をしておくと言う感じです[^1]。`core.wait()`は[第1回](https://qiita.com/snishym/items/5c33710aca0fea3d41ec)，[第2回](https://qiita.com/snishym/items/31c980f08e62190469f9)，[第3回](https://qiita.com/snishym/items/1bdf90cf51ba28cc4ef3)で出てきました。`()`内に指定した秒数だけ処理を待機する関数です。
`event.getKeys()`は名前の通りキー入力を取得する関数です。ただし，`event.waitKeys()`と違い，キー入力を待たないので，この関数を単独で使っても一瞬で次の処理に進んでしまい，キー入力を取得するのはかなり難しいです。次のコードを実行してみてください。

[^1]: この方法はwindow描画のバックエンドで`pyglet`を利用する場合にしか利用できないようです（[公式リファレンス](https://www.psychopy.org/api/core.html#psychopy.core.wait)）。描画のバックエンドはPsychoPyの設定>一般>winTypeで確認できます[十河先生のサイト](http://www.s12600.net/psy/python/18-2.html)（ちょっと古いですが参考になるはずです）。デフォルトでは`pyglet`になっているはずです。現在利用できるバックエンドは[3種類あります](https://www.psychopy.org/api/visual/window.html#psychopy.visual.Window)。

```python:get_key_response_severe.py
from psychopy import visual, core, event

win = visual.Window()

instruction = visual.TextStim(win, "いずれかのキーを押してください")
instruction.draw()
win.flip()

resp = event.getKeys() # 押されたキーをrespという名前で利用する
print(resp) # 出力

win.close()
```

多くの人は`None`が得られたのではないでしょうか。そもそも一瞬で画面の提示が終わってしまったはずです。
このままでは全く使い物になりません。それでは`core.wait()`を組み合わせて5秒待つようにした以下のコードを実行して，だいたい1秒間隔でキー入力をしてみてください。なお，あとの説明のために反応時間も取得するようにしています。

```python:get_key_response_bad.py
from psychopy import visual, core, event

win = visual.Window()
instruction = visual.TextStim(win, "いずれかのキーを押してください")

stopwatch = core.Clock() # stopwatchという名前の時計を用意

instruction.draw()
win.flip()

stopwatch.reset() # 時計の時間をリセットする
core.wait(5) # 5秒待つ
resp = event.getKeys(timeStamped = stopwatch) # timeStampedに用意した時計を指定
print(resp)

win.close()
```

`[['space', 4.8015793999657035], ['space', 4.802112399949692], ['space', 4.80252439994365], ['space', 4.802825400023721]]`みたいな結果が得られたのではないでしょうか。確かに1秒，2秒，3秒，4秒時点で反応していたとしたら4つ反応が得られていることに間違いはないですが，出力されている秒数がどれもだいたい4.8秒ほどになってしまっています。全く意図したように動いておらず，これも使い物になりません。
それでは，次のコードを実行し，同様に1秒間隔でキー入力をしてください。

```python:get_key_response_good.py
from psychopy import visual, core, event

win = visual.Window()
instruction = visual.TextStim(win, "いずれかのキーを押してください")

stopwatch = core.Clock() # stopwatchという名前の時計を用意

instruction.draw()
win.flip()

stopwatch.reset() # 時計の時間をリセットする
core.wait(5, hogCPUperiod=5.0) # 5秒待つ
resp = event.getKeys(timeStamped=stopwatch) # timeStampedに用意した時計を指定
print(resp)

win.close()
```

いい感じに反応を取得できたのではないでしょうか。先程のコードとの違いは， `core.wait()`で`hogCPUperiod=5.0`としている点です。`core.wait()`は最初に指定した秒数だけPsychoPyの処理を止めているのですが，その次に`hogCPUperiod=秒数`という引数を指定することで，待ち時間の終了何秒前からバックグラウンドの処理を始めるかを決めることができます。この引数を明示的に設定しない場合はデフォルト値として`0.2`が使用されるようになっています。そのため，先程のコードでは，待ち時間5秒が終了する0.2秒前つまり，4.8秒時点から処理が再開され`event.getKeys()`が実行されたため，反応時間が4.8秒程度だと出力されたということになります。
一方で，今回のコードでは`hogCPUperiod=5.0`と指定してバックグラウンドの処理を待機時間終了5秒前，つまり待機時間中常にバックグラウンドで`event.getKeys()`の処理を行っていたため，意図したキー入力のタイミングの反応時間が取得されたということになります[^2]。
さて，決まった時間だけ刺激を提示しながら反応を取得するという目的を非常にシンプルなコードで達成することができました。次節では，`for`+`getKeys()`で同様のことを実現する方法を紹介します。多くの場合`core.wait()`+`getKeys()`で十分かもしれませんが，たまに凝ったことをしようとすると応用が効きにくいです。例えば，「方向キーの入力で提示刺激を動かしたい」という場合には，`for`をつかった方が楽です（ぜひご自身で試してみてください）。

[^2]: 厳密には違うはずですが，直感的には納得できる説明だと思います。

# `for` + `event.getKeys()`

この組み合わせの要点は，`for`ループで刺激を提示している間に`event.getKeys()`で反応を取得するというものです。そういえば，本チュートリアルシリーズでは一貫して`waitKeys()`を利用していたため，`for`文で刺激を提示する方法をそもそも説明していませんでした。まずはこれについて説明します。

```python:show_text_for.py
from psychopy import visual

win = visual.Window()
text = visual.TextStim(win, 'こんにちは')

for _ in range(300): # 300回win.flip()を繰り返す。60hz（1秒に60回）のディスプレイなら刺激が5秒間提示される。
    text.draw()
    win.flip()

win.close()
```

上のコードを実行すると，5秒間刺激が提示されたはずです。このコードでは`core.wait()`のように処理を止めるのではなく，むしろ，`draw()`による刺激の描画と`win.flip()`による画面の更新を`for`で繰り返すことで5秒間刺激を提示し続けています。`win.flip()`はディスプレイの画面の更新に合わせて実験画面（`win`）の更新を行います。最近の標準的なPCディスプレイの画面の更新（[リフレッシュレート](https://www.google.com/search?&q=%E3%83%AA%E3%83%95%E3%83%AC%E3%83%83%E3%82%B7%E3%83%A5%E3%83%AC%E3%83%BC%E3%83%88+windows10)）は60hz（1秒間に60回）がデフォルトですので，更新を300回行えば5秒刺激を提示しているということになります。
これに`getKeys()`によるキー入力処理を追加すれば`core.wait()` + `getKeys()`と同様のキー入力の取得を行なうことができます。

```python:get_key_response_for.py
from psychopy import visual, core, event

win = visual.Window()
instruction = visual.TextStim(win, "いずれかのキーを押してください")

stopwatch = core.Clock()

resp = [] # 反応を保存するための空のリストを作成
stopwatch.reset()
for _ in range(300): # 300回ディスプレイの更新を繰り返す。60hz（1秒に60回）のディスプレイなら刺激が5秒間提示される。
    instruction.draw()
    win.flip()
    resp += event.getKeys(timeStamped=stopwatch) # respにキー入力をどんどん追加していく

print(resp)

win.close()
```

`core.wait()` + `getKeys()`の場合と異なるのは，`resp = []`と事前に宣言している点と`resp += event.getKeys()`で`+=`としている点です。`+=`を使わず，`resp = event.getKeys()`だと，キー入力が取得されるたびに`resp`の値が更新されてしまいます。これを防いで，複数の反応を取得するために事前に反応を入れる容器となる空のリストを`resp = []`で用意しておき，`+=`でそこに反応を追加していくようにしています。今紹介した変更点があるため，`core.wait()`を使う場合よりも面倒ですが，こちらのほうが刺激の提示に工夫が必要な場合に対応しやすいです。

# `event.clearEvents()`を忘れない

さて，単一の試行における反応の取得は以上になりますが，実際の実験課題では複数の試行が実施されます。`event.getKeys()`を使う際には各試行ブロックの最初に`event.clearEvents()`を実行するようにしましょう。これは，1つ前の試行でのキー入力が次の試行で処理されてしまわないようにするために重要です。`getKeys()`の処理から漏れてしまったキー入力は[次の試行の反応計測に悪影響を及ぼすことがあります](https://qiita.com/snishym/items/322e37ab2fd68d379303)。

```python:get_resp_task.py
from psychopy import visual, core, event

win = visual.Window()
letter_stim = visual.TextStim(win)
letters = ["L","R","L","R"]

stopwatch = core.Clock()

for letter in letters:
    letter_stim.setText(letter)

    resp = []
    event.clearEvents() # <--------- これ！！！！
    stopwatch.reset()
    for _ in range(120):
        letter_stim.draw()
        win.flip()
        resp += event.getKeys(keyList=['left', 'right'], timeStamped=stopwatch)

    print(resp)

win.close()
```

この例では`for`文を使った処理を用いていますが，`core.wait()`を使う場合にも気をつけてください。ちなみに，`event.waitKeys()`はデフォルトでその実行と同時に`event.clearEvents()`をするようになっているので書く必要はありません。

# おわりに

今回は`event.getKeys()`を使わないキー反応の取得方法を紹介しました。合わせて`for`文を利用した刺激の提示方法についても紹介しました。これらを利用することでご自身の実験手続きに合わせて柔軟に刺激の提示・反応の取得方法を設定できると思います。

[【第5回】データの保存](https://qiita.com/snishym/items/f80607cbe462f8a4e1d4)
