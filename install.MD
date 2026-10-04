# ESP-WebSDR インストール・オフライン運用ガイド

このフォーク（`NW-Lab/esp-web-sdr`）は、ESP32系チップを ESP-SDR として利用するブラウザベースのスペクトラムビューアーと、対応ファームウェア用のブラウザフラッシャーです。アプリ、フラッシャー、依存ライブラリ、`firmware/` 配下のファームウェアイメージを同じリポジトリーに含めています。

> **オフライン利用:** このリポジトリーを取得した後は、ローカルサーバー経由のビューアー利用と、同梱イメージを使うブラウザ上のファームウェア書き込みにインターネット接続は必要ありません。

## 1. 必要なもの

| 項目 | 用途 |
|---|---|
| デスクトップ版 Chrome / Edge / Brave | Web Serial によるESP32とのUSBシリアル通信 |
| Python 3 | `localhost` 上の静的Webサーバー起動（追加パッケージ不要） |
| 対応ESP32ボードとUSBケーブル | ESP-SDRの受信・ファームウェア書き込み |

他のシリアルモニター、Arduino IDE、esptool、SDRソフトウェアなどが同じポートを開いている場合は、先に閉じてください。

## 2. リポジトリーを取得する

ネット接続がある環境で、フォークをクローンします。

```sh
git clone https://github.com/NW-Lab/esp-web-sdr.git
cd esp-web-sdr
```

以降の起動・閲覧・同梱ファームウェアの選択はオフラインで行えます。ZIPで取得した場合も、展開後に同じ手順で起動できます。

## 3. オフラインでローカル起動する

ESP-WebSDR は Web Serial を使うため、`index.html` を直接ダブルクリックして `file:///...` で開くことはできません。必ず `localhost` のローカルサーバー経由で開いてください。

### Windows

`start-windows.bat` をダブルクリックするか、コマンドプロンプトから実行します。

```bat
start-windows.bat
```

### macOS / Linux

```sh
chmod +x start-linux-macos.sh
./start-linux-macos.sh
```

起動後、ブラウザで次を開きます。

```text
http://localhost:8765/
```

ポート `8765` が使用中の場合は、任意の空きポートを指定できます。

```sh
./start-linux-macos.sh --port 9000
```

Windowsでは次のように指定します。

```bat
start-windows.bat --port 9000
```

サーバーは `127.0.0.1` のみに待ち受けるため、LANには公開されません。停止するにはサーバーを起動したターミナルで `Ctrl+C` を押します。

## 4. ESP-WebSDR を使う

1. `http://localhost:8765/` を開きます。
2. **Connect ESP-SDR** を選択し、対象ボードのUSBシリアルポートを選びます。
3. 中心周波数、ゲイン、サンプルレート、FFTサイズを設定します。
4. スペクトラムとウォーターフォールを確認します。

### Wi-Fi CH 表示

スペクトラム下部の測定バーに **Wi-Fi CH** を表示します。

- 中心周波数が標準チャンネル中心周波数と一致する場合: `CH 1 · 2.4 GHz` のように表示
- 中心周波数が一致しない場合: 受信スパンに含まれる標準チャンネルを要約表示
- 対応する対応表: 2.4 GHz（CH 1–14）、主要5 GHz帯（CH 36–177）、6 GHz帯（CH 1–233）

> この表示は周波数対応表です。ボードの実際の受信可能範囲、地域の電波利用規則、またはチャンネル利用可否を保証するものではありません。

## 5. ファームウェアを書き込む

ブラウザフラッシャーはローカルで起動できます。

1. ローカルサーバーを起動します。
2. `http://localhost:8765/flash.html` を開きます。
3. 実際のチップ種別とフラッシュ容量に一致するプロファイルを選びます。
4. **Connect device** を選び、対象USBシリアルデバイスを選択します。
5. **Install ESP-SDR firmware** を選びます。

> **重要:** この操作は既存のファームウェアとパーティションテーブルを置き換えます。チップ種別・フラッシュ容量が不明な場合は書き込まないでください。起動モードに入らない場合は BOOT を押したまま RESET または再接続を行います。

### esptool を使う場合

esptoolで書き込む運用では、選択したプロファイルの次のファイルをオフラインで参照できます。

```text
firmware/<profile>/flash_command.txt
firmware/<profile>/flash_args
```

実行するコマンド、シリアルポート、ボーレート、およびチップ指定を実機に合わせて確認してください。`flash_command.txt` は同梱イメージの書き込みアドレスを確認するための資料です。

## 6. トラブルシューティング

| 現象 | 確認事項 |
|---|---|
| `WebSerial support is required` | Chrome / Edge / Braveのデスクトップ版を使用しているか確認 |
| `Serve this page over HTTPS or localhost` | `file://` ではなく `http://localhost:8765/` で開いているか確認 |
| ポートを選べない／接続できない | 別のアプリがシリアルポートを使用していないか確認。USBケーブル・ドライバーも確認 |
| フラッシャーが起動モードへ入れない | BOOTを押したままRESETまたは再接続し、BOOTを離して再試行 |
| `Address already in use` | `--port 9000` など別のポート番号で起動 |

## 7. 開発者向け検証

Node.js がある環境では、ビューアー・フラッシャー・Wi-Fi CH 対応表の回帰テストを実行できます。

```sh
node --test tests/*.test.mjs
```

このフォークは GPL-3.0-or-later です。詳細は [LICENSE](LICENSE) と、フラッシャー依存関係の `flasher/vendor/LICENSE-*` を参照してください。
