# transcript-to-clip

YouTubeアーカイブの文字起こしをもとに、LLMで切り抜き候補を出し、PowerShell + FFmpegで動画を切り出すための小さな補助ツールです。

LLMに「どこを切り抜くか」を考えてもらい、該当箇所を大まかに切り出すためのPowerShellコマンドを生成させます。

その後、Premiere Proなどの編集アプリで尺の調整・テロップ作成を行い、切り抜き動画として仕上げることを前提にしています。

## コンセプト

このリポジトリの成果物は、以下の2つに分かれています。

```text
LLM / プロンプト
  - YouTubeアーカイブの文字起こしを読む
  - 切り抜き候補を提案する
  - 開始時刻と終了時刻からDurationを計算する
  - PowerShell用のNew-Clipコマンドを生成する
  - プロンプトの本文はChatGPT Atlasに対して投げる前提のつくり

PowerShell / New-Clip
  - 出力先フォルダを作成する
  - FFmpegを呼び出す
  - 指定範囲を切り出してエンコードする
  - クリップファイルを書き出す
```

## 必要なもの

* NVIDIA GPU搭載のWindows PC
* PowerShell 7
* FFmpeg
* ChatGPT Atlas

FFmpegは別途インストールし、`ffmpeg` コマンドをPowerShellから実行できる状態にしてください。

このリポジトリにはFFmpeg本体は含めていません。

## 使い方

`New-Clip` 関数をPowerShellプロファイル、または現在のPowerShellセッションに読み込みます。

PowerShellで以下を実行すると、プロファイルのパスを確認できます。

```powershell
$profile
```

表示されたファイルに `New-Clip.ps1` の中身をコピペしてください。
ファイルが存在しない場合は、新しく作成してください。

その後、LLMが生成した `New-Clip` コマンドをPowerShellに貼り付けて実行します。

以下は生成されたコマンドの例です。

```powershell
$InputFile = "$env:USERPROFILE\Videos\4K Video Downloader+\sample-archive.mp4"
$OutDir = "C:\shared\generated-clips\sample-archive"

New-Clip $InputFile $OutDir "00:03:50" "00:01:05" "clip_01_sample.mp4"
New-Clip $InputFile $OutDir "00:04:52" "00:02:16" "clip_02_sample.mp4"

Write-Host "clip generation completed."
```

## プロンプト側の役割

LLMには、YouTubeアーカイブの文字起こしを渡したうえで、以下のような出力を依頼します。

* 切り抜き候補を5〜10件出す
* 各候補に開始時刻・終了時刻を付ける
* 映像編集アプリで最終調整しやすいように前後に余白を入れる
* 候補ごとに理由とタイトル案を出す
* 開始時刻〜終了時刻からDurationを計算する
* `New-Clip` を使ったPowerShellコマンドを生成する
* 出力ファイル名は英数字・ハイフン・アンダースコアのみを使う

LLMが切り抜き候補の選定とコマンド生成を担当し、実際の動画切り出しは `New-Clip` が行います。

## パスとプライバシー

サンプルでは、入力ファイルに `$env:USERPROFILE` を使用しています。

```powershell
$InputFile = "$env:USERPROFILE\Videos\4K Video Downloader+\sample-archive.mp4"
```

これにより、READMEや記事などでサンプルコードを公開しても、Windowsのユーザー名を含む絶対パスを出さずに済みます。

出力先の例には、`C:\shared\generated-clips` を使用しています。

```powershell
$OutDir = "C:\shared\generated-clips\sample-archive"
```

このパスはローカル作業用の例です。
自分の環境に合わせて変更してください。

## 注意事項

* 入力動画ファイルは、あらかじめローカルに保存されている必要があります。
* 切り抜き候補の選定は、YouTubeアーカイブの文字起こしをもとにLLMが行います。
* 文字起こしの精度や内容によって、候補の品質は変わります。
* 書き出したクリップは、必要に応じてPremiere Proなどで最終トリムしてください。
* 現在のFFmpeg設定は、NVIDIA NVENC環境を前提にしています。

## ライセンス

このリポジトリは MIT License で公開しています。

FFmpegは別ライセンスのソフトウェアです。
このリポジトリではFFmpeg本体を配布せず、外部コマンドとして呼び出すだけにしています。
