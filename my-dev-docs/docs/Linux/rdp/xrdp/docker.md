# xrdp + docker

## dockerコンテナでxrdp実行

* xrdpとxrdp-sesmanの両方のサービスを起動する必要がある。
* dockerは基本的に1コンテナにつき1サービス(通常はsystemctlなどは使用できない)
* xrdpをコンテナで動かしているDockerfileではsupervisorを使用して、サービスの同時起動を実現している模様
* 他のやり方もある？
