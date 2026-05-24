# 毎朝7時にDiscordへ「今日のアナログホラー案」を投稿するシステム

このリポジトリは、毎日朝7時（日本時間）にDiscordの指定チャンネルへ、YouTube用のアナログホラー企画案を1件投稿するためのPythonスクリプトです。

OpenAI APIキーがある場合は、毎日新しい企画案をAIで生成します。APIキーがない場合でも、動作確認用の固定サンプル案を投稿できます。

## 1. このシステムでできること

- Discord Webhookを使って、指定チャンネルへ自動投稿できます。
- GitHub Actionsで毎朝7時（日本時間）に自動実行できます。
- ローカルPCから手動でテスト投稿できます。
- 投稿内容は、5〜10分のYouTube動画にしやすい具体的なアナログホラー案です。
- `OPENAI_API_KEY` が未設定でも、フォールバック案で最低限の動作確認ができます。

投稿される内容は、次の形式です。

```text
【今日のアナログホラー案】作品タイトル

1. 一言コンセプト
2. どんな動画か
3. 視聴者が最初に感じる違和感
4. 中盤で明らかになる異常
5. 最後のオチ
6. 使える演出
7. サムネ・タイトル案
8. 制作難易度
9. YouTubeで伸びそうな理由
```

## 2. Discord Webhookの作り方

1. Discordを開き、投稿したいサーバーに入ります。
2. 投稿先にしたいチャンネルの歯車アイコンを押します。
3. 左メニューから「連携サービス」を開きます。
4. 「ウェブフック」を選びます。
5. 「新しいウェブフック」を押します。
6. 名前をわかりやすくします。例: `Analog Horror Bot`
7. 投稿先チャンネルが正しいことを確認します。
8. 「ウェブフックURLをコピー」を押します。

コピーしたURLが `DISCORD_WEBHOOK_URL` です。外部に公開しないでください。

## 3. OpenAI APIキーの設定方法

1. OpenAI Platformにログインします。
2. APIキーのページで新しいAPIキーを作成します。
3. 作成したキーをコピーします。

このキーが `OPENAI_API_KEY` です。これも外部に公開しないでください。

APIキーがない状態でも、このシステムはフォールバック案を使って動作確認できます。ただし、毎日違う高品質な案を生成したい場合はAPIキーを設定してください。

## 4. GitHub Secretsへの登録方法

GitHub Actionsで安全に動かすため、Webhook URLやAPIキーはコードに直接書かず、GitHub Secretsに登録します。

1. GitHubでこのリポジトリを開きます。
2. 上部メニューの「Settings」を開きます。
3. 左メニューの「Secrets and variables」から「Actions」を開きます。
4. 「New repository secret」を押します。
5. 次の2つを登録します。

```text
Name: DISCORD_WEBHOOK_URL
Secret: DiscordでコピーしたWebhook URL
```

```text
Name: OPENAI_API_KEY
Secret: OpenAI Platformで作成したAPIキー
```

必要な場合だけ、使用モデルも変更できます。

```text
Name: OPENAI_MODEL
Secret: gpt-5-mini
```

通常は `OPENAI_MODEL` を登録しなくても大丈夫です。未設定なら `gpt-5-mini` が使われます。

## 5. GitHub Actionsで毎朝7時に動かす方法

自動実行の設定は `.github/workflows/daily.yml` に入っています。

```yaml
- cron: "0 22 * * *"
```

GitHub ActionsのcronはUTC基準です。日本時間はUTCより9時間進んでいるため、日本時間の朝7時はUTCの前日22時です。

つまり、`0 22 * * *` と書くことで、毎日日本時間の朝7時に実行されます。

GitHub Actionsを有効化する手順:

1. GitHubでこのリポジトリを開きます。
2. 上部メニューの「Actions」を開きます。
3. 初回だけ確認ボタンが出る場合があります。その場合は有効化してください。
4. 左側に `Daily Analog Horror Idea` というワークフローが表示されれば準備完了です。

手動で試す場合:

1. GitHubの「Actions」を開きます。
2. `Daily Analog Horror Idea` を選びます。
3. 「Run workflow」を押します。
4. 成功するとDiscordに投稿されます。

## 6. ローカルでテスト投稿する方法

まずPythonをインストールしておいてください。Python 3.11以上を推奨します。

次に、このフォルダで依存ライブラリを入れます。

```bash
pip install -r requirements.txt
```

`.env.example` をコピーして `.env` という名前のファイルを作ります。

```bash
cp .env.example .env
```

WindowsのPowerShellなら、次でも大丈夫です。

```powershell
Copy-Item .env.example .env
```

作成した `.env` を開き、次のように自分の値を入れます。

```env
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/...
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-5-mini
```

投稿せずに内容だけ確認する場合:

```bash
python main.py --preview
```

Discordへ実際に投稿する場合:

```bash
python main.py
```

`OPENAI_API_KEY` が空の場合は、固定サンプル案が投稿されます。Discord投稿の確認だけしたいときに便利です。

## 7. よくあるエラーと対処法

### `DISCORD_WEBHOOK_URL が未設定です`

Discord Webhook URLが設定されていません。

対処法:

- ローカル実行なら `.env` に `DISCORD_WEBHOOK_URL` を書いてください。
- GitHub Actionsなら GitHub Secrets に `DISCORD_WEBHOOK_URL` を登録してください。

### Discordへの投稿に失敗しました

Webhook URLが間違っている、削除されている、またはDiscord側で拒否されています。

対処法:

- Discordのチャンネル設定からWebhook URLを作り直してください。
- GitHub Secretsや `.env` に余計な空白が入っていないか確認してください。

### `OPENAI_API_KEY が未設定です`

OpenAI APIキーが設定されていません。

この場合はエラーで止まらず、フォールバック案を投稿します。毎日違う案をAI生成したい場合は `OPENAI_API_KEY` を設定してください。

### OpenAI APIでの生成に失敗しました

APIキーが間違っている、利用上限に達している、ネットワークエラーが起きている可能性があります。

この場合もフォールバック案を使って投稿します。GitHub Actionsのログに失敗理由が出ます。

### GitHub Actionsが指定時刻に動かない

GitHub Actionsのスケジュール実行は、混雑状況によって数分遅れることがあります。

また、長期間更新のないリポジトリではスケジュール実行が止まる場合があります。その場合はActions画面から手動実行して確認してください。

## 8. アイデアの出力形式を変えたい場合の編集場所

主に編集する場所は `prompts.py` です。

- `CATEGORIES`: アイデアのカテゴリ一覧です。
- `SYSTEM_PROMPT`: AIに守らせたい全体方針です。
- `build_user_prompt`: Discordに投稿する形式や項目を指定しています。
- `fallback_idea`: APIキーがないときに使う固定サンプル案です。

たとえば投稿項目を増やしたい場合は、`build_user_prompt` の番号リストを編集してください。

## 最終的にあなたがやること

1. Discord Webhook URLを取得する。
2. OpenAI APIキーを用意する。
3. GitHub Secretsに `DISCORD_WEBHOOK_URL` と `OPENAI_API_KEY` を登録する。
4. GitHub Actionsを有効化する。

これで、毎朝7時にDiscordへ「今日のアナログホラー案」が自動投稿されます。
