# Namespace

## 概要

Namespaceは、Kubernetesクラスター内のリソースを論理的に分けるための区画。同じクラスターで複数の環境やチームのリソースを整理するときに使える。

## 必要性

Namespaceを指定して操作すると、リソースの確認や管理の範囲を分けられる。異なるNamespaceなら、DeploymentやServiceなどの名前をそれぞれの範囲内で再利用できる。Namespace自体はNodeや仮想マシンではなく、リソースの分類単位。

## 実習

### 学習用Namespaceを作成

```bash
kubectl create namespace k8s-learning
```

実行結果:

```text
namespace/k8s-learning created
```

この操作でNamespaceオブジェクトが1つ追加された。既存の`web-demo` DeploymentとServiceは移動せず、`default` Namespaceに残る。

### Namespace一覧を確認

```bash
kubectl get namespaces
```

実行結果:

```text
NAME                 STATUS   AGE
default              Active   22h
k8s-learning         Active   18s
kube-node-lease      Active   22h
kube-public          Active   22h
kube-system          Active   22h
local-path-storage   Active   22h
```

`k8s-learning`は`Active`で表示され、クラスター内で利用できる状態だった。

### 入力ミス

```bash
kubectl get namespases
```

実行結果:

```text
error: the server doesn't have a resource type "namespases"
```

`namespases`はリソース名の入力ミス。kubectlが未知のリソースタイプとして拒否し、状態は変更されなかった。正しい複数形は`namespaces`。短縮形の`ns`も利用できる。

### Namespaceごとのリソースを比較

```bash
kubectl get deployments,services,pods -n default
kubectl get deployments,services,pods -n k8s-learning
```

確認結果:

- `default`には`web-demo` Deploymentが1つ、`web-demo`と`kubernetes` Serviceが2つ、`web-demo` Podが1つあった。
- `k8s-learning`にはDeployment・Service・Podがなく、`No resources found in k8s-learning namespace.`と表示された。
- これはエラーではない。Namespace自体は存在して`Active`だが、まだリソースを作成していない状態。
- `-n <namespace>`は、一覧表示の対象Namespaceを指定するオプション。

## 気づき

- `kubectl create namespace ...`はクラスターの状態を変更するコマンド。
- `kubectl get namespaces`はNamespace一覧を読むコマンド。
- NamespaceはPodを実行するNodeとは別の概念。Nodeは実行場所、Namespaceはリソースの論理的な区画。

## 次のテーマ

[ConfigMapとSecret](06-configmap-secret.md)で、設定値をNamespace内のPodから使う方法を学ぶ。
