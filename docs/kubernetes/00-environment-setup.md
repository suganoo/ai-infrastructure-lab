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
6. Docker DesktopでKubernetesクラスターを起動した（ユーザー実施）。Ubuntuからkubectlで接続し、Nodeが`Ready`であることを確認した。

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

### Kubernetesクラスターへの接続

```bash
kubectl config current-context
kubectl get nodes
```

実行結果:

```text
docker-desktop

NAME                    STATUS   ROLES           AGE    VERSION
desktop-control-plane   Ready    control-plane   2m1s   v1.36.1
```

確認できたこと:

- kubectlの接続先コンテキストは`docker-desktop`。
- `desktop-control-plane`というNodeが1台あり、状態は`Ready`。
- このNodeは`control-plane`の役割を持ち、Kubernetesのバージョンは`v1.36.1`。

### NamespaceとシステムPodの観察

```bash
kubectl get namespaces
kubectl get pods --all-namespaces
```

実行結果:

```text
NAME                 STATUS   AGE
default              Active   3m7s
kube-node-lease      Active   3m7s
kube-public          Active   3m7s
kube-system          Active   3m7s
local-path-storage   Active   3m3s
```

全NamespaceのPodは9個で、すべて`READY`が`1/1`、`STATUS`が`Running`、再起動回数が0だった。

確認できた主なPod:

- `kube-system`: CoreDNSが2個、etcd、kindnet、kube-apiserver、kube-controller-manager、kube-proxy、kube-scheduler
- `local-path-storage`: local-path-provisionerが1個

## 用語メモ

- `default`は、Namespaceを指定せずに作成したリソースが入る基本Namespace。
- `kube-system`はKubernetesシステムのリソース用Namespace。
- `kube-node-lease`にはNodeのハートビートに使うLeaseオブジェクトが置かれる。
- `kube-public`は認証されていない利用者からも読み取り可能なNamespaceとして作られる。
- `local-path-storage`はこのクラスターに追加されたストレージ関連Namespace。名前から存在を確認できたが、用途の詳細や設定はまだ調べていない。
- CoreDNSはクラスター内DNSを提供する。PodやServiceの名前解決を担う。
- etcdはKubernetes APIの状態を保存するキーバリューストア。
- kube-apiserverはKubernetes APIを提供し、kubectlなどのクライアントが操作する入口。
- kube-controller-managerは各種コントローラーを動かし、望ましい状態に近づける処理を担う。
- kube-schedulerは配置先が決まっていないPodのNodeを選ぶ。
- kube-proxyはServiceのネットワーク動作を支える。
- `kindnet`というPod名から、このクラスターはkind系のネットワーク構成を使っていると読み取れる。

### Docker Desktop終了後の再開確認

Docker Desktopを終了した後に再起動し、既存のクラスターとワークロードを確認した。

```bash
kubectl get nodes
kubectl get deployments,services,pods
```

確認結果:

- `desktop-control-plane`は`Ready`。クラスター作成から22時間経過していた。
- `web-demo` Deploymentは`1/1`、Serviceは`10.96.207.176:80`のまま存在していた。
- `web-demo` Podは`1/1 Running`。再起動回数は1回で、確認時の118秒前に再起動していた。出力だけから再起動の原因は特定できない。

この確認では、Docker Desktopを終了・再起動した後もクラスターのリソースが残っていることを観察できた。

## 気づき

- UbuntuのWSL連携を有効にすると、Ubuntu内にDocker Engineを別途導入しなくてもDocker Desktopのエンジンを利用できる。
- `docker --version`はCLIの存在を確認し、`docker info`はクライアントからサーバーへの接続とサーバー情報を確認する。
- Kubernetesクラスターは1台のNodeで起動し、Nodeの`Ready`状態を確認できた。
- NamespaceとシステムPodを一覧表示した。CoreDNS、APIサーバー、etcdなどの基盤Podが`Running`であることを確認した。
- `web-demo` Deploymentを作成し、Deployment・ReplicaSet・Podがそれぞれ1つで、Podが`Running`かつ`Ready`であることを確認した。詳細は[Deploymentの学習記録](03-deployment.md)を参照。
- `web-demo`用のClusterIP Serviceを作成し、ClusterIPとEndpointSliceのバックエンドを確認した。詳細は[Serviceの学習記録](04-service.md)を参照。
- `k8s-learning` Namespaceを作成し、一覧で`Active`状態を確認した。`namespases`という入力ミスはエラーになったが、状態変更は発生しなかった。詳細は[Namespaceの学習記録](05-namespace.md)を参照。

## 次に確認すること

初期環境の確認と基礎テーマの実習結果はこの記録にまとめた。各テーマの詳細は[Deployment](03-deployment.md)、[Service](04-service.md)、[Namespace](05-namespace.md)などの個別記録を参照。基礎編の次のテーマはKubeAI。
