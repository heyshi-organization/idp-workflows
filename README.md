# idp-workflows

Platform teamが管理する、CI/CDのReusable workflow(`on: workflow_call`)を集約するリポジトリ。

`idp-gitops-application`(および将来追加される開発チームリポジトリ)側の`.github/workflows/`は、
ビルド・イメージpush・K8sマニフェストのimage tag書き換えといった実処理を持たず、ここで定義される
Reusable workflowを`uses:`で呼び出し、アプリ名やパスなどを`with:`で渡すだけの薄い定義になる。

## 目的

- CI/CDのベースロジックをPlatform teamが一元管理し、変更・改善を全開発チームに横展開できるようにする
- 開発チーム側は「何をどこにデプロイするか」だけを意識すればよく、ビルドパイプラインの実装詳細を
  各リポジトリで重複して持たない

## ディレクトリ構成(予定)

```
idp-workflows/
└── .github/workflows/
    ├── build-and-push.yml   … on: workflow_call。イメージビルド・push・manifestのimage tag書き換え
    └── smoke-test.yml       … on: workflow_call。Argo CDのsync/health確認・アプリ疎通確認
```

呼び出し側(`idp-gitops-application`)の例:

```yaml
jobs:
  call-ci:
    uses: heyshi-organization/idp-workflows/.github/workflows/build-and-push.yml@main
    with:
      app-path: apps/sample-app
      manifest-path: manifests/sample-app/deployment.yaml
      image-name: sample-app
    secrets: inherit
```
