# 環境構築手順
docker network create -d bridge test-net --subnet 192.168.254.0/24 --gateway=192.168.254.1

minikube start -p test-cluster --network=test-net --driver=docker --static-ip=192.168.254.10
minikube start -p staging-cluster --network=test-net --driver=docker --static-ip=192.168.254.20
minikube start -p prod-cluster --network=test-net --driver=docker --static-ip=192.168.254.30

kubectx test-cluster
kubectl create namespace argocd
kubectl apply -k argocd

argocd cluster --kubeconfig ~/.config/minikube_config add staging-cluster
argocd cluster --kubeconfig ~/.config/minikube_config add prod-cluster

# 環境分離手順

## 最終形

- staging-cluster: 本番クラスタを管理
- test-cluster: ステージングとテスト環境を管理

## 手順

- autosyncを切っておく
  - マニフェストから切らないと元に戻ってしまう
- バックアップを取る
  - argocd admin export
- 環境の切り分け
  - クラスタごとに設定を切り分けられればいいんだけど
- 元にもどらないように慎重に設定を消していく
  - applicationsetから該当の環境の設定を消す
  - non-cascading delete（設定だけ消してリソースは消さない）をする
  - argocd appset deleteに--cascadeオプションはまだ実装されていないので、消すとリソースごと消えてしまう！

- 元クラスタはapp of appsになっているのでそのように組み替え
  - めんどくさいけどリポジトリをつくってテストするか・・・


## 認証
