# idp-workflows

プラットフォームチームが管理する、CI/CDのReusable workflow(`on: workflow_call`)を集約するリポジトリ。

`idp-gitops-application`(および将来追加されるプロダクトチームのリポジトリ)側の`.github/workflows/`は、
ビルド・イメージpush・K8sマニフェストのimage tag書き換えといった実処理を持たず、ここで定義される
Reusable workflowを`uses:`で呼び出し、アプリ名やパスなどを`with:`で渡すだけの薄い定義になる。

現在は未整備。ARC(セルフホストランナー)の導入が終わった後に作成する。

## 目的

- CI/CDのベースロジックをプラットフォームチームが一元管理し、変更・改善を全プロダクトチームに横展開できるようにする
- プロダクトチーム側は「何をどこにデプロイするか」だけを意識すればよく、ビルドパイプラインの実装詳細を
  各リポジトリで重複して持たない

## ディレクトリ構成(予定)

```
idp-workflows/
└── .github/workflows/
    ├── build-and-push.yml   … on: workflow_call。イメージビルド・push・manifestのimage tag書き換え
    └── smoke-test.yml       … on: workflow_call。Argo CDのsync/health状態の確認
```

CIからはクラスタを変更しない(`kubectl`で直接applyしない)。デプロイ後のアプリの疎通確認は、CIではなく
Argo CDのPostSync Hookで行い、CIはその結果を待つだけにする。

呼び出し側(`idp-gitops-application`)の例:

```yaml
jobs:
  call-ci:
    uses: heyshi-organization/idp-workflows/.github/workflows/build-and-push.yml@main
    with:
      app-path: apps/apple
      manifest-path: manifests/apple/deployment.yaml
      image-name: apple
    secrets: inherit
```
