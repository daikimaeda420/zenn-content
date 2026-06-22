---
title: "CodexでYouTube動画編集の半自動化GUIを作った記録"
emoji: "🎬"
type: "tech"
topics: ["codex", "youtube", "video", "automation", "remotion"]
published: true
---

![CodexでYouTube動画編集の半自動化GUIを作った記録](/images/codex-articles/youtube-auto-edit-thumbnail.png)

# はじめに

ゲーム制作を進める中で、YouTube向けの制作ログ動画も継続的に出したくなりました。

ただ、毎回やる作業はかなり似ています。

- 台本を書く
- 音声を作る
- 画面録画を配置する
- 字幕を合わせる
- 立ち絵やロゴを載せる
- 本編とショートを書き出す
- 投稿タイトル、説明文、タグを作る

これを毎回手作業でやると、ゲーム本体を作る時間が削られます。

そこで、Codexと相談しながら、ローカルで動くYouTube動画編集の半自動化パイプラインを作りました。この記事は、その制作記録です。

# 作ったもの

作ったのは、Markdown台本、音声、画面録画、キャラ素材、BGMをまとめて扱い、Remotionで動画を書き出す仕組みです。

主な構成は次の通りです。

```text
video-input/
  youtube-script-voicevox.md
  publish.json
  recordings/
    prompt.mov
    play.mov
  characters/
    hamukin-talk.png
    hamukin-idle.png
    mae-talk.png
    mae-idle.png
  bgm/
    bgm.mp3

src/video/
  video.tsx
  shorts.tsx
  generated/
    timeline.json
    project.json

scripts/video/
  prepare.mjs
  generate-voices.mjs
  gui-server.mjs
  gui.html
  timeline-utils.mjs

scripts/publish/
  metadata.mjs
  youtube-auth.mjs
  youtube.mjs
  social-package.mjs
```

動画の仕様は `VIDEO_PIPELINE.md` にまとめ、実装は `scripts/video` と `src/video` に分けました。

# なぜGUIを作ったか

最初は、台本からタイムラインを生成して、そのままRemotionで書き出せばよいと考えていました。

ただ、実際に動画を作ると、次のような調整が必ず発生します。

- 字幕の位置を少し下げたい
- 立ち絵を少し大きくしたい
- セクションごとに使う録画を変えたい
- 冒頭だけ再生速度を変えたい
- キャラを左右反転したい
- 音声と字幕が合っているか確認したい

これをJSONだけで編集するのはつらいです。

そこで、`npm run video:gui` でブラウザGUIを起動し、素材確認、タイムライン調整、音声生成、プレビュー、完成版書き出しまで操作できるようにしました。

```bash
npm run video:gui
```

通常は次のURLで開きます。

```text
http://127.0.0.1:5174/
```

GUI側では、録画、キャラ、BGM、台本、投稿メタ情報の不足を見られるようにしています。素材が足りない場合も、仮素材やテンプレートで進められるようにしました。

# タイムラインを構造化した

重要だったのは、動画を「一本の長い手作業データ」として扱わないことでした。

台本をもとに、セクション、発話、話者、録画素材、字幕、音声ファイルを構造化し、`timeline.json` に落とします。

```text
Markdown台本
  ↓
タイムライン生成
  ↓
VOICEVOX音声生成
  ↓
Remotionプレビュー
  ↓
本編MP4 / ショートMP4
```

構造化しておくと、あとからGUIで調整した内容も `project.json` として保存できます。

```text
src/video/generated/project.json
public/video/generated/project.json
```

この形にしたことで、Codexに対しても「動画編集を直して」ではなく、「セクションごとの録画割り当てを見直して」「ショート側の冒頭58秒だけ調整して」のように具体的に依頼しやすくなりました。

# 本編とショートを同じ素材から作る

YouTubeでは、本編だけでなくショートも同時に作りたい場面があります。

そこで、`AutoVideo` と `AutoShorts` を分け、同じ台本と素材から横長本編と縦長ショートを書き出せるようにしました。

```bash
npm run video:render
npm run video:shorts
```

出力は次のように分けています。

```text
video-output/final.mp4
video-output/shorts.mp4
```

ショートは完全に別動画として作り直すのではなく、既存の台本、音声、ゲームプレイ録画を再利用します。これにより、制作ログを一本作ったら、短い導線動画も同時に用意できます。

# 共通Loading演出を入れた

動画のブロックが切り替わるところには、共通のLoading演出を入れるようにしました。

```text
白背景
ロゴ
マエのGIF
Loading...
短い音声
```

これは、コード生成パート、ゲームプレイパート、締めのような区切りを見せるためです。

手作業で毎回挿入するのではなく、タイムライン上のブロック切り替わりに自動で差し込むようにしました。

# 投稿準備も自動化した

動画を書き出したあとも、投稿まわりの作業があります。

- タイトル
- 説明文
- タグ
- ショート説明文
- SNS向け投稿文

これらは `video-input/publish.json` をもとに生成します。

```bash
npm run publish:metadata
npm run publish:social-package
```

YouTubeへの投稿は、YouTube Studioをブラウザ操作で自動クリックするのではなく、YouTube Data APIを使う方針にしました。

```bash
npm run publish:youtube:auth
npm run publish:youtube
```

ローカルのOAuth情報や生成済み動画は公開リポジトリに入れない前提です。

# 音声はまだ改善余地がある

現時点ではVOICEVOXを使っています。

キャラクター音声としては便利ですが、自分の肉声に近い自然なナレーションにしたい場合は、将来的に別の音声合成サービスを検討する余地があります。

制作中には、OpenAIのCustom Voice系の選択肢やElevenLabs APIも候補として考えました。ただし、料金、商用利用、声の権利、YouTube収益化時の扱いは変わりやすいため、導入前に最新の公式条件を確認する必要があります。

現段階では、音声システムを差し替えられるようにしておくことを優先しています。

# Codexに任せてよかったところ

Codexが特に役に立ったのは、次のような部分です。

- 仕様書を先にMarkdownで整理する
- スクリプトの責務を分ける
- GUIとCLIを同じデータ構造につなぐ
- 生成物と入力素材の置き場所を分ける
- READMEや運用手順を残す
- 何度も使うコマンドを `package.json` に集約する

動画編集そのものを完全にAI任せにしたというより、「繰り返し作業を減らす制作環境」をCodexと一緒に作った感覚です。

# 反省点

最初から完成動画の見栄えを詰めようとすると、実装が重くなります。

先にやるべきだったのは、次の順番でした。

1. 台本からタイムラインを作る
2. 音声を生成する
3. プレビューを出す
4. GUIで調整する
5. 本編を書き出す
6. ショートを書き出す
7. 投稿メタ情報を作る

この順番にすると、一つずつ動作確認できます。

逆に、字幕デザイン、演出、音声品質、テンポ調整を最初から全部やろうとすると、どこが原因で破綻しているのか分かりにくくなります。

# まとめ

今回作ったのは、動画を自動で面白くしてくれる魔法のシステムではありません。

ただ、毎回同じ作業を繰り返さなくてよい状態にはかなり近づきました。

特に良かったのは、動画制作を「素材」「台本」「タイムライン」「出力」「投稿メタ」に分けたことです。ここを分けておくと、あとから音声合成だけ差し替える、ショートだけ改善する、GUIだけ改修する、といった変更がしやすくなります。

今後は、音声の自然さ、字幕デザイン、投稿後の分析まで含めて、制作ログ動画を継続しやすい形に育てていきたいです。
