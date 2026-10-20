# ha-rain-announce

雨が降り出したとき・やんだときに、Home Assistant からスマートスピーカー（Google Home など）に発話させる設定です。

![システム図](docs/system.png)

## 動き

| 判定 | 条件 | 雨フラグ |
|---|---|---|
| 降り出し | 降水強度が 0.5 mm/h を超えた、かつ雨フラグが OFF | ON にする |
| 止み | 0.1 mm/h 未満が 10 分続いた、かつ雨フラグが ON | OFF にする |

- 雨フラグは発話しないときも更新します。ずっと晴れている日に「やみました」と言わないためです
- 発話するのは、在宅・指定の時間帯（初期値 7〜22 時）・カレンダーに時間指定の予定が無い、をすべて満たすときだけです
- 読み上げ中だけ音量を上げ、数秒後に元の音量へ戻します
- 数値と文言は Blueprint の入力で変えられます

## 必要なもの

- Home Assistant（2024.10 以降）
- Yahoo! JAPAN の**アプリケーション ID**
- TTS エンジン（下の例は Google 翻訳 TTS）
- HA から再生できるスピーカー
- 在宅判定に使う `person`、予定を見る `calendar`（どちらも任意）

Yahoo! 気象情報 API は日本国内の座標だけに対応しています。

## 設定

### 1. アプリケーション ID を取得する

[Yahoo!デベロッパーネットワーク](https://developer.yahoo.co.jp/) でアプリケーションを登録し、Client ID（アプリケーション ID）を控えます。

`secrets.yaml` に追記します。

```yaml
yolp_appid: <取得したアプリケーション ID>
```

### 2. 降水強度センサーと雨フラグを追加する

[`configuration.yaml.example`](configuration.yaml.example) の中身を `configuration.yaml` に追記し、HA を再起動します。

- 座標は HA の「ホーム」ゾーンから読むので、書き換えは不要です
- 再起動後、`sensor.yolp_rain_intensity` に mm/h の値が入り、`input_boolean.was_raining` ができていることを確認します

この 2 つは Blueprint に含められないため、手で追記します。

### 3. Blueprint を取り込む

[![Blueprint をインポート](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Funicco%2Fha-rain-announce%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Frain_announce.yaml)

ボタンが使えない場合は、「設定 → オートメーションとシーン → ブループリント → ブループリントをインポート」に次の URL を貼ります。

```
https://github.com/unicco/ha-rain-announce/blob/main/blueprints/automation/rain_announce.yaml
```

### 4. オートメーションを作る

取り込んだ Blueprint から作成し、次を選びます。

| 入力 | 選ぶもの |
|---|---|
| 降水強度センサー | `sensor.yolp_rain_intensity` |
| 雨フラグ | `input_boolean.was_raining` |
| スピーカー | 読み上げさせる `media_player` |
| TTS エンジン | 例: `tts.google_translate_en_com`（言語は `ja` のままで日本語を読みます） |
| 在宅を判定する人・予定を見るカレンダー | 任意 |

## 注意

- **アプリケーション ID は人と共有しないでください。** 公開するリポジトリや記事に書かず、`secrets.yaml` に置きます
- Yahoo! の Web API は、原則として**利用者自身の便宜のための非商用目的**で使えます（[WebAPI の商用利用](https://support.yahoo-net.jp/PccDeveloper/s/article/H000011080)）。利用上限は 1 アプリケーションにつき 24 時間で 5 万リクエストで、5 分ごとの取得なら 1 日 288 回です
- Yahoo! 気象情報 API・Google 翻訳 TTS とも、提供の継続は保証されていません。Google 翻訳 TTS は非公式の読み上げ機能を使う統合なので、使えなくなったら Google Cloud TTS や Piper など別の TTS エンジンに差し替えてください
- カレンダーの「終日の予定」は無視します。終日予定を入れると、ずっと発話しなくなるためです

## ライセンス

[MIT](LICENSE)

---

Webサービス by Yahoo! JAPAN （https://developer.yahoo.co.jp/sitemap/）
