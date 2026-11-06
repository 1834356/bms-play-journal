# BMS Score Viewer（LR2 / BMZ）

LR2と[BMZ-player](https://github.com/hyrorre/bmz-player)のスコアDBから、難易度表のランプ・最小BP・スコアレートとプレイ記録を表示するWebページです。片方だけでも使えます。GitHub Pagesで公開すれば、自宅のPCが停止していてもスマートフォンから最後に更新した記録を閲覧できます。

このリポジトリは配布用です。実際のスコア・プレイ履歴・ローカル設定は含みません。[公開ページ](https://1834356.github.io/bms-score-viewer/)では設定案内を表示し、「デモ」を押すと架空のデータを試せます。

## 利用上の注意

このツールはLR2・BMZ-playerの非公式ビューアです。動作や集計結果の正確性は保証しません。利用前にスコアDBのバックアップを取ってください。

公開リポジトリに同期すると、閲覧用JSONに含まれるスコアやプレイ履歴も公開されます。詳しくは[データの公開範囲](#データの公開範囲と計算方法)を確認してください。

## できること

- 複数の難易度表を切り替えて、レベル別のランプを集計。グラフを押すとランプの内訳を表示し、内訳のランプを押すと該当する譜面一覧へ移動します。
- 譜面一覧にランプ・最小BP・スコアレートを表示。曲名検索、レベル・ランプ絞り込み、並べ替えに対応します。
- ページ上部の「LR2 / BMZ」で記録を切り替えます。難易度表・検索文字・絞り込み・並び順はプレイヤーごとに記憶し、デモの設定は別に保存します。
- 長い曲名は省略し、押すとBMS-IRの譜面ページを別タブで開きます。
- 日別に総ノーツ数・プレイ数と、ランプ・BP・スコアの更新を表示します。日別の更新は1枚のPNGに保存できます。
- 月別カレンダーに日ごとの総ノーツ数を表示し、日付を押すと日別記録へ移動します。演奏時間はLR2のみ表示します。
- 難易度表外の譜面名もローカルの曲情報DBから取得します。LR2のスコアDBは日時付きでバックアップできます。

## 1. 必要なもの

Windows、GitHubアカウント、Python 3.10以上、Gitと、使うプレイヤーのDBが必要です。PowerShellで以下が実行できることを確認してください。

```powershell
python --version
git --version
```

Pythonは[公式サイト](https://www.python.org/downloads/windows/)、Gitは[公式サイト](https://git-scm.com/downloads/win)から導入できます。Pythonのインストール時にはPATHに追加してください。Pythonの追加ライブラリは不要です。

- **LR2:** スコアDBを使います。日別・月別記録には[BeMusicSeekerのLR2プレイログ機能](https://neeted.github.io/bemusicseeker-unofficial-fork/manual.ja.html#lr2プレイログ)で作られる `bms_lr2_play_history` が必要です。ログがなくてもベストスコアは表示できます。
- **BMZ:** 使用中のプロファイルの `score.db` と、同じBMZ環境の `library.db` を使います。日別・月別記録はBMZの保存済み履歴を使うので、BeMusicSeekerは不要です。

## 2. 自分のリポジトリを作る

[このリポジトリ](https://github.com/1834356/bms-score-viewer)の「Fork」から自分のアカウントへフォークします。「Use this template」から作成することもできます。以下ではリポジトリ名を `bms-score-viewer` とします。無料のGitHub Pagesを使用する場合は公開リポジトリで作成してください。

PowerShellで、`YOUR_NAME` を自分のGitHubユーザー名に置き換えて実行します。

```powershell
git clone https://github.com/YOUR_NAME/bms-score-viewer.git
cd bms-score-viewer
Copy-Item config.example.json config.local.json
```

Gitのコミット用の名前・メールが未設定の場合は、このフォルダ内で設定します。メールにはGitHubの設定画面にある非公開用のnoreplyアドレスも使えます。

```powershell
git config user.name "自分の名前"
git config user.email "自分のメールアドレス"
```

## 3. 使うプレイヤーを設定する

`config.local.json` をメモ帳などで開き、自分の環境に合わせて変更します。

### LR2だけ使う場合

```json
{
  "database": "D:/LR2/LR2files/Database/Score/your-player.db",
  "backupDirectory": "C:/BMSbackup/scoredb",
  "tableUrls": [
    "https://stellabms.xyz/sl/table.html",
    "https://stellabms.xyz/st/table.html",
    "https://miraiscarlet.github.io/bms/table/genocide_insane/insane_bms.html"
  ]
}
```

- `database`: 使用しているLR2プレイヤーのスコアDB。通常は `LR2files/Database/Score/` 内の `.db` ファイルです。
- `backupDirectory`: バックアップ先のフォルダ。存在しなければ自動作成します。OneDrive内のフォルダも指定できます。バックアップが不要なら、この項目を削除できます。
- `tableUrls`: 表示したい難易度表のURL。初期設定はSatellite、Stella、発狂BMS難易度表です。

JSONのパスは例のように `/` で区切ると、そのまま記入できます。`\` を使う場合は `\\` と記入してください。最後の項目にはカンマを付けません。

譜面タイトル用の `song.db` は、スコアDBの `Score` フォルダの1つ上から自動検出します。別の場所の場合は `"songDatabase": "D:/LR2/LR2files/Database/song.db"` を設定に追加してください。初期設定の `"bmz": null` はそのままで構いません。

### BMZだけ使う場合

`database` と `backupDirectory` を削除し、`bmz` と `tableUrls` を設定します。初期設定を以下の例で置き換えて、パスを自分の環境に合わせる方法でも構いません。

```json
{
  "bmz": {
    "scoreDatabase": "D:/your-bmz/data/profiles/default/score.db",
    "libraryDatabase": "D:/your-bmz/data/library.db",
    "ruleMode": "Beatoraja",
    "lnSetting": "AutoLn"
  },
  "tableUrls": [
    "https://stellabms.xyz/sl/table.html",
    "https://stellabms.xyz/st/table.html",
    "https://miraiscarlet.github.io/bms/table/genocide_insane/insane_bms.html"
  ]
}
```

- `scoreDatabase`: 使用中のBMZプロファイルの `score.db`。プロファイルを変えている場合は `default` をそのフォルダ名に置き換えます。
- `libraryDatabase`: BMZの曲情報DB。難易度表のMD5と、BMZがスコアの識別に使うSHA-256を照合します。
- `ruleMode`: `Beatoraja`、`Lr2Oraja`、`Dx`。BMZで普段使う判定ルールに合わせてください。省略時は `Beatoraja` です。
- `lnSetting`: `AutoLn`、`AutoCn`、`AutoHcn`、`ForceLn`、`ForceCn`、`ForceHcn`。省略時は `AutoLn` です。

### LR2とBMZを併用する場合

LR2の `database` と必要に応じて `backupDirectory` を残し、BMZの例にある `bmz` を同じJSONの最上位に追加します。`tableUrls` は1つだけ設定し、両方で共用します。

`config.local.json` と元DBはGitの対象外です。自分の設定を `config.example.json` に書き込まないでください。

すでにLR2版を使っている場合は、フォークにこのリポジトリの更新を取り込み、PC側も更新します。既存の `config.local.json` はそのまま使えます。BMZも使う場合は `bmz` の項目を追加してください。

## 4. 最初の更新を実行する

このフォルダ内の `update-and-backup.cmd` をダブルクリックします。設定したプレイヤーの閲覧用JSONを作成し、自分のリポジトリに同期します。LR2のバックアップ先を設定している場合は、先にLR2のスコアDBをバックアップします。初回は自分のGitHubアカウントで認証してください。

PowerShellから実行する場合は次のコマンドです。

```powershell
.\update.ps1 -Push
```

スクリプトはこのフォルダのGitリモート `origin` の `main` に同期します。フォークしたリポジトリをクローンした場合は、自分のリポジトリが更新先になります。

バックアップ・難易度表の取得・データ出力のいずれかに失敗すると、GitHubへの同期を中止します。

## 5. GitHub Pagesで公開する

自分のリポジトリの **Settings → Pages** を開き、以下を設定して保存します。

- Source: **Deploy from a branch**
- Branch: **main**
- Folder: **/(root)**

公開処理が完了すると、Pagesの設定画面に閲覧URLが表示されます。リポジトリ名が `bms-score-viewer` なら、通常は `https://YOUR_NAME.github.io/bms-score-viewer/` です。スマートフォンからこのURLを開いてください。上部の「LR2 / BMZ」で切り替えられます。`?player=bmz` を付けるとBMZを直接開けます。BMZだけ設定した場合は、通常のURLでもBMZが表示されます。

参考: [GitHub Pagesの公開元を設定する](https://docs.github.com/ja/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)

## 6. プレイ後に更新する

プレイした後、`update-and-backup.cmd` を実行します。公開処理の完了後、閲覧ページを再読み込みすると新しい記録を確認できます。併用時にBMZだけを更新する場合は、`.\update.ps1 -BmzOnly -Push` を実行します。

難易度表は毎回取得します。閲覧用JSONは出力日時を除いた内容を比較し、同じならファイルを書き換えません。未送信コミットがない場合はGitHubへの送信も省略します。LR2のバックアップは、最新のコピーと同じ内容なら新しいファイルを残しません。

GitHubに同期せず、バックアップとJSON出力だけ実行する場合:

```powershell
.\update.ps1
```

LR2のバックアップだけ実行する場合:

```powershell
.\update.ps1 -BackupOnly
```

バックアップは `<DB名>_YYYYMMDD_HHMMSS_ffffff.db`（日本時間）で保存し、過去のファイルは上書き・削除しません。SQLiteのバックアップ機能で整合性を保ってコピーし、確認を通過したものだけを保存します。

同じ内容でも新しく保存したい場合は `python -X utf8 backup_scores.py --config config.local.json --force` を実行できます。

## 難易度表を追加・削除する

`config.local.json` の `tableUrls` を編集して保存し、`update-and-backup.cmd` を実行します。

**追加:** 配列の中に難易度表のURLを追加します。各URLを `"` で囲み、項目間をカンマで区切ってください。

**削除:** 不要なURLの行を削除します。例えばSatelliteとStellaだけにする場合は以下です。

```json
"tableUrls": [
  "https://stellabms.xyz/sl/table.html",
  "https://stellabms.xyz/st/table.html"
]
```

上の断片は設定ファイル内の `tableUrls` 部分を置き換える例です。他の設定はそのまま残します。併用時に両方の表を更新するには `update-and-backup.cmd` を実行してください。

難易度表は1つ以上必要です。URLは重複させず、MD5を含む一般的な難易度表形式のページ、またはヘッダJSONを指定してください。MD5がない譜面はスコアと照合できないため除外します。対応していない形式や取得できないURLがある場合は、更新時にエラーを表示します。

表を削除しても元のスコアDBや譜面ファイルには影響しません。日別記録は、対象のプレイ履歴にあれば難易度表外の譜面も集計します。

## データの公開範囲と計算方法

公開ページと閲覧用JSON（LR2は `data/viewer.json`、BMZは `data/bmz.json`）には、URLを知っている人がアクセスできます。JSONには曲名・ランプ・BP・EXスコア・総ノーツ数・日別記録などを含みます。更新スクリプトがコミットするのはこのJSONだけです。元DB、設定、認証情報、リプレイ、ゴースト、ローカルパスは含めません。

譜面一覧・日別更新一覧・全プレイ履歴・1画面表示の曲名は、MD5で対応するBMS-IRの譜面ページへのリンクです。長い曲名は省略表示を維持し、全文の展開はしません。デモや譜面のMD5がない記録ではリンクを表示しません。

スコアレートは `EXスコア ÷ (総ノーツ数 × 2) × 100`、EXスコアは `perfect × 2 + great` で計算します。未プレイや総ノーツ数未取得の場合は「—」を表示します。

日別記録の「1画面表示（スクリーンショット用）」を押すと、選択した日の集計と全更新を複数列に並べ、画面内に収めます。日付は表示中にも変更できます。日別記録の「ランプ更新のみを1画面表示」から、ランプが上がった譜面だけを直接表示できます。1画面表示の「表示」でも「すべての更新」と「ランプ更新のみ」を切り替えられます。この表示では集計欄もランプ更新譜面数に切り替わります。1画面表示の「画像を保存」で、日付・集計・表示中の全更新をPNGに保存できます。操作ボタンは画像に含めません。iPhoneなどファイル共有に対応したタッチ端末では共有メニューを開き、PCなどではPNGをダウンロードします。画像はブラウザ内で作成します。「通常表示へ戻る」またはEscで閉じます。 スマートフォンでは表示ボタンを日付の下に縦並びで配置します。 ホーム画面に追加したWebアプリなどで以前の画面が残る場合は「最新版を読み込む」を押してください。画面上部と日別記録に配置し、現在のURLのクエリを更新してページを読み直します。ボタン自体がない旧版では、Safariなどのブラウザから最新版URLを開き直してください。公開URLに `?dailyView=lamps#daily` を付けるとランプ更新のみの1画面表示を直接開けます（既にクエリがある場合は `&dailyView=lamps` を追加）。件数が多いほど文字が小さくなるため、横向きや大きな画面での撮影がおすすめです。全プレイ履歴と説明は通常表示で確認できます。

日別の打鍵数はプレイした譜面の総ノーツ数の合計です。LR2は `new_totalnotes × player_playcount_delta`、BMZは保存された各プレイの `total_notes` を使います。途中終了・FAILEDでもログに総ノーツ数があれば加算し、値がないプレイはノーツ合計から除外します。日本時間で集計し、同じ譜面が複数の難易度表に載っていてもプレイを二重に数えません。

BMZのベストには取り込んだスコアも反映されます。日別・月別は設定した判定ルールとLN設定の単曲プレイを集計し、他のプレイヤーから取り込んだ履歴、IR取り込み元と紐づく履歴、コース、オートプレイ、BATTLEを除外します。日ごとの演奏時間は取得できないため「—」と表示します。BMZが履歴を残さないプレイは集計されません。

ランプ更新は通常ランプの上昇、BP更新は以前の最小BPの減少、スコア更新は以前のEXスコアの上昇です。初回BP・スコア記録は改善に含めません。新規AAAは最大スコアの8/9以上になった譜面です。

## よくある確認事項

- 「未設定」と表示される: 自分のDBを設定し、更新を正常に完了させてください。初期JSONは空です。
- LR2の日別記録が出ない: BeMusicSeekerのログ設定と、確定済みログがDBにあるかを確認してください。
- BMZの記録が出ない: `scoreDatabase` が使用中のプロファイルを指していることと、`ruleMode`・`lnSetting` がBMZでのプレイ時の設定と合っていることを確認してください。
- 曲名が取得できない: LR2では `song.db`（別の場所なら `songDatabase`）、BMZでは `libraryDatabase` のパスを確認してください。
- GitHubの同期に失敗する: `git remote -v` で自分のリポジトリが `origin` になっているかと、認証・コミット用の名前とメールを確認してください。
- PowerShellがスクリプト実行を禁止する: Windowsの実行ポリシーを確認してください。自身のPCで許可する場合は `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` を使用できます。組織管理のPCでは管理者の設定に従ってください。

## 技術構成と確認

フロントエンドはHTML/CSS/JavaScript、エクスポータとバックアップはPython標準ライブラリだけを使用します。DBは読み取り専用で開きます。ページは同じ公開先から相対パスでJSONを読み込むため、フォーク先のデータを表示します。

```powershell
python -X utf8 -m unittest discover -s tests -p "test_*.py"
node tests/test_daily_groups.js
node tests/test_png_save.js
```
