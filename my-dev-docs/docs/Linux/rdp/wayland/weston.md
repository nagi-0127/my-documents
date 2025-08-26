# weston

## 設定ファイル

* シングルアプリケーション用の設定ファイル

```ini
[core]
; kiosk shell (単一アプリケーション)
shell=kiosk

[autolaunch]
; weston起動時のスクリプトを指定
path=/usr/local/bin/start.sh
; pathで指定したスクリプトの実行が終了したらwestonも終了するように設定
watch=true

[output]
; 詳細不明（とりあえずないと表示されない）
name=wine
app-ids=wine
```