# Kubernetes上でのアプリケーション管理

## 宣言的な管理

Kubernetesリソースの望む状態をYAMLマニフェストに記述し、`kubectl apply`でクラスターへ反映できる。マニフェストをGitで管理すると、変更の記録・レビュー・再適用がしやすい。

この実習では、リポジトリ内の次のファイルでDeploymentを定義した。

```text
kubernetes/manifests/manifest-demo.yaml
```

主な定義:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: manifest-demo
  namespace: k8s-learning
spec:
  replicas: 2
```

`kind`はリソースの種類、`metadata.name`はDeployment名、`metadata.namespace`は所属Namespace、`spec.replicas`は望むPod数を示す。

## 実習結果

最初に`replicas: 1`のマニフェストを適用した。

```text
deployment.apps/manifest-demo created
```

DeploymentとPodがそれぞれ1つで、Podは`Running`だった。

次にファイルの`replicas`を2に変更し、もう一度適用した。

```text
deployment.apps/manifest-demo configured
```

確認結果:

```text
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
manifest-demo   2/2     2            2           7m38s

NAME                             READY   STATUS    RESTARTS   AGE
manifest-demo-7566b694d5-c9mkb   1/1     Running   0          8s
manifest-demo-7566b694d5-t46k5   1/1     Running   0          7m43s
```

## 定義ファイルとクラスター内オブジェクト

- **マニフェストの定義元**は、作業リポジトリの`kubernetes/manifests/manifest-demo.yaml`。
- **ライブオブジェクト**は、`kubectl apply`がKubernetes APIに送信し、クラスター内に作成・保存されたDeployment`manifest-demo`。
- `kubectl get deployment manifest-demo -n k8s-learning -o yaml`で、クラスター側の現在の定義を取得できる。出力にはAPIサーバー等が設定する項目も含まれるため、リポジトリのソースファイルとは同一ではない。
- マニフェストの`replicas`を1から2に変更して再適用すると、DeploymentコントローラーがPod数を2に合わせた。

### ライブオブジェクトYAMLの確認

```bash
kubectl get deployment manifest-demo -n k8s-learning -o yaml
```

確認した項目:

- `metadata.annotations.kubectl.kubernetes.io/last-applied-configuration`には最後に適用したマニフェストのJSONが保存され、`replicas: 2`だった。
- `metadata.generation: 2`はspecが作成後に更新されたことを示す。
- `metadata.annotations.deployment.kubernetes.io/revision: "1"`はDeploymentのPodテンプレートの履歴番号。レプリカ数の変更はPodテンプレート変更ではないためrevisionは増えない。
- `spec.replicas: 2`は望む状態。
- `status.replicas: 2`と`status.readyReplicas: 2`は、観測時の現在数とReady数。`availableReplicas: 2`も2つが利用可能であることを示す。
- `spec.strategy`, `imagePullPolicy`, `protocol`, `restartPolicy`など、マニフェストに明示していない値もAPI既定値としてライブ設定に現れる。
- `resources: {}`はこのDeploymentのnginxコンテナにrequests/limitsを指定していないことを示す。

## 次に学ぶこと

Kubernetes上の基本的なアプリケーション管理を実施した。次はKubeAIへの接続方法と、Kubernetes上で推論サービスを管理する仕組みを学ぶ。

## 参考

- [Kubernetes公式: 設定ファイルによる宣言的管理](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/)
- [Kubernetes公式: オブジェクトの操作](https://kubernetes.io/docs/concepts/overview/working-with-objects/)
