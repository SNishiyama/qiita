# はじめに

本記事は，「jsPsych による心理学実験作成チュートリアル」の第 7 回の記事です。[第 6 回](https://qiita.com/snishym/items/e0f82fa972970cda632c)では参加者情報の取得方法を紹介しました。そこで実験課題は完成しましたが，もう少し実験プログラムを作り込んで行きます。今回は課題の教示，課題の練習をこれまでのコードに追加します。

このチュートリアルシリーズの目的・概要等が気になった方はこちらの[全体のまとめ](https://qiita.com/snishym/items/1e0511f8622282993ed1)をご一読ください。

# 教示・長文テキスト

参加者が課題を行うにあたって，どのように進めればいいか説明する必要があります。といっても，これまで使い続けてきた`html-keyboard-response`を使えばいいだけです[^1]。

[^1]: ただし，教示文の改行や配置（左寄せ・中央寄せ）にこだわりがあったり，図なども教示に表示したいという場合は，パワーポイントなどで教示用の画像を作って`image-keyboard-response`で提示させたほうが楽かもしれません。html なので，自由なレイアウトが可能ですが使い方を勉強する必要があります。

```html:show_instruction.html
<!DOCTYPE html>
<html>
<head>
  <script src="../jspsych.js"></script>
  <script src="../plugins/jspsych-html-keyboard-response.js"></script>
  <link rel="stylesheet" href="../css/jspsych.css"></link>
</head>
<body></body>
<script>

  var instruction = {
    type: 'html-keyboard-response',
    stimulus: '教示です',
  }

  jsPsych.init({
    timeline: [instruction],
  });
</script>

</html>
```

このコードについて特に説明することはありません。これを課題の前に入れれば，教示に続けて課題が実行されます。しかしながら，「教示です」なんて教示はありません。もっと課題の概要・取り組み方を説明しなければなりません。

本シリーズで作成してきたサイモン課題は，まず「キー押しの課題です」。具体的には，「L と R がランダムに提示されます」。そして，参加者がしなければならないのは，提示された刺激が「『L』なら左手人差し指で f キーを押し，『R』なら右手人差し指で j キーを押す」ということです。適切に改行しながらこれを反映すると，`stimulus:`の部分は以下のようになります。`<br>`で改行することができます。

```javascript
var instruction = {
  type: 'html-keyboard-response',
  stimulus:
    'これからキー押し課題を行ってもらいます。<br>「L」「R」がランダムに表示されるので，<br><br>「L」が表示されたら左手人差し指で f キー<br>「R」が表示されたら右手人差し指で j キー<br><br> を押してください。<br>準備ができたらいずれかのキーを押して課題を始めてください。',
};
```

これで教示としては OK になりました。改行を 2 つ重ねていると空白行ができるので，段落などのまとまりを作って，見やすくしたりすることができます。しかし，実験を作っている立場からすると，1 行で書くのは見にくくて不便です。

解決方法の一つは，いくつかの文字列に分割して，`+`で連結するようにします。`+`を使っている部分ではコード上で改行することができるので，コードを見やすくすることができます。

```javascript
var instruction = {
  type: 'html-keyboard-response',
  stimulus:
    'これからキー押し課題を行ってもらいます。<br>' +
    '「L」「R」がランダムに表示されるので，<br><br>' +
    '「L」が表示されたら左手人差し指で f キー<br>' +
    '「R」が表示されたら右手人差し指で j キー<br><br>' +
    'を押してください。<br>' +
    '準備ができたらいずれかのキーを押して課題を始めてください。',
};
```

# 課題の練習

実際の実験では，分析用のデータを取る（つまり本番）の前に，練習課題を行います。参加者の教示の理解を確認したり，反応のマッピングを生成したりするために重要です。そこで，練習課題用のブロックも作成しましょう。本番と異なるのは試行数だけなので，別々のリピート数で生成した`trial_types_practice`と`trial_types_main`を作成して，それぞれを`timeline_variables`に指定した変数を作成すれば大丈夫です。色々を省略しますが，以下のように書けます。

```javascript
var trial_types = [
  { letter: 'L', pos: 'left', cond: 'cong' },
  { letter: 'R', pos: 'left', cond: 'incong' },
  { letter: 'L', pos: 'right', cond: 'incong' },
  { letter: 'R', pos: 'right', cond: 'cong' },
];

var trial = {
  type: 'html-keyboard-response',
  // 省略
};

// 練習と本番で別々のtimeline_variablesを作成する
var trial_types_practice = jsPsych.randomization.repeat(trial_types, 2);
var trial_types_main = jsPsych.randomization.repeat(trial_types, 10);

// それぞれをtimeline_variablesに指定したブロックを作成する
var simon_practice = {
  timeline_variables: trial_types_practice,
  timeline: [trial],
};
var simon_main = {
  timeline_variables: trial_types_main,
  timeline: [trial],
};

jsPsych.init({
  timeline: [simon_practice, simon_main],
  // データの保存とか
});
```

基本的にこれで OK です。分析の際は，`internal_node_id`の列を参照すれば本番の試行だけを取り出すことが可能です。もし，より明示的にどの部分が課題の練習/本番なのかをデータに残しておきたいのであれば，少し冗長にはなりますが，以下のように書けばいいでしょう。

<details><summary>コード例</summary><div>

```javascript
// 練習と本番で別々のtimeline_variablesを作成する
var trial_types_practice = [
  { letter: 'L', pos: 'left', cond: 'cong', phase: 'practice' },
  { letter: 'R', pos: 'left', cond: 'incong', phase: 'practice' },
  { letter: 'L', pos: 'right', cond: 'incong', phase: 'practice' },
  { letter: 'R', pos: 'right', cond: 'cong', phase: 'practice' },
];

var trial_types_main = [
  { letter: 'L', pos: 'left', cond: 'cong', phase: 'main' },
  { letter: 'R', pos: 'left', cond: 'incong', phase: 'main' },
  { letter: 'L', pos: 'right', cond: 'incong', phase: 'main' },
  { letter: 'R', pos: 'right', cond: 'cong', phase: 'main' },
];

var trial = {
  type: 'html-keyboard-response',
  // 省略
  data: {
    letter: jsPsych.timelineVariable('letter'),
    pos: jsPsych.timelineVariable('pos'),
    cond: jsPsych.timelineVariable('cond'),
    phase: jsPsych.timelineVariable('phase'), // phase情報を追加する
  },
  // 省略
};

trial_types_practice = jsPsych.randomization.repeat(trial_types_practice, 2);
trial_types_main = jsPsych.randomization.repeat(trial_types_main, 10);

// それぞれをtimeline_variablesに指定したブロックを作成する
var simon_practice = {
  timeline_variables: trial_types_practice,
  timeline: [trial],
};
var simon_main = {
  timeline_variables: trial_types_main,
  timeline: [trial],
};

jsPsych.init({
  timeline: [simon_practice, simon_main],
  // データの保存とか
});
```

</div></details>

# サイモン課題の完成

教示・練習もできたので，あとは注視点と合わせてこれまでのコードに付け足せば，サイモン課題は完成です。

<details><summary>コード例</summary><div>

```html:simon.html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <script src="../jspsych.js"></script>
  <script src="../plugins/jspsych-html-keyboard-response.js"></script>
  <script src="../plugins/jspsych-survey-text.js"></script>
  <script src="../plugins/jspsych-survey-multi-choice.js"></script>
  <link rel="stylesheet" href="../css/jspsych.css"></link>
  <style>
    .text_left {
      position: absolute;
      left: 40%;
      top: 50%;
      transform: translateY(-50%) translateX(-50%);
      -webkit-transform: translateY(-50%) translateX(-50%);
    }
    .text_right {
      position: absolute;
      right: 40%;
      top: 50%;
      transform: translateY(-50%) translateX(50%);
      -webkit-transform: translateY(-50%) translateX(50%);
    }
  </style>
</head>
<body></body>
<script>

  // 参加者情報の取得
  var par_info = {};

  var par_id = {
    type: 'survey-text',
    questions: [
      {prompt: '参加者IDを入力してください', columns: 10, required: true, name: 'participantID'},
    ],
    button_label: '次へ',
    on_finish: function(data) {
      par_info.id = JSON.parse(data.responses).participantID // 一時保存
    }
  };

  var age = {
    type: 'survey-text',
    questions: [
      {prompt: '年齢を入力してください', columns: 3, required: true, name: 'age'},
    ],
    button_label: '次へ',
    on_finish: function(data) {
      par_info.age = JSON.parse(data.responses).age // 一時保存
    }
  };

  var gender = {
    type: 'survey-multi-choice',
    questions: [
      {prompt: "性別を選択してください", options: ['男性', '女性'], required: true, horizontal: true, name: 'gender'},
    ],
    button_label: '次へ',
    on_finish: function(data) {
      par_info.gender = JSON.parse(data.responses).gender // 一時保存
    }
  };


  // 教示
  var instruction = {
    type: 'html-keyboard-response',
    stimulus: 'これからキー押し課題を行ってもらいます。<br>' +
              '「L」「R」がランダムに表示されるので，<br><br>' +
              '「L」が表示されたら左手人差し指で f キー<br>'+
              '「R」が表示されたら右手人差し指で j キー<br><br>' +
              'を押してください。<br>' +
              '準備ができたらいずれかのキーを押して課題を始めてください。',
  }

  var go_practice = {
    type: 'html-keyboard-response',
    stimulus: 'まず課題の練習を行います。<br>' +
              '左手と右手の人差指を それぞれ f, j キーの上においてください。<br><br>' +
              '準備ができたらいずれかのキーを押して課題を始めてください。',
    post_trial_gap: 1000
  }

  var go_main = {
    type: 'html-keyboard-response',
    stimulus: 'まず本番を行います。<br>' +
              '左手と右手の人差指を それぞれ f, j キーの上においてください。<br><br>' +
              '準備ができたらいずれかのキーを押して課題を始めてください。',
    post_trial_gap: 1000
  }


  // 注視点
  var fixation = {
    type: 'html-keyboard-response',
    stimulus: "+",
    choices: jsPsych.NO_KEYS,
    trial_duration: 1000,
    post_trial_gap: 1000,
  }


  // サイモン課題
  var trial_types = [
    {letter: 'L', pos: 'left', cond: 'cong'},
    {letter: 'R', pos: 'left', cond: 'incong'},
    {letter: 'L', pos: 'right', cond: 'incong'},
    {letter: 'R', pos: 'right', cond: 'cong'},
  ];

  var trial = {
    type: 'html-keyboard-response',
    stimulus: function() {
      return '<div class="text_' + jsPsych.timelineVariable('pos', true) + '">' + jsPsych.timelineVariable('letter', true) + '</div>'
    },
    choices: ['f', 'j'],
    trial_duration: 1000,
    post_trial_gap: 1000, // ITI
    data: {
      letter: jsPsych.timelineVariable('letter'),
      pos: jsPsych.timelineVariable('pos'),
      cond: jsPsych.timelineVariable('cond')
    },
    on_finish: function(data) {
      data.key = jsPsych.pluginAPI.convertKeyCodeToKeyCharacter(data.key_press)
      if (data.letter == 'L') {
        data.correct = Number(data.key == 'f')
      } else {
        data.correct = Number(data.key == 'j')
      }
    }
  }

  trial_types_practice = jsPsych.randomization.repeat(trial_types, 2);
  trial_types_main = jsPsych.randomization.repeat(trial_types, 10);

  // それぞれをtimeline_variablesに指定したブロックを作成する
  var simon_practice = {
    timeline_variables: trial_types_practice,
    timeline:[trial]
  }
  var simon_main = {
    timeline_variables: trial_types_main,
    timeline: [trial],
  }

  jsPsych.init({
    timeline: [par_id, age, gender, instruction, go_practice, fixation, simon_practice, go_main, fixation, simon_main],
    on_finish: function() {
      jsPsych.data.addProperties(par_info);
      jsPsych.data.get().localSave('csv', 'data.csv');
    },
  });
</script>

</html>
```

</div></details>

試行間隔の時間（ITI）を設けるために，これまでずっと`jsPsych.init({})`に`default_iti`という項目を設けていましたが，これを削除して，適宜必要な部分に`post_trial_gap`という項目を設定しています（具体的には，`go_practice`，`go_main`，`fixation`, `trial`）。`default_iti`だと画面が遷移するたびに指定した時間だけ空白の画面が挿入されます。しかし，参加者情報の入力などでは必要ないので，必要な部分だけに ITI を設けるようにしました。

# おわりに

これでサイモン課題実験が完成しました。お疲れ様です。ぜひ，身近な人に実施してみてください。

もちろん，チュートリアルということで，jsPsych の機能のほんの一部しか紹介していません。ウィンドウの全画面表示の方法すら紹介していません。しかし，ここまでやり遂げたみなさんなら，ダウンロードした jsPsych に同梱されている example コードや，[jsPsych の公式リファレンス](https://www.jspsych.org/)を見ながら，ご自身の研究課題に合わせてコードを書くことができるはずです。

これからもがんばってください。

# 2021/03/15 更新

このチュートリアルのつぎのステップとして読むといいかもしれない記事を zenn というサイトで公開しています（[zenn の マイページ](https://zenn.dev/snishiyama)）。今後は zenn で記事を公開していくつもりなので，そちらもたまに見に来ていただけると幸いです。
