# ConfigMapとSecret

## ConfigMap

### 概要

ConfigMapは、秘密ではない設定値をキーと値で保存するAPIオブジェクト。コンテナイメージと環境別の設定を分離でき、Podから環境変数やファイルとして利用できる。

### 実習

#### NamespaceにConfigMapを作成

```bash
kubectl create configmap app-settings --from-literal=APP_MODE=learning -n k8s-learning
kubectl get configmap app-settings -n k8s-learning -o yaml
```

実行結果:

```text
configmap/app-settings created
```

YAMLの`data`:

```yaml
data:
  APP_MODE: learning
```

#### Deploymentから参照してPodで確認

```bash
kubectl create deployment config-demo --image=nginx:alpine -n k8s-learning
kubectl set env deployment/config-demo --from=configmap/app-settings -n k8s-learning
kubectl rollout status deployment/config-demo -n k8s-learning
kubectl exec deployment/config-demo -n k8s-learning -- printenv APP_MODE
```

確認結果:

```text
deployment.apps/config-demo created
deployment.apps/config-demo env updated
deployment "config-demo" successfully rolled out
learning
```

### 仕組みと気づき

- ConfigMapを作成しただけではPodに設定されない。Podテンプレート側から参照する必要がある。
- `kubectl set env --from=configmap/app-settings`はConfigMapの各キーを環境変数としてPodテンプレートに設定する。この実習では`APP_MODE`が`app-settings`の`APP_MODE`キーを参照した。
- ConfigMapと参照するPodは同じNamespaceに置く必要がある。今回、両方を`k8s-learning`に作成した。
- DeploymentのPodテンプレートを更新すると新しいPodへのロールアウトが起きる。
- ConfigMapを環境変数として使う場合、値はPod起動時に読み込まれる。ConfigMapを後から変更しても、既存Podの環境変数は自動更新されない。
- ConfigMapにはパスワードやトークンなどの秘密情報を保存しない。

## Secret

Secretはパスワード、トークン、鍵などの機密情報を保存するためのAPIオブジェクト。ConfigMapと似た方法でPodに渡せる。

### 注意点

Secretの値は通常のYAML出力ではBase64表現になるが、Base64は暗号化ではない。KubernetesのSecretデータは、初期設定ではetcdに暗号化されず保存される。実際の環境ではRBACで閲覧権限を絞り、必要に応じて保存時暗号化などを設定する。実習では実在するパスワードやトークンを使わない。

### SecretをDeploymentから使う実習

架空の値`not-a-real-token`だけを使った。

```bash
kubectl create secret generic demo-credentials --from-literal=DEMO_TOKEN=not-a-real-token -n k8s-learning
kubectl get secret demo-credentials -n k8s-learning -o yaml
kubectl set env deployment/config-demo --from=secret/demo-credentials -n k8s-learning
kubectl rollout status deployment/config-demo -n k8s-learning
kubectl describe deployment config-demo -n k8s-learning
kubectl exec deployment/config-demo -n k8s-learning -- printenv DEMO_TOKEN
```

確認結果:

```text
secret/demo-credentials created
deployment.apps/config-demo env updated
deployment "config-demo" successfully rolled out
not-a-real-token
```

SecretのYAML出力では`DEMO_TOKEN`の値が`bm90LWEtcmVhbC10b2tlbg==`と表示された。これはサンプル値のBase64表現。

Deploymentの`describe`では次の参照が見えた:

```text
APP_MODE:    <set to the key 'APP_MODE' of config map 'app-settings'>
DEMO_TOKEN:  <set to the key 'DEMO_TOKEN' in secret 'demo-credentials'>
```

## rolloutの観察

- `kubectl set env`がDeploymentのPodテンプレートを変更するため、Deploymentコントローラーが新しいReplicaSetを作り、Podを置き換える。今回の`revision: 3`と新旧ReplicaSetのイベントがその証拠。
- `kubectl rollout status`はロールアウトを開始するコマンドではなく、進行状況を待って結果を表示するコマンド。
- ロールアウト前後で同じ設定値を使っていれば、`describe`の参照内容も`printenv`の値も同じに見えるのが正常。Podの個体は置き換わっても、渡すべき値が同じだから。
- ConfigMap/Secretの値だけを更新しても、環境変数として起動済みPodに渡された値は自動で変わらない。新しいPodを作成して反映する必要がある。

ロールアウト後にReplicaSetとPodを確認した。

```bash
kubectl get replicasets,pods -n k8s-learning
```

```text
NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/config-demo-7bb68b6558   0         0         0       32m
replicaset.apps/config-demo-7d5bcf598c   0         0         0       34m
replicaset.apps/config-demo-d858f9587    1         1         1       20m

NAME                              READY   STATUS    RESTARTS   AGE
pod/config-demo-d858f9587-94pd8   1/1     Running   0          20m
```

新しいReplicaSetが1 PodをReadyで維持し、古いReplicaSetは0に縮小されていた。古いReplicaSetの記録が一覧に残るため、ロールアウト前後の履歴を確認できる。

### 参考

- [Kubernetes公式: ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Kubernetes公式: Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Kubernetes公式: Secretのセキュリティベストプラクティス](https://kubernetes.io/docs/concepts/security/secrets-good-practices/)
