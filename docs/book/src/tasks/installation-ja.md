# インストール

## 前提条件

- ローカル環境に [kubectl] をインストールしセットアップします
- [management cluster] をインストールし構成します

## マネジメント クラスターのセットアップ

Cluster API は kubectl を介してアクセスできる既存の Kubernetes クラスターを必要とします。以下のオプションから 1 つ選択します:

1. **Kind**

{{#tabs name:"kind-cluster" tabs:"AWS|Azure|GCP|vSphere|OpenStack,Docker"}}
{{#tab AWS|Azure|GCP|vSphere|OpenStack}}

<aside class="note warning">

<h1>注意</h1>

**サポートされる [kind] の最小のバージョン** : v0.6.x

[kind] は本番環境用に設計されたものではなく、開発環境のみを対象としています。

</aside>

  ```bash
  kind create cluster --name=clusterapi
  kubectl cluster-info --context kind-clusterapi
  ```
{{#/tab }}
{{#tab Docker}}

<aside class="note warning">

<h1>注意</h1>

**サポートされる [kind] の最小のバージョン** : v0.6.x

[kind] は本番環境用に設計されたものではなく、開発環境のみを対象としています。

</aside>

<aside class="note warning">

<h1>注意</h1>

Docker プロバイダーは本番環境用に設計されたものではなく、開発環境のみを対象としています。

</aside>

  Docker プロバイダーはホスト上の Docker にアクセスする必要があるため、カスタムの Cluster 用の設定が必要です:

  ```bash
  cat > kind-cluster-with-extramounts.yaml <<EOF
kind: Cluster
apiVersion: kind.sigs.k8s.io/v1alpha3
nodes:
  - role: control-plane
    extraMounts:
      - hostPath: /var/run/docker.sock
        containerPath: /var/run/docker.sock
EOF
  kind create cluster --config ./kind-cluster-with-extramounts.yaml --name clusterapi
  kubectl cluster-info --context kind-clusterapi
  ```
{{#/tab }}
{{#/tabs }}


2. **既存のマネジメント クラスター**

プロダクション向けのユースケースでは、適切なバックアップとディザスタ リカバリー ポリシーおよび手順を備えた「実際の」 Kubernetes クラスターを使う必要があります。

```bash
export KUBECONFIG=<...>
```

3. **ピボット**

ピボットは、最初の kind Cluster を取得して新しいワークロード クラスターを作成し、 Cluster API の CRD を移行してワークロード クラスターをマネジメント クラスターに変換するプロセスです。


## インストール

[kubectl] を使って、 [management cluster] にコンポーネントを作成します:

#### Cluster API のインストール

```bash
kubectl create -f {{#releaselink gomodule:"sigs.k8s.io/cluster-api" asset:"cluster-api-components.yaml" version:"0.2.x"}}
```

#### ブートストラップ プロバイダーのインストール

{{#tabs name:"tab-installation-bootstrap" tabs:"Kubeadm"}}
{{#tab Kubeadm}}

最新のコンポーネント ファイルについては [Kubeadm provider releases](https://github.com/kubernetes-sigs/cluster-api-bootstrap-provider-kubeadm/releases) を確認してください。

```bash
kubectl create -f {{#releaselink gomodule:"sigs.k8s.io/cluster-api-bootstrap-provider-kubeadm" asset:"bootstrap-components.yaml" version:"0.1.x"}}
```

{{#/tab }}
{{#/tabs }}


#### インフラストラクチャー プロバイダーのインストール

{{#tabs name:"tab-installation-infrastructure" tabs:"AWS,Azure,Docker,GCP,vSphere,OpenStack"}}
{{#tab AWS}}

<aside class="note warning">

<h1>必要なアクション</h1>

クレデンシャル管理、IAM、および AWS の要件に関する詳細な情報は、 [AWS Provider Prerequisites](https://github.com/kubernetes-sigs/cluster-api-provider-aws/blob/master/docs/prerequisites.md) ドキュメントを参照してください。

</aside>

#### clusterawsadm のインストール

[AWS provider releases] から最新の `clusterawsadm` バイナリをダウンロードしパスに配置します。

##### コンポーネントの作成

最新のコンポーネント ファイルについては [AWS provider releases] を確認してください。

```bash
# clusterawsadm を使い、 Base64 でエンコードされたクレデンシャルを作成します。
# このコマンドは環境変数とエンコーダーを使い、エンコードされたクレデンシャルを
# Kubernetes Secret となる値に保存します。
export AWS_B64ENCODED_CREDENTIALS=$(clusterawsadm alpha bootstrap encode-aws-credentials)

The `envsubst` application is provided by the `gettext` package on Linux and via Brew on MacOS.

# コンポーネントの作成
curl -L {{#releaselink gomodule:"sigs.k8s.io/cluster-api-provider-aws" asset:"infrastructure-components.yaml" version:"0.4.x"}} \
  | envsubst \
  | kubectl create -f -
```

{{#/tab }}
{{#tab Azure}}

<aside class="note warning">

<h1>必要なアクション</h1>

認可、 AAD 、および Azure の要件に関する詳細な情報は、 [Azure Provider Prerequisites](https://github.com/kubernetes-sigs/cluster-api-provider-azure/blob/master/docs/getting-started.md#prerequisites) ドキュメントを参照してください。

</aside>

最新のコンポーネント ファイルについては [Azure provider releases](https://github.com/kubernetes-sigs/cluster-api-provider-azure/releases) を確認してください。

```bash
# Base64 でエンコードされたクレデンシャルを作成
export AZURE_SUBSCRIPTION_ID_B64="$(echo -n "$AZURE_SUBSCRIPTION_ID" | base64 | tr -d '\n')"
export AZURE_TENANT_ID_B64="$(echo -n "$AZURE_TENANT_ID" | base64 | tr -d '\n')"
export AZURE_CLIENT_ID_B64="$(echo -n "$AZURE_CLIENT_ID" | base64 | tr -d '\n')"
export AZURE_CLIENT_SECRET_B64="$(echo -n "$AZURE_CLIENT_SECRET" | base64 | tr -d '\n')"
```

```bash
curl -L {{#releaselink gomodule:"sigs.k8s.io/cluster-api-provider-azure" asset:"infrastructure-components.yaml" version:"0.3.x"}} \
  | envsubst \
  | kubectl create -f -
```

{{#/tab }}
{{#tab Docker}}

最新のコンポーネント ファイルについては [Docker provider releases](https://github.com/kubernetes-sigs/cluster-api-provider-docker/releases) を確認してください。

```bash
kubectl create -f {{#releaselink gomodule:"sigs.k8s.io/cluster-api-provider-docker" asset:"provider-components.yaml" version:"0.2.x"}}
```

{{#/tab }}
{{#tab GCP}}

<aside class="note warning">

<h1>必要なアクション</h1>

以下の GCP クレデンシャルへのパスを更新します。

</aside>

##### コンポーネントの作成

最新のコンポーネント ファイルについては [GCP provider releases](https://github.com/kubernetes-sigs/cluster-api-provider-gcp/releases) を確認してください。

```bash
# クレデンシャル JSON をもとに、 Base64 でエンコードされたクレデンシャルを作成します。
# このコマンドは環境変数とエンコーダーを使い、エンコードされたクレデンシャルを
# Kubernetes Secret となる値に保存します。
export GCP_B64ENCODED_CREDENTIALS=$( cat /path/to/gcp-credentials.json | base64 | tr -d '\n' )

# コンポーネントの作成
curl -L {{#releaselink gomodule:"sigs.k8s.io/cluster-api-provider-gcp" asset:"infrastructure-components.yaml" version:"0.3.x"}} \
  | envsubst \
  | kubectl create -f -
```

{{#/tab }}
{{#tab vSphere}}

vSphere VM テンプレートには公式の CAPV マシン イメージを使う必要があります。
これを行う方法については [Uploading CAPV Machine Images](https://github.com/kubernetes-sigs/cluster-api-provider-vsphere/blob/master/docs/getting_started.md#uploading-the-capv-machine-image) を参照してください。

```bash
# Kubernetes の Secret として vCenter クレデンシャルをアップロード
$ cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  labels:
    control-plane: controller-manager
  name: capv-system
---
apiVersion: v1
kind: Secret
metadata:
  name: capv-manager-bootstrap-credentials
  namespace: capv-system
type: Opaque
stringData:
  username: "<my vCenter username>"
  password: "<my vCenter password>"
EOF

$ kubectl create -f {{#releaselink gomodule:"sigs.k8s.io/cluster-api-provider-vsphere" asset:"infrastructure-components.yaml" version:"0.5.x"}}
```

最新のコンポーネント ファイルについては [vSphere provider releases](https://github.com/kubernetes-sigs/cluster-api-provider-vsphere/releases) を確認してください。

vSphere の要件、クレデンシャル管理、権限に関する詳細な情報は、 [getting started guide](https://github.com/kubernetes-sigs/cluster-api-provider-vsphere/blob/master/docs/getting_started.md) ドキュメントを参照してください。

{{#/tab }}
{{#tab OpenStack}}

最新のコンポーネント ファイルについては [OpenStack provider releases](https://github.com/kubernetes-sigs/cluster-api-provider-openstack/releases) を確認してください。

前提条件などの詳細な情報については、 [getting started guide](https://github.com/kubernetes-sigs/cluster-api-provider-openstack/blob/master/docs/getting-started.md) ドキュメントを参照してください。

```bash
kubectl create -f {{#releaselink gomodule:"sigs.k8s.io/cluster-api-provider-openstack" asset:"infrastructure-components.yaml" version:"0.2.x"}}
```

{{#/tab }}
{{#/tabs }}


<!-- links -->
[kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[components]: ../reference/glossary.md#provider-components
[kind]: https://sigs.k8s.io/kind
[management cluster]: ../reference/glossary.md#management-cluster
[target cluster]: ../reference/glossary.md#target-cluster
[AWS provider releases]: https://github.com/kubernetes-sigs/cluster-api-provider-aws/releases
