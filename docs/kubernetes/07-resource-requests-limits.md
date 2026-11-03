# CPU・メモリのrequestsとlimits

## 概要

コンテナが必要とするCPU・メモリ量と、利用できる上限を指定する設定。

- **request**: スケジューラーがPodを配置できるNodeか判断するときに使う要求量。実使用量の固定割り当てではなく、利用可能ならコンテナはrequestを超えて使える。
- **limit**: コンテナが利用できる上限。CPU limitはスロットリングで制限され、メモリlimitを超えた場合はコンテナがOOM killされることがある。

## 単位

- CPU `100m`は100ミリCPU、つまり1 CPUコア相当の`0.1`。
- メモリ`Mi`はメビバイト（Mebibyte）。`1Mi = 1,048,576 bytes`で、`64Mi = 67,108,864 bytes`。
- `Mi`は2進単位、`M`は10進単位。大文字小文字に注意する。

## 実習

`k8s-learning` Namespaceにある`config-demo` Deploymentのコンテナへ、学習用の値を設定した。

```bash
kubectl set resources deployment/config-demo -n k8s-learning --requests=cpu=100m,memory=64Mi --limits=cpu=250m,memory=128Mi
kubectl describe deployment config-demo -n k8s-learning
```

`set resources`の実行結果:

```text
deployment.apps/config-demo resource requirements updated
```

`describe`で確認した値:

```text
Limits:
  cpu:     250m
  memory:  128Mi
Requests:
  cpu:     100m
  memory:  64Mi
```

Deploymentの`revision`は4となり、新しいReplicaSet`config-demo-884d5bc9d`が1 Podを作成、以前のReplicaSetは0 Podへ縮小された。これはPodテンプレートのリソース設定を変更したことによるロールアウト。

## 気づき

- `kubectl describe deployment`でPodテンプレートに設定されたrequests/limitsを確認できる。
- requestsとlimitsはDeploymentのPodテンプレートに記述され、その設定から作られるPodに適用される。
- 単一Nodeの学習クラスターでは配置先の違いは観察しにくいが、requestはスケジューリングに使われる。
- メモリrequestは上限ではない。メモリ使用上限はlimitで指定する。

## 参考

- [Kubernetes公式: コンテナとPodのリソース管理](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [kubectl set resources公式リファレンス](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_set/kubectl_set_resources/)

## 次に学ぶこと

Deploymentのレプリカ数を変更し、スケーリングを観察する。
