# スケーリング

## 概要

スケーリングは、ワークロードの容量を需要に合わせて増減すること。水平スケーリングはPodのレプリカ数を増減し、垂直スケーリングは各Podに与えるCPUやメモリを調整する。

## 手動でDeploymentを水平スケール

`default` Namespaceの`web-demo`を1レプリカから3レプリカへ変更した。

```bash
kubectl get deployment web-demo
kubectl scale deployment/web-demo --replicas=3
kubectl get deployment web-demo
```

変更前の確認:

```text
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
web-demo   1/1     1            1           23h
```

スケール要求の結果:

```text
deployment.apps/web-demo scaled
```

変更後:

```text
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
web-demo   3/3     3            3           23h
```

## PodとServiceの確認

```bash
kubectl get pods -l app=web-demo -o wide
kubectl get endpointslices -l kubernetes.io/service-name=web-demo
```

確認結果:

| Pod | Ready | IP | Node |
|---|---:|---|---|
| `web-demo-5df467d9c5-6f5x5` | 1/1 | `10.244.0.2` | `desktop-control-plane` |
| `web-demo-5df467d9c5-ncv8g` | 1/1 | `10.244.0.10` | `desktop-control-plane` |
| `web-demo-5df467d9c5-rs76n` | 1/1 | `10.244.0.11` | `desktop-control-plane` |

EndpointSliceには3つのPod IPが登録され、Serviceが3レプリカをバックエンドとして認識していた。

## 気づき

- `kubectl scale`でDeploymentの望むレプリカ数を変えると、ReplicaSetがPod数を調整する。
- Pod一覧の`READY 1/1`は、そのPodで通常コンテナ1つがReadyであることを示す。このDeploymentの各Podにはnginxコンテナが1つある。3 Pod合計でアプリケーションコンテナは3つ。
- 3 Podすべて同じ`desktop-control-plane` Nodeに配置された。Podを増やす水平スケールとNodeを増やすクラスターのスケールは別のこと。
- ServiceのEndpointSliceにも3つのPod IPが入り、DeploymentのPodがServiceの接続先に加わった。
- この学習クラスターは1 Nodeなので、レプリカを増やしてもNode障害への冗長性は得られない。

## 次に学ぶこと

アプリケーションの更新、状態確認、ログ確認など、Kubernetes上でのアプリケーション管理を学ぶ。

## 参考

- [Kubernetes公式: 手動でDeploymentをスケール](https://kubernetes.io/docs/tasks/run-application/scale-deployment/)
- [Kubernetes公式: スケーリング](https://kubernetes.io/docs/concepts/workloads/autoscaling/)
