# xrdp

linuxでのrdpプロトコルを使用したリモートデスクトップサーバ

## 設定ファイル

### `/etc/xrdp/xrdp.ini`

* xrdpサーバ設定(接続ip, port, username, passwordなど)

### `/etc/xrdp/sesman.ini`

* セッションマネージャーの設定ファイル

### `/etc/xrdp/startwm.sh`

* RDP接続時にxrdpが実行するスクリプト

### `~/.xsession`

* ユーザのホームディレクトリ内に作成する。
* **接続ユーザごとにスタートアップアプリケーション等を設定できるっぽい**
