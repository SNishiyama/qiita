# はじめに

本記事は，「PsychoPy Coderによる心理学実験作成チュートリアル」の第1回の記事です。今回はPsychoPyをインストールし，とりあえず刺激を提示してみます。詳細は右側の見出しリストをご覧ください。

このチュートリアルシリーズの目的・概要等が気になった方はこちらの[全体のまとめ](https://qiita.com/snishym/items/8b52db0d901cf5744463)をご一読ください。

# 下準備

### PsychoPyのインストール

[公式のダウンロードページ](https://www.psychopy.org/download.html)のトップにあるボタンをクリックしてダウンロードしてください。
ダウンロードが終了したらインストーラーを起動し，指示通り進めてください。結構時間がかかります。

インストールを待っている間に次の2つの準備を済ませておきましょう。

### 練習用のフォルダを作成

デスクトップに`psychopy-practice`みたいな名前で練習用のフォルダを新規作成してください。

### ファイルの拡張子を表示する

https://pc-karuma.net/windows-10-show-explorer-file-name-extension/
を参考にファイルの拡張子が表示されるようにしてください。リンク先のページでは拡張子についても簡単に説明があります。

# Coderを開く

PsychoPyを起動すると，Builder（下図左）とCoder（下図右）の両方のウィンドウが開きます。
![psychopy_builder-coder.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/459905/8b8e68c7-533a-c73b-5b01-bc6f90d5b760.png)

今回はCoderを使うので，Builderはそっと閉じましょう。

### 新規ファイルの作成

Coderの左上にある紙っぽいアイコンをクリックすると新しいファイルを作成できます。

# とりあえずPsychopyを使ってみる

### ウィンドウを表示する

新しいファイルに以下のコードを入力し，ファイルを保存してください（`control + s`またはフロッピーのボタンを押して保存できます）。初回はファイル名を要求されるので**後で見返したときに分かりやすい名前にしましょう**。例えば，下のコードエリアの左上に書いてあるように，`show_window`と付けて保存してください。少なくとも，**半角英数字にしてください**。

そして，ウィンドウの上部にある緑色のボタンをクリックしてください。コードが実行されます。

```python:show_window.py
from psychopy import visual, core # 1. 必要なツールの指定

win = visual.Window() # 2. 画面を表示する
core.wait(3) # 3. 3秒待つ

win.close() # 4. 画面を閉じる
```

灰色のウィンドウが表示されて，3秒くらいで消えたと思います。エラーもないのにウィンドウも表示されなかった場合は，もしかしたらCoderのウィンドウに隠れているかもしれません[^1]。

[^1]: エラーが出る場合は，コードの大文字小文字が違っていないか確認してください。

上記のコードは4行で構成されていて，それぞれ

1. `psychopy`という道具箱（パッケージと言います）から`visual`と`core`を取り出す
2. `win`という名前のウィンドウを用意・表示
3. 3秒待機
4. ウィンドウを閉じる

という処理をしています。それぞれの行の右のほうに書いているコメントの通りですね。なお，Pythonでは，`#`に続けてメモを書くことができます[^2]。他の人や将来の自分のために，なるべくメモを残しておくようにしましょう。

[^2]: もう少し詳しく言うと，`#`より右側はコードとして認識・処理されません。したがって，メモを書くときだけでなく，一時的にある行のコードを実行したくない場合にも`#`は便利です。行頭に`#`をつけて行ごとコメント化し，処理を回避できます（コメントアウトと言います）。Coder上では，`control + /`で（複数行でも）コメントアウトができます。

### ウィンドウに文字を提示する

新しいファイルを作成して，以下のコードを書いてください。ご覧の通り，`win = visual.Window()`と`core.wait(3)`の間に3行増えただけです。

ファイルを（例えば）`show_text`という名前で保存して，緑のボタンを押してください。

```python:show_text.py
from psychopy import visual, core # 必要なツールの指定

win = visual.Window() # 画面を表示する

hello = visual.TextStim(win, "こんにちは，世界！") # 1. 文字刺激の準備
# hello = visual.TextStim(win, "こんにちは，世界！", font = "Hiragino Kaku Gothic Pro W3") # Macの場合

hello.draw() # 2. 刺激の描画
win.flip() # 3. 画面に反映

core.wait(3) # 3秒待つ

win.close() # 画面を閉じる
```



ウィンドウの中央に「こんにちは，世界！」と言う文字列が表示されたはずです[^1]。**Macの場合，デフォルトのフォントでは日本語の表示が崩れてしまうので，上記コードのようにフォントを指定してください。以降の記事ではこれについて言及していませんが，日本語を提示する際は注意してください**。

新たに増えた行の処理はそれぞれ，

1. 「こんにちは，世界！」という文字刺激（`TextStim`）を`hello`という名前で準備
2. 刺激の描画
3. 画面に反映

となります。

### ウィンドウに画像を提示する

#### 下準備

提示するための画像を練習用のフォルダ（`psychopy-practice`）に用意します。このページの画像ファイル（CoderとBuilderの画像）を「右クリック>名前をつけて保存」から`builder-coder.png`という名前で保存してください。

#### コードを書く

新しいファイルを作成して，以下のコードを書いてください。

```python:show_image.py
from psychopy import visual, core

win = visual.Window()

photo = visual.ImageStim(win, "builder-coder.png") # 1. 画像刺激の準備

photo.draw() # 2. 刺激の描画
win.flip()

core.wait(3)

win.close()
```

ウィンドウの中央に画像が表示されたはずです。

直前の`show_text`から2点変更点があります。

1. 「こんにちは，世界！」画像刺激（`ImageStim`）を`photo`という名前で準備
2. `hello.draw()` —-> `photo.draw()` に変更（コード上での刺激の名前を`photo`にしたから）

1つ目は気付きやすいですが，2つ目を見落としてしまった人もいるかもしれません。それでもエラーが出る場合は，ファイルの名前や保存先が間違っているかもしれません。エラーメッセージをよく確認しましょう。

**2021/07/02更新**

一部端末で以下のようなエラーが表示されることがあるようです。エラーの下から4行目の右端に`pix2deg`や`pix2cm`とある場合は，psychopyの `設定 > 一般`の中の単位が `deg` や `cm` になってないか確認してください。そうなっている場合は，`norm` や `height` にすると正常に動作する可能性が高いです。

<details><summary>エラー例</summary><div>

```
################ Running: C:\PsychoPy\psychopy-practice\pic.py #################
pygame 1.9.6
Hello from the pygame community. https://www.pygame.org/contribute.html
3.4469     WARNING     Monitor specification not found. Creating a temporary one...
3.4492     WARNING     User requested fullscreen with size [800 600], but screen is actually [3440, 1440]. Using actual size
Traceback (most recent call last):
  File "C:\PsychoPy\psychopy-practice\pic.py", line 5, in <module>
    photo = visual.ImageStim(win, "builder-coder.png") # 1. 画像刺激の準備
  File "C:\Program Files\PsychoPy3\lib\site-packages\psychopy\visual\image.py", line 96, in __init__
    self.color = color
  File "C:\Program Files\PsychoPy3\lib\site-packages\psychopy\visual\basevisual.py", line 365, in color
    self.foreColor = value
  File "C:\Program Files\PsychoPy3\lib\site-packages\psychopy\visual\image.py", line 277, in foreColor
    self.setImage(self._imName, log=False)
  File "C:\Program Files\PsychoPy3\lib\site-packages\psychopy\visual\image.py", line 338, in setImage
    setAttribute(self, 'image', value, log)
  File "C:\Program Files\PsychoPy3\lib\site-packages\psychopy\tools\attributetools.py", line 141, in setAttribute
    setattr(self, attrib, value)
  File "C:\Program Files\PsychoPy3\lib\site-packages\psychopy\tools\attributetools.py", line 32, in __set__
    newValue = self.func(obj, value)
  File "C:\Program Files\PsychoPy3\lib\site-packages\psychopy\visual\image.py", line 328, in image
    self.size = None  # set size to default
  File "C:\Program Files\PsychoPy3\lib\site-packages\psychopy\tools\attributetools.py", line 32, in __set__
    newValue = self.func(obj, value)
  File "C:\Program Files\PsychoPy3\lib\site-packages\psychopy\visual\basevisual.py", line 1470, in size
    self.win.monitor)
  File "C:\Program Files\PsychoPy3\lib\site-packages\psychopy\tools\monitorunittools.py", line 269, in pix2deg
    raise ValueError(msg % monitor.name)
ValueError: Monitor __blank__ has no known size in pixels (SEE MONITOR CENTER)
##### Experiment ended. #####
```

</div></details>

### 複数の刺激を一度に提示する

最後に，複数の刺激を一度に提示します。新しいファイルを開いて，以下のコードを書いてください。ご覧の通り，`show_text`と`show_image`を組み合わせただけです。

ファイルを`show_text_and_image`という名前で保存して，緑のボタンを押して実行してください。

```python:show_text_and_image.py
from psychopy import visual, core

win = visual.Window()

hello = visual.TextStim(win, "こんにちは，世界！") # 1. 文字刺激の準備
photo = visual.ImageStim(win, "builder-coder.png") # 1. 画像刺激の準備

photo.draw() # 2. 刺激の描画
hello.draw()
win.flip()

core.wait(3)

win.close()
```

文字刺激と画像刺激がダダ被りだと思いますが，二つの刺激が表示されたはずです（画像の配色が暗めだと分かりやすい）。刺激の準備の時に，提示位置を指定すれば重複を防ぐことができますが，それは今後の記事で取り扱うので，今回は簡単のため割愛です。

ここで覚えて欲しいのは，`win.flip()`の前にそれぞれの刺激を`.draw()`すれば，一気に表示できるということです。`hello.draw()`と`photo.draw()`の順序を入れ替えるとどうなるかも確認してみてください。

# おわりに

以上で第1回は終了です。文字・画像刺激以外にも，図形や音声，リッカートスケールなども提示することができます。ただ提示するだけならどれも簡単なので，[公式リファレンス](https://www.psychopy.org/api/visual.html)を参考に提示してみてください。

次回は，実験課題の基本である刺激の系列提示を行います。

[【第2回】刺激の系列提示](https://qiita.com/snishym/items/31c980f08e62190469f9)
