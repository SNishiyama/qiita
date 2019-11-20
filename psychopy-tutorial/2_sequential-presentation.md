# はじめに

本記事は，「PsychoPy Coderによる心理学実験作成チュートリアル」の第2回の記事です。[第1回](https://qiita.com/snishym/items/5c33710aca0fea3d41ec)ではPsychoPyをインストールし，とりあえず刺激を提示することを体験しました。今回は，刺激の系列提示およびリスト・for文について解説します。詳細は右側の見出しリストをご覧ください。

このチュートリアルシリーズの目的・概要等が気になった方はこちらの[全体のまとめ](https://qiita.com/snishym/items/8b52db0d901cf5744463)をご一読ください。

# サイモン課題とは

サイモン効果は，求められた反応と位置とが競合する場合に，競合しない場合と比べて反応が遅くなる効果です。例えば，左手のキーでの反応が求められた刺激が右側に提示されると，その刺激が左側に提示される場合と比べて反応が遅くなります。詳しくは文献等を参照してください。

本シリーズのサイモン課題では，「L」「R」という2つの文字を刺激として用いることにします。

# 刺激の素朴な系列提示

LとRを2回ずつ提示してみましょう。以下のコードをCoderに書いて実行してみてください。

```python:simple_sequence.py
from psychopy import visual, core

win = visual.Window()

letter_stim = visual.TextStim(win) # 1. テキストを指定せずに刺激を準備する

letter_stim.setText("L") # Lを指定 
letter_stim.draw()
win.flip()
core.wait(1)

letter_stim.setText("R") # Rを指定
letter_stim.draw()
win.flip()
core.wait(1)

letter_stim.setText("L") # Lを指定 
letter_stim.draw()
win.flip()
core.wait(1)

letter_stim.setText("R") # Rを指定 
letter_stim.draw()
win.flip()
core.wait(1)

win.close() # 画面を閉じる
```

きちんとコピペをすることができれば，問題なく実行できたと思います。

ただ，このコードは実際の実験のコードとしては全く使えません。この例ではLとRを2回ずつ計4回の提示だったからコピペで対応できました。しかし，実際の実験ではもっと多くの試行を実施するはずです。例えば100試行実施するとして，同様の作業をするべきでしょうか。もちろん，そんなはずはありません。100試行分のコピペした後に，提示時間を変更するとなったら100箇所修正する必要があります[^1]。提示のタイミングで何かの反応（e.g., キー押し）を取得することに変更になったら，同様に100箇所分修正する必要があります。そもそも，コピペがちょうど100個かどうか確認する必要があります。プログラミングでわざわざこんなことをする道理はありません。

[^1]: この問題自体は提示時間を代入した変数（e.g., `duration = 1`）を作成すれば解決しますが。。。

for文を使うことで，刺激の系列提示をその数に左右されることなく記述することができます。

# リストとfor文

さて，さっそくfor文を説明したいところですが，for文の処理にはリストというデータ型が利用されます。そのため，まずリストを紹介した後，for文の説明を行います。
リストは以下のように`[]`で複数の要素をまとめたものです。カンマ`,`で各要素を区切ります。

```python:list_example.py
num_list = [1,2,3,4]
text_list = ["a","b","c","d"]
mixed_list = [1,"a",3,4]
```

リストの中身は数字，文字列，さまざまなデータが入ります。また，3つ目の例にもあるように，同じリストの中に数字や文字列などの「型」の異なった要素をまとめることもできます。

for文はこのリストの要素を一つ一つ取り出して，それぞれの要素に対して順番にブロック内の処理を実行していきます。

```python:for_example.py
num_list = [1,2,3,4]

for i in num_list: # num_listから順番に値を取り出して，ブロック内でiという名前で利用する
    print(i * 100) # 取り出した値（i）に100をかける
```

以上の処理がfor文の使い方になります。なお，上の例を実行した時の結果はCoderの下部に出力されます（出力というタブ）。
for文を書く際の注意点は以下の2点です。

1. `for`と書いた行の末尾には必ず`:`をつける。
2. ブロック内の処理はインデント（**半角スペース4つ分**）を下げて入力する

1点目は守られていないと必ずエラーになります。`:`は気づきにくいくせにないとエラーになるので，エラーになった場合はまず`:`があるかどうかを確認しましょう。2点目については以下のコードを実行してみると実感しやすいと思います。

```python:for_inout.py
num_list = [1,2,3,4]

for i in num_list: 
    print("inside") # 4回実行される

print("outside") # 1回実行される
```

実行してみると，inside が4回出力された後，outside が1回出力されたはずです。`print("inside")`は4文字分字下げされているため，forブロックの内部と認識され，`num_list`の要素の数，つまり4回実行されます。一方で，最終行の`print("outside")`は字下げされていないので，forの**ブロック外**と認識された，1回だけ実行されました。

それでは，`print("outside")`を4文字分字下げしたらどうなるでしょうか？ぜひ試してみてください。

# for文を使った系列提示

最初のコードと同様に，for文を使用して刺激（L，R）を2回ずつ提示してみましょう。

```python:sequential_presentation.py
from psychopy import visual, core

win = visual.Window()

letter_stim = visual.TextStim(win) 
letter_list = ["L", "R"] * 2 # L, Rが2個ずつのリストを作る。リストには自然数nを掛けて要素の数をn倍することができる。

for letter in letter_list: # letter_listから順番に要素を取り出して，ブロック内でletterという名前で使用する
    letter_stim.setText(letter) # 取り出した要素を刺激テキストとしてセットする
    letter_stim.draw()
    win.flip()
    core.wait(1)

win.close()
```

最初よりもかなり簡潔になったと思います。念のため，2点補足です。

1. `for なんちゃら in リスト:` のなんちゃらにはどんな名前でも使用することができます[^2]。よくサンプルでは`i``j``k`などのアルファベット1文字が使われますが，そうしないといけないわけではありません。実際にコードを書く際にはあとで見返した時に分かりやすい名前をつけましょう（もちろんアルファベットで）。
2. `.setText()`は字のままですが，表示するテキストを設定し直すための関数（メソッド）です。

[^2]: ただし，同じコード内で他に使用している名前は避けてください。

上記のコードでは，一箇所変更するだけで試行数を50にも100にもすることができます。ぜひ該当箇所を変更して試してみてください。

# おわりに

今回は刺激を系列提示する方法を紹介しました。そのためにリストとfor文を利用しました。for文はリスト内の要素それぞれに対して同じ処理を実行するための構文です。繰り返しの処理が得意なプログラミング言語らしい構文だと思います。

しかし，刺激は画面中央にしか提示されていません。サイモン課題の一致条件・不一致条件を作り出すために，次回では刺激の提示位置を変更する方法を紹介します。

[【第3回】刺激の提示位置の調整](https://qiita.com/snishym/items/1bdf90cf51ba28cc4ef3)