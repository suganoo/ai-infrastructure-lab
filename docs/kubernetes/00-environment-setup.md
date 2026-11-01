# Kubernetes学習環境の準備

## 概要

Windows 11上のWSL2 UbuntuからDocker Desktopを利用できるようにし、Docker Desktop上で学習用Kubernetesクラスターを起動し始めた。

## 環境

- Windows 11 Home 25H2（OSビルド 26200.9711）
- WSL 2.7.13.0
- WSLカーネル 6.18.33.2-2
- Ubuntu 26.04（WSL2）
- Docker Desktop 4.97.0
- Docker CLI 29.9.2
- PCメモリ 32GB
- GPU: AMD Radeon 860M（この学習ではNVIDIA GPU/CUDAを前提としない）

## 実施したこと

1. 作業フォルダの状態を確認した。既存ファイルはREADME.mdのみで、Gitは`main`ブランチ、確認時点で変更なし。
2. Windows 11、WSL2、Ubuntuの状態を確認した。UbuntuはWSL2で登録されていた。
3. Docker DesktopをWindowsに導入して起動した（ユーザーが実施）。
4. Docker Desktopの設定でWSL 2を選択し、既定のWSLディストリビューションとの連携を有効にした。Ubuntuが既定のディストリビューション。
5. UbuntuターミナルからDocker CLIとサーバーへの接続を確認した。
6. 現在、Docker DesktopでKubernetesクラスターを起動中（ユーザー報告）。起動完了後にkubectlで状態を確認する。

## 実習コマンドと確認結果

### Docker CLIのバージョン

```bash
docker --version
```

確認結果:

```text
Docker version 29.9.2, build a82e528
```

### Dockerエンジンへの接続

```bash
docker info
```

主な確認結果:

- Client/ServerともにDocker 29.9.2
- ServerのOperating Systemは`Docker Desktop`
- ServerのKernel Versionは`6.18.33.2-microsoft-standard-WSL2`
- Architectureは`x86_64`
- Dockerエンジンから見えるCPUは16、メモリは14.52 GiB
- 確認時点でコンテナ数0、イメージ数0

## 気づき

- UbuntuのWSL連携を有効にすると、Ubuntu内にDocker Engineを別途導入しなくてもDocker Desktopのエンジンを利用できる。
- `docker --version`はCLIの存在を確認し、`docker info`はクライアントからサーバーへの接続とサーバー情報を確認する。
- Docker DesktopのKubernetesクラスターは現在起動中のため、NodeやPodの状態はまだ記録していない。

## 次に確認すること

クラスター起動後、Ubuntuから以下を実行してKubernetesの接続先とNodeを確認する。

```bash
kubectl config current-context
kubectl get nodes
```
