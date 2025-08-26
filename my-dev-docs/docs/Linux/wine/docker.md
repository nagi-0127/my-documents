# Wineコンテナ

## Dockerfile

```dockerfile
FROM ubuntu:24.04 AS base

ENV WINEPREFIX=/wine
ENV XDG_CONFIG_HOME=/weston

RUN apt update && apt install -y wget
# Wineのインストール
RUN wget -nc -O /etc/apt/keyrings/winehq-archive.key https://dl.winehq.org/wine-builds/winehq.key && wget -nc -P /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/ubuntu/dists/jammy/winehq-jammy.sources
# 32bitアーキテクチャ追加
RUN dpkg --add-architecture i386
# 色々インストール
RUN apt update &&\
    apt-get install -y \
        # Wine本体
        winehq-stable\
        # winetricks(windows追加機能インストール用)
        winetricks \
        # 音声転送
        pulseaudio \
        # プロセス管理
        supervisor \
        # 画面転送
        weston \
    && apt clean

# DirectXインストール directx9 dxvk
RUN winetricks cjkfonts directx9 dxvk

COPY ./weston_entrypoint.sh /usr/local/bin/entrypoint.sh
COPY weston.ini ${XDG_CONFIG_HOME}/weston.ini
COPY ./launch_weston.sh /usr/local/bin/launch_weston.sh
COPY ./launch.sh /usr/local/bin/start.sh
COPY ./pulse.sh /usr/local/bin/pulse.sh
COPY sakura.exe /tmp/sakura.exe

RUN chmod +x /usr/local/bin/launch_weston.sh
RUN chmod +x /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/start.sh
RUN chmod +x /usr/local/bin/pulse.sh

RUN { \
  echo "[supervisord]"; \
  echo "user=root"; \
  echo "nodaemon=true"; \
  echo "logfile=/var/log/supervisor/supervisord.log"; \
  echo "childlogdir=/var/log/supervisor"; \
  echo "[program:dbus]"; \
  echo "command=/usr/bin/dbus-daemon --system --nofork --nopidfile"; \
  echo "[program:weston]"; \
  echo "user=wineuser"; \
  echo "command=/bin/sh /usr/local/bin/launch_weston.sh"; \
  echo "[program:pulseaudio]"; \
  echo "user=wineuser"; \
  echo "command=/bin/sh /usr/local/bin/pulse.sh"; \
} > /etc/rdp.conf

RUN useradd -m wineuser
# RUN mkdir ${WINEPREFIX} && chown wineuser ${WINEPREFIX}
# USER wineuser

ENTRYPOINT [ "/usr/local/bin/entrypoint.sh" ]
```

## entrypoint.sh

pulseaudioの起動がうまくいかない。wineuserで起動しているはずだが、rootのconfigを読み込もうとし権限不足となってしまう。

```sh
#!/bin/bash -e

USER_ID=$(id -u wineuser)
GROUP_ID=$(id -g wineuser)

export RUNTIME_DIR=/run/user/${USER_ID}
export XDG_RUNTIME_DIR=/run/user/${USER_ID}
install -o $USER_ID -g $GROUP_ID -m 0700 -d $RUNTIME_DIR
chown wineuser $RUNTIME_DIR
chown wineuser $WINEPREFIX

openssl genrsa -out /home/wineuser/rsa.key 2048
chown wineuser /home/wineuser/rsa.key

mkdir -p /root/.config/pulse
chown wineuser /root/.config/pulse

supervisord -c /etc/rdp.conf
```

## launch_weston.sh

```sh
#!/bin/bash -e

weston --backend=rdp --rdp4-key=/home/wineuser/rsa.key
```

## start.sh

westonの起動時スクリプト

```
#!/bin/bash -e

wine /tmp/sakura.exe
```