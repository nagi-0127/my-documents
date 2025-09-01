# weston rdp rail shell

WSLgで使われている[weston](https://github.com/microsoft/weston-mirror)をコンテナ内でビルドする。

## Dockerfile

```Dockerfile
FROM ubuntu:24.04 AS base

# add package build dependencies
RUN sed -i 's/^Types: deb$/Types: deb deb-src/' /etc/apt/sources.list.d/ubuntu.sources
RUN apt-get update && apt-get install -y build-essential librsvg2-dev meson ninja-build libwayland-dev libegl-dev libgles-dev libxkbcommon-dev && apt-get build-dep -y weston

WORKDIR /tmp

# クローン or ソースコードをコピー
# clone https://github.com/microsoft/weston-mirror.git
# RUN git clone https://github.com/microsoft/weston-mirror.git
COPY ./weston-mirror /tmp/weston-mirror

# change cwd
WORKDIR /tmp/weston-mirror

ARG PREFIX=/opt/rdprail
ARG LD_LIBRARY_PATH=/opt/rdprail/lib

# build
RUN meson build --prefix=${PREFIX} --libdir=${LD_LIBRARY_PATH} --sysconfdir=/etc \
    # 'Weston backend: DRM/KMS'
    -Dbackend-drm=false \
    # 'DRM/KMS backend support for VA-API screencasting'
    -Dbackend-drm-screencast-vaapi=false \
    # 'Weston backend: headless (testing)'
    -Dbackend-headless=false \
    # 'Weston backend: RDP remote screensharing'
    -Dbackend-rdp=true \
    # 'Compositor: RDP screen-sharing support'
    -Dscreenshare=false \
    # 'Weston backend: Wayland (nested)'
    -Dbackend-wayland=false \
    # 'Weston backend: X11 (nested)'
    -Dbackend-x11=false \
    # 'Weston backend: fbdev'
    -Dbackend-fbdev=false \
    # 'Default backend when no parent display server detected'
    # [ 'auto' 'drm' 'wayland' 'x11' 'fbdev' 'headless' 'rdp' ]
    -Dbackend-default='rdp' \
    # 'Weston renderer: EGL / OpenGL ES 2.x'
    -Drenderer-gl=true \
    # 'Weston launcher for systems without logind'
    -Dweston-launch=false \
    # 'Xwayland: support for X11 clients inside Weston'
    -Dxwayland=false \
    # 'Xwayland: path to installed Xwayland binary'
    -Dxwayland-path='/usr/bin/Xwayland' \
    # 'systemd service plugin: state notify watchdog socket activation'
    -Dsystemd=false \
    # 'wslgd plugin: state notify'
    -Dwslgd=false \
    # 'Virtual remote output with GStreamer on DRM backend'
    -Dremoting=false \
    # 'Virtual remote output with Pipewire on DRM backend'
    -Dpipewire=false \
    # 'Weston shell UI: traditional desktop'
    -Dshell-desktop=true \
    # 'Weston shell UI: fullscreen/kiosk'
    -Dshell-fullscreen=false \
    # 'Weston shell UI: IVI (automotive)'
    -Dshell-ivi=false \
    # 'Weston shell UI: kiosk (desktop apps)'
    -Dshell-kiosk=true \
    # 'Weston shell UI: RDP remote application shell'
    -Dshell-rdprail=true \
    # 'Weston desktop shell: default helper client selection'
    -Ddesktop-shell-client-default='weston-desktop-shell' \
    # 'Compositor color management: lcms'
    -Dcolor-management-lcms=true \
    # 'Compositor color management: colord (requires lcms)'
    -Dcolor-management-colord=true \
    # 'Compositor: support systemd-logind D-Bus protocol'
    -Dlauncher-logind=false \
    # 'JPEG loading support'
    -Dimage-jpeg=false \
    # 'WebP loading support'
    -Dimage-webp=false \
    # 'Sample clients: optimize window resize performance'
    -Dresize-pool=true \
    # 'Tools: screen recording decoder tool'
    -Dwcap-decode=true \
    # 'Tests: output JUnit XML results'
    -Dtest-junit-xml=true \
    # 'Tests: allow running with GL-renderer'
    -Dtest-gl-renderer=true \
    # 'Generate documentation'
    -Ddoc=false
RUN ninja -C build && ninja -C build install

FROM ubuntu:24.04 AS build

COPY --from=base /opt/rdprail /opt/rdprail

ENV LD_LIBRARY_PATH=/opt/rdprail/lib
ENV XDG_RUNTIME_DIR=/tmp


RUN apt-get update\
    && apt-get install -y\
    libwayland-client0\
    libwayland-server0\
    libinput10\
    libwayland-cursor0\
    libfreerdp-server2-2t64\
    libxkbcommon0\
    libpango-1.0-0\
    libpangocairo-1.0-0

ENV XDG_CONFIG_HOME=/tmp
COPY weston.ini ${XDG_CONFIG_HOME}/weston.ini
CMD [ "/opt/rdprail/bin/weston" ]

```

## RAIL設定

* RDP接続側(.rdpファイルに追記)で起動するアプリケーションを指定する。

```
# アプリケーションモードで接続
remoteapplicationmode:i:1
remoteapplicationname:s:hoge
# 接続先で起動したいアプリケーションの起動コマンドを設定
remoteapplicationprogram:s:/opt/rdprail/bin/weston-info
```


## TODO 

* pulseaudio組み込み
* RAILでの起動
  * weston.iniで`autolaunch`or`shell`の設定でできるか検証
