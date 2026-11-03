# Service

## 概要

Serviceは、Pod群へのネットワーク接続を安定して提供するKubernetesリソース。Serviceは対象Podを選び、クラスター内IPやDNS名でアクセスできるようにする。

## 必要性

Podは再作成・入れ替えでIPアドレスが変わることがある。クライアントが個々のPod IPを直接追跡する代わりにServiceを接続先にすることで、Podの構成が変わっても同じServiceを通じて接続できる。

## 仕組み

```mermaid
flowchart LR
    Client[クラスター内のクライアント] --> S[Service<br/>ClusterIP: 10.96.207.176:80]
    S --> E[EndpointSlice<br/>10.244.0.5:80]
    E --> P[web-demo Pod<br/>Nginx :80]
```

Serviceはselectorに一致するPodを見つけ、Kubernetesがそのバックエンド情報をEndpointSliceに反映する。Serviceの`port`がクライアント側のポート、`targetPort`がPod側の転送先ポート。

## 実習

### DeploymentをServiceとして公開

```bash
kubectl expose deployment web-demo --port=80 --target-port=80
```

実行結果:

```text
service/web-demo exposed
```

### ServiceとEndpointSliceを確認

```bash
kubectl get service web-demo
kubectl get endpointslices -l kubernetes.io/service-name=web-demo
```

実行結果:

```text
NAME       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
web-demo   ClusterIP   10.96.207.176   <none>        80/TCP    46s

NAME             ADDRESSTYPE   PORTS   ENDPOINTS    AGE
web-demo-fsvg9   IPv4          80      10.244.0.5   96s
```

## 実行結果から分かったこと

- Serviceの種類は`ClusterIP`。クラスター内部向けのServiceで、外部IPは割り当てられていないため`EXTERNAL-IP`は`<none>`。
- クラスター内の接続先として`10.96.207.176:80`が割り当てられた。
- Serviceに対応するEndpointSliceがあり、バックエンドとして`10.244.0.5:80`が登録されている。
- ServiceのClusterIPとバックエンドのIPは異なる。クライアントは安定したServiceのIP/名前を使い、KubernetesのServiceネットワーク機能がEndpointSliceに記録されたバックエンドへ転送する。

## 気づき

- `kubectl expose`の成功メッセージはService作成の受付を示す。接続先が登録されたことはServiceとEndpointSliceを確認して判断する。
- ServiceがあるだけではPCやインターネットからアクセスできない。今回の`ClusterIP`はクラスター内向け。
- ServiceのIPとPodのIPを混同しない。Serviceは安定した窓口、EndpointSliceは現在のバックエンド一覧として考えると整理しやすい。

## 参考

- [Kubernetes公式: Service](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Kubernetes公式: EndpointSlices](https://kubernetes.io/docs/concepts/services-networking/endpoint-slices/)

## 次に学ぶこと

Namespaceを作成し、リソースを分けて管理する方法を学ぶ。
