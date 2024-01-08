# argocd-sample

## `values.yaml`の生成・設定

`values.yaml`を生成

```bash
$ helm repo add argo https://argoproj.github.io/argo-helm
$ helm repo update
$ helm show values --version 5.27.1 argo/argo-cd > argo-cd-values.yaml
```

## Helmfile を利用した各環境へのデプロイ

`helmfile`コマンドの `-e` オプションに `environment` 配下に準備した環境名を指定すると、当該環境名の `values.yaml` が適用される。

```bash
$ helmfile apply -e prod -f helmfile-argo-cd.yaml
```

## Argo CD 確認

```bash
$ kubectl get pods -n argocd
```

## ファイルの説明

### `helmfile-argo-cd.yaml`

Argo CD を Kubernetes クラスタへインストールするための Helmfile のマニフェスト

### `environment/prod/argo-cd-values.yaml`

Argo CD を本番環境向けにインストールする際の設定値

## Argo CD の初期設定

Argo CD のサーバーへポートフォワードして、argocd コマンドを実行できるようにする

```bash
$ kubectl port-forward svc/argocd-server -n argocd 8443:443
```

Argo CD の初期パスワードを確認

```bash
$ argocd admin initial-password -n argocd
```

Argo CD ログイン　※ポートフォワードを使用した HTTP 通信を利用するため`--insecure`を指定しているが、本来インターネット経由でアクセスする場合は HTTPS 通信を使用する

```bash
$ argocd login localhost:8443 --insecure --username admin --password {表示されたパスワード}
```

admin ユーザーのパスワードをリセット

```bash
$ argocd account update-password --insecure
```

Argo CD ログアウト

```bash
$ argocd logout localhost:8443
```

パスワードリセット後不要なため、初期パスワード削除

```bash
$ kubectl delete secret -n argocd argocd-initial-admin-secret
```

## Argo CD とアプリケーションの連携

```bash
$ kubectl apply -f application.yaml
```

## GitHub と連携してログインする方法

- GitHub OAuth Apps のトークン作成

```bash
$ echo -n 'kubernetes-secret-password' | base64
```

kubectl edit configmap argocd-cm -n argocd
kubectl edit -n argocd secret argocd-secret
kubectl apply -f github-https-repo-secret.yaml

# deployment.yaml で serviceAccount を作成してアプリ連携

# GitHub OAuth Apps でのログイン

https://github.com/argoproj/argo-cd/blob/master/docs/getting_started.md
https://argo-cd.readthedocs.io/en/stable/operator-manual/user-management/

# Helm の使用
