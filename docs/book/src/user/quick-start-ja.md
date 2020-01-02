# クイック スタート

このチュートリアルでは Cluster API を使って 1 つ以上の Kubernetes クラスターを作成する方法の基本を説明します。

{{#include ../tasks/installation-ja.md}}

## 使用方法

Cluster API 、ブートストラップおよびインフラストラクチャーリソースがインストールされたので、シングル ノード クラスターの作成に進みましょう。

このチュートリアルでは、クラスターに `capi-quickstart` という名前をつけます。

{{#tabs name:"tab-usage-cluster-resource" tabs:"AWS,Azure,Docker,GCP,vSphere,OpenStack"}}
{{#tab AWS}}

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: Cluster
metadata:
  name: capi-quickstart
spec:
  clusterNetwork:
    pods:
      cidrBlocks: ["192.168.0.0/16"]
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
    kind: AWSCluster
    name: capi-quickstart
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: AWSCluster
metadata:
  name: capi-quickstart
spec:
  # この値をクラスターをデプロイするリージョンに変更します
  region: us-east-1
  # この値を AWS アカウントに存在する有効な SSH キーペアに変更します
  sshKeyName: default
```
{{#/tab }}
{{#tab Azure}}

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: Cluster
metadata:
  name: capi-quickstart
spec:
  clusterNetwork:
    pods:
      cidrBlocks:
      - 192.168.0.0/16
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
    kind: AzureCluster
    name: capi-quickstart
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: AzureCluster
metadata:
  name: capi-quickstart
spec:
  # この値をクラスターをデプロイするリージョンに変更します
  location: southcentralus
  networkSpec:
    vnet:
      name: capi-quickstart-vnet
  # この値をクラスターをデプロイするリソース グループに変更します
  resourceGroup: capi-quickstart
```
{{#/tab }}
{{#tab Docker}}

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: Cluster
metadata:
  name: capi-quickstart
spec:
  clusterNetwork:
    pods:
      cidrBlocks: ["192.168.0.0/16"]
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
    kind: DockerCluster
    name: capi-quickstart
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: DockerCluster
metadata:
  name: capi-quickstart
```
{{#/tab }}
{{#tab GCP}}
<aside class="note warning">

<h1>必要なアクション</h1>

必要に応じて以下のプロジェクトを固有の値に書き換えリージョンを更新します。

</aside>

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: Cluster
metadata:
  name: capi-quickstart
spec:
  clusterNetwork:
    pods:
      cidrBlocks:
      - 192.168.0.0/16
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
    kind: GCPCluster
    name: capi-quickstart
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: GCPCluster
metadata:
  name: capi-quickstart
spec:
  project: capi-quickstart
  region: us-central1
```
{{#/tab }}
{{#tab vSphere}}

<aside class="note warning">

<h1>必要なアクション</h1>

このクイック スタートでは、あなたの環境に基づいて置き換える必要のある以下の vSphere 環境を想定しています。

| プロパティ       | 値                    | 説明                                                                                                                                                                     |
|----------------|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| vCenter Server | 10.0.0.1                 | vCenter サーバーの IP アドレス、または完全修飾ドメイン名 (FQDN)                                                                                                              |
| Datacenter     | SDDC-Datacenter          | VM がデプロイされるデータセンター                                                                                                                                          |
| Datastore      | DefaultDatastore         | VM で使用するデータストア                                                                                                                                                |
| Resource Pool  | '*/Resources'            | VM が配置されるリソース プール。インベントリ パスにアスタリスク (*) を使用する場合、全体をシングル クオートで囲う必要がある点に注意してください                                            |
| VM Network     | vm-network-1             | VM で使用する VM ネットワーク                                                                                                                                            |
| VM Folder      | vm                       | VM が配置される VM フォルダー                                                                                                                                            |
| VM Template    | ubuntu-1804-kube-v1.16.2 | VM で使用する VM テンプレート                                                                                                                                            |

</aside>

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: Cluster
metadata:
  name: capi-quickstart
spec:
  clusterNetwork:
    pods:
      cidrBlocks: ["192.168.0.0/16"] # CIDR block used by Calico.
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
    kind: VSphereCluster
    name: capi-quickstart
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: VSphereCluster
metadata:
  name: capi-quickstart
spec:
  cloudProviderConfiguration:
    global:
      insecure: true
      secretName: cloud-provider-vsphere-credentials
      secretNamespace: kube-system
    network:
      name: vm-network-1
    providerConfig:
      cloud:
        controllerImage: gcr.io/cloud-provider-vsphere/cpi/release/manager:v1.0.0
      storage:
        attacherImage: quay.io/k8scsi/csi-attacher:v1.1.1
        controllerImage: gcr.io/cloud-provider-vsphere/csi/release/driver:v1.0.1
        livenessProbeImage: quay.io/k8scsi/livenessprobe:v1.1.0
        metadataSyncerImage: gcr.io/cloud-provider-vsphere/csi/release/syncer:v1.0.1
        nodeDriverImage: gcr.io/cloud-provider-vsphere/csi/release/driver:v1.0.1
        provisionerImage: quay.io/k8scsi/csi-provisioner:v1.2.1
        registrarImage: quay.io/k8scsi/csi-node-driver-registrar:v1.1.0
    virtualCenter:
      10.0.0.1:
        datacenters: SDDC-Datacenter
    workspace:
      datacenter: SDDC-Datacenter
      datastore: DefaultDatastore
      folder: vm
      resourcePool: '*/Resources'
      server: 10.0.0.1
  server: 10.0.0.1
```
{{#/tab }}
{{#tab OpenStack}}

<aside class="note warning">

<h1>必要なアクション</h1>

これらの例には、リソースを作成する前に置き換える必要のある環境変数が含まれています。

</aside>

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: Cluster
metadata:
  name: capi-quickstart
spec:
  clusterNetwork:
    services:
      cidrBlocks: ["10.96.0.0/12"]
    pods:
      cidrBlocks: ["192.168.0.0/16"] # CIDR block used by Calico.
    serviceDomain: "cluster.local"
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
    kind: OpenStackCluster
    name: capi-quickstart
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: OpenStackCluster
metadata:
  name: capi-quickstart
spec:
  cloudName: ${OPENSTACK_CLOUD}
  cloudsSecret:
    name: cloud-config
  nodeCidr: ${NODE_CIDR}
  externalNetworkId: ${OPENSTACK_EXTERNAL_NETWORK_ID}
  disablePortSecurity: true
  disableServerTags: true
---
apiVersion: v1
kind: Secret
metadata:
  name: cloud-config
type: Opaque
data:
  # このファイルは通常の OpenStack cloud.yaml 形式である必要があります
  clouds.yaml: ${OPENSTACK_CLOUD_CONFIG_B64ENCODED}
  cacert: ${OPENSTACK_CLOUD_CACERT_B64ENCODED}
```
{{#/tab }}
{{#/tabs }}

クラスター オブジェクトを作成したので、コントロール プレーン マシーン を作成できます。

{{#tabs name:"tab-usage-controlplane-resource" tabs:"AWS,Azure,Docker,GCP,vSphere,OpenStack"}}
{{#tab AWS}}

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: Machine
metadata:
  name: capi-quickstart-controlplane-0
  labels:
    cluster.x-k8s.io/control-plane: "true"
    cluster.x-k8s.io/cluster-name: "capi-quickstart"
spec:
  version: v1.15.3
  bootstrap:
    configRef:
      apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
      kind: KubeadmConfig
      name: capi-quickstart-controlplane-0
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
    kind: AWSMachine
    name: capi-quickstart-controlplane-0
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: AWSMachine
metadata:
  name: capi-quickstart-controlplane-0
spec:
  instanceType: t3.large
  # この IAM プロファイルは前提条件の一部です
  iamInstanceProfile: "control-plane.cluster-api-provider-aws.sigs.k8s.io"
  # この値を AWS アカウントに存在する有効な SSH キーペアに変更します
  sshKeyName: default
---
apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
kind: KubeadmConfig
metadata:
  name: capi-quickstart-controlplane-0
spec:
  # これらの値に関するより詳細な情報は
  # Kubeadm ブートストラップ プロバイダーのドキュメントを参照してください
  initConfiguration:
    nodeRegistration:
      name: '{{ ds.meta_data.local_hostname }}'
      kubeletExtraArgs:
        cloud-provider: aws
  clusterConfiguration:
    apiServer:
      extraArgs:
        cloud-provider: aws
    controllerManager:
      extraArgs:
        cloud-provider: aws
```
{{#/tab }}
{{#tab Azure}}

<aside class="note warning">

<h1>必要なアクション</h1>

これらの例には、リソースを作成する前に置き換える必要のある環境変数が含まれています。
以下の YAML の内容を `controlplane.yaml` に貼り付けた後、次のコマンドを実行して必要な変数を注入し、リソースを作成できます:

```bash
export AZURE_SUBSCRIPTION_ID_B64="$(echo -n "$AZURE_SUBSCRIPTION_ID" | base64 | tr -d '\n')"
export AZURE_TENANT_ID_B64="$(echo -n "$AZURE_TENANT_ID" | base64 | tr -d '\n')"
export AZURE_CLIENT_ID_B64="$(echo -n "$AZURE_CLIENT_ID" | base64 | tr -d '\n')"
export AZURE_CLIENT_SECRET_B64="$(echo -n "$AZURE_CLIENT_SECRET" | base64 | tr -d '\n')"
SSH_KEY_FILE="capi-quickstart.ssh-key"
ssh-keygen -t rsa -b 2048 -f "${SSH_KEY_FILE}" -N '' 1>/dev/null
export SSH_PUBLIC_KEY=$(cat "${SSH_KEY_FILE}.pub" | base64 | tr -d '\r\n')
```

```bash
cat controlplane.yaml \
  | envsubst \
  | kubectl create -f -
```

</aside>

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: Machine
metadata:
  name: capi-quickstart-controlplane-0
  labels:
    cluster.x-k8s.io/control-plane: "true"
    cluster.x-k8s.io/cluster-name: "capi-quickstart"
spec:
  version: v1.16.1
  bootstrap:
    configRef:
      apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
      kind: KubeadmConfig
      name: capi-quickstart-controlplane-0
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
    kind: AzureMachine
    name: capi-quickstart-controlplane-0
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: AzureMachine
metadata:
  name: capi-quickstart-controlplane-0
spec:
  image:
    offer: capi
    publisher: cncf-upstream
    sku: k8s-1dot16-ubuntu-1804
    version: latest
  location: southcentralus
  osDisk:
    diskSizeGB: 30
    managedDisk:
      storageAccountType: Premium_LRS
    osType: Linux
  sshPublicKey: ${SSH_PUBLIC_KEY}
  vmSize: Standard_B2ms
---
apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
kind: KubeadmConfig
metadata:
  name: capi-quickstart-controlplane-0
spec:
  # これらの値に関するより詳細な情報は
  # Kubeadm ブートストラップ プロバイダーのドキュメントを参照してください
  clusterConfiguration:
    apiServer:
      extraArgs:
        cloud-config: /etc/kubernetes/azure.json
        cloud-provider: azure
      extraVolumes:
      - hostPath: /etc/kubernetes/azure.json
        mountPath: /etc/kubernetes/azure.json
        name: cloud-config
        readOnly: true
      timeoutForControlPlane: 20m
    controllerManager:
      extraArgs:
        allocate-node-cidrs: "false"
        cloud-config: /etc/kubernetes/azure.json
        cloud-provider: azure
      extraVolumes:
      - hostPath: /etc/kubernetes/azure.json
        mountPath: /etc/kubernetes/azure.json
        name: cloud-config
        readOnly: true
  files:
  - content: |
      {
        "cloud": "AzurePublicCloud",
        "tenantId": "${AZURE_TENANT_ID}",
        "subscriptionId": "${AZURE_SUBSCRIPTION_ID}",
        "aadClientId": "${AZURE_CLIENT_ID}",
        "aadClientSecret": "${AZURE_CLIENT_SECRET}",
        "resourceGroup": "capi-quickstart",
        "securityGroupName": "capi-quickstart-controlplane-nsg",
        "location": "${AZURE_LOCATION}",
        "vmType": "standard",
        "vnetName": "capi-quickstart",
        "vnetResourceGroup": "capi-quickstart",
        "subnetName": "capi-quickstart-controlplane-subnet",
        "routeTableName": "capi-quickstart-node-routetable",
        "userAssignedID": "capi-quickstart",
        "loadBalancerSku": "standard",
        "maximumLoadBalancerRuleCount": 250,
        "useManagedIdentityExtension": false,
        "useInstanceMetadata": true
      }
    owner: root:root
    path: /etc/kubernetes/azure.json
    permissions: "0644"
  initConfiguration:
    nodeRegistration:
      kubeletExtraArgs:
        cloud-config: /etc/kubernetes/azure.json
        cloud-provider: azure
      name: '{{ ds.meta_data.local_hostname }}'
```
{{#/tab }}
{{#tab Docker}}

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: Machine
metadata:
  name: capi-quickstart-controlplane-0
  labels:
    cluster.x-k8s.io/control-plane: "true"
    cluster.x-k8s.io/cluster-name: "capi-quickstart"
spec:
  version: v1.15.3
  bootstrap:
    configRef:
      apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
      kind: KubeadmConfig
      name: capi-quickstart-controlplane-0
  infrastructureRef:
    kind: DockerMachine
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
    name: capi-quickstart-controlplane-0
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: DockerMachine
metadata:
  name: capi-quickstart-controlplane-0
---
apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
kind: KubeadmConfig
metadata:
  name: capi-quickstart-controlplane-0
spec:
  initConfiguration:
    nodeRegistration:
      kubeletExtraArgs:
        # リソースが完全に使い果たされる前にバッファを提供するために、デフォルトのしきい値は
        # 高くなっていますが、より多くのリソースを必要とします。
        # これらの低いしきい値によって、より少ないリソースで実行できます。
        # テストまたは開発のみに適しています。
        eviction-hard: nodefs.available<0%,nodefs.inodesFree<0%,imagefs.available<0%
  clusterConfiguration:
    controllerManager:
      extraArgs:
        # クラウド プロバイダーなしでダイナミック ストレージ プロビジョニングを有効にします。
        # テストまたは開発のみに適しています。
        enable-hostpath-provisioner: "true"
```
{{#/tab }}
{{#tab GCP}}
<aside class="note warning">

<h1>必要なアクション</h1>

以下のインスタンス タイプとゾーンを自由に更新してください。

</aside>

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: Machine
metadata:
  name: capi-quickstart-controlplane-0
  labels:
    cluster.x-k8s.io/control-plane: "true"
    cluster.x-k8s.io/cluster-name: "capi-quickstart"
spec:
  version: v1.15.3
  bootstrap:
    configRef:
      apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
      kind: KubeadmConfig
      name: capi-quickstart-controlplane-0
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
    kind: GCPMachine
    name: capi-quickstart-controlplane-0
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: GCPMachine
metadata:
  name: capi-quickstart-controlplane-0
spec:
  instanceType: n1-standard-2
  zone: us-central1-a
---
apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
kind: KubeadmConfig
metadata:
  name: capi-quickstart-controlplane-0
spec:
  # これらの値に関するより詳細な情報は
  # Kubeadm ブートストラップ プロバイダーのドキュメントを参照してください
  initConfiguration:
    nodeRegistration:
      name: '{{ ds.meta_data.local_hostname }}'
      kubeletExtraArgs:
        cloud-provider: gce
  clusterConfiguration:
    apiServer:
      extraArgs:
        cloud-provider: gce
    controllerManager:
      extraArgs:
        cloud-provider: gce
```
{{#/tab }}
{{#tab vSphere}}

<aside class="note warning">

<h1>必要なアクション</h1>

このクイック スタートでは、あなたの環境に基づいて置き換える必要のある以下の vSphere 環境を想定しています。

| プロパティ       | 値                    | 説明                                                                                                                                                                     |
|----------------|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| vCenter Server | 10.0.0.1                 | vCenter サーバーの IP アドレス、または完全修飾ドメイン名 (FQDN)                                                                                                              |
| Datacenter     | SDDC-Datacenter          | VM がデプロイされるデータセンター                                                                                                                                          |
| Datastore      | DefaultDatastore         | VM で使用するデータストア                                                                                                                                                |
| Resource Pool  | '*/Resources'            | VM が配置されるリソース プール。インベントリ パスにアスタリスク (*) を使用する場合、全体をシングル クオートで囲う必要がある点に注意してください                                            |
| VM Network     | vm-network-1             | VM で使用する VM ネットワーク                                                                                                                                            |
| VM Folder      | vm                       | VM が配置される VM フォルダー                                                                                                                                            |
| VM Template    | ubuntu-1804-kube-v1.16.2 | VM で使用する VM テンプレート                                                                                                                                            |
</aside>

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: Machine
metadata:
  name: capi-quickstart-controlplane-0
  labels:
    cluster.x-k8s.io/control-plane: "true"
    cluster.x-k8s.io/cluster-name: "capi-quickstart"
spec:
  version: v1.16.2
  bootstrap:
    configRef:
      apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
      kind: KubeadmConfig
      name: capi-quickstart-controlplane-0
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
    kind: VSphereMachine
    name: capi-quickstart-controlplane-0
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: VSphereMachine
metadata:
  labels:
    cluster.x-k8s.io/cluster-name: capi-quickstart
    cluster.x-k8s.io/control-plane: "true"
  name: capi-quickstart-controlplane-0
  namespace: default
spec:
  datacenter: SDDC-Datacenter
  diskGiB: 50
  memoryMiB: 2048
  network:
    devices:
    - dhcp4: true
      dhcp6: false
      networkName: vm-network-1
  numCPUs: 2
  template: ubuntu-1804-kube-v1.16.2
---
apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
kind: KubeadmConfig
metadata:
  name: capi-quickstart-controlplane-0
  namespace: default
spec:
  clusterConfiguration:
    apiServer:
      extraArgs:
        cloud-provider: external
    controllerManager:
      extraArgs:
        cloud-provider: external
    imageRepository: k8s.gcr.io
  initConfiguration:
    nodeRegistration:
      criSocket: /var/run/containerd/containerd.sock
      kubeletExtraArgs:
        cloud-provider: external
      name: '{{ ds.meta_data.local_hostname }}'
  preKubeadmCommands:
  - hostname "{{ ds.meta_data.local_hostname }}"
  - echo "::1         ipv6-localhost ipv6-loopback" >/etc/hosts
  - echo "127.0.0.1   localhost {{ ds.meta_data.local_hostname }}" >>/etc/hosts
  - echo "{{ ds.meta_data.local_hostname }}" >/etc/hostname
```
{{#/tab }}
{{#tab OpenStack}}

<aside class="note warning">

<h1>必要なアクション</h1>

これらの例には、リソースを作成する前に置き換える必要のある環境変数が含まれています。

</aside>

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: Machine
metadata:
  name: capi-quickstart-controlplane-0
  labels:
    cluster.x-k8s.io/control-plane: "true"
    cluster.x-k8s.io/cluster-name: "capi-quickstart"
spec:
  version: v1.15.3
  bootstrap:
    configRef:
      apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
      kind: KubeadmConfig
      name: capi-quickstart-controlplane-0
  infrastructureRef:
    apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
    kind: OpenStackMachine
    name: capi-quickstart-controlplane-0
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: OpenStackMachine
metadata:
  name: capi-quickstart-controlplane-0
spec:
  flavor: m1.medium
  image: ${IMAGE_NAME}
  availabilityZone: nova
  floatingIP: ${FLOATING_IP}
  cloudName: ${OPENSTACK_CLOUD}
  cloudsSecret:
    name: cloud-config
---
apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
kind: KubeadmConfig
metadata:
  name: capi-quickstart-controlplane-0
spec:
  # これらの値に関するより詳細な情報は
  # Kubeadm ブートストラップ プロバイダーのドキュメントを参照してください
  initConfiguration:
    localAPIEndpoint:
      advertiseAddress: '{{ ds.ec2_metadata.local_ipv4 }}'
      bindPort: 6443
    nodeRegistration:
      name: '{{ ds.meta_data.local_hostname }}'
      criSocket: "/var/run/containerd/containerd.sock"
      kubeletExtraArgs:
        cloud-provider: openstack
        cloud-config: /etc/kubernetes/cloud.conf
  clusterConfiguration:
    controlPlaneEndpoint: "${FLOATING_IP}:6443"
    imageRepository: k8s.gcr.io
    apiServer:
      extraArgs:
        cloud-provider: openstack
        cloud-config: /etc/kubernetes/cloud.conf
      extraVolumes:
      - name: cloud
        hostPath: /etc/kubernetes/cloud.conf
        mountPath: /etc/kubernetes/cloud.conf
        readOnly: true
    controllerManager:
      extraArgs:
        cloud-provider: openstack
        cloud-config: /etc/kubernetes/cloud.conf
      extraVolumes:
      - name: cloud
        hostPath: /etc/kubernetes/cloud.conf
        mountPath: /etc/kubernetes/cloud.conf
        readOnly: true
      - name: cacerts
        hostPath: /etc/certs/cacert
        mountPath: /etc/certs/cacert
        readOnly: true
  files:
  - path: /etc/kubernetes/cloud.conf
    owner: root
    permissions: "0600"
    encoding: base64
    # このファイルは OpenStack クラウド プロバイダーの形式である必要があります
    content: |-
      ${OPENSTACK_CLOUD_CONFIG_B64ENCODED}
  - path: /etc/certs/cacert
    owner: root
    permissions: "0600"
    content: |
      ${OPENSTACK_CLOUD_CACERT_B64ENCODED}
  users:
  - name: capo
    sudo: "ALL=(ALL) NOPASSWD:ALL"
    sshAuthorizedKeys:
    - "${SSH_AUTHORIZED_KEY}"
```
{{#/tab }}
{{#/tabs }}

コントロール プレーンが起動し実行されたら、 [target cluster] の Kubeconfig を取得しましょう:

{{#tabs name:"tab-getting-kubeconfig" tabs:"AWS|Azure|GCP|vSphere|OpenStack, Docker"}}
{{#tab AWS|Azure|GCP|vSphere|OpenStack}}

```bash
kubectl --namespace=default get secret/capi-quickstart-kubeconfig -o json \
  | jq -r .data.value \
  | base64 --decode \
  > ./capi-quickstart.kubeconfig
```

{{#/tab }}
{{#tab Docker}}

```bash
kubectl --namespace=default get secret/capi-quickstart-kubeconfig -o json \
  | jq -r .data.value \
  | base64 --decode \
  > ./capi-quickstart.kubeconfig
```

MacOS で docker-for-mac を使用する場合は、正しい kubeconfig を取得するためにいくつかの追加の手順が必要になります:

```bash
# アクセスできないコンテナ IP ではなく、ロード バランサーの公開ポートを kubeconfig に設定します
sed -i -e "s/server:.*/server: https:\/\/$(docker port capi-quickstart-lb 6443/tcp | sed "s/0.0.0.0/127.0.0.1/")/g" ./capi-quickstart.kubeconfig

# 127.0.0.1 に署名されていないため、 CA を無視します
sed -i -e "s/certificate-authority-data:.*/insecure-skip-tls-verify: true/g" ./capi-quickstart.kubeconfig
```

{{#/tab }}
{{#/tabs }}

CNI ソリューションをデプロイします。ここでは例として Calico を使用します。

{{#tabs name:"tab-usage-addons" tabs:"Calico"}}
{{#tab Calico}}

```bash
kubectl --kubeconfig=./capi-quickstart.kubeconfig \
  apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
```

<aside class="note warning">

<h1>必要なアクション (Docker プロバイダー v0.2.0 のみ)</h1>

[既知の問題](https://github.com/kubernetes-sigs/kind/issues/891) が Calico と Docker プロバイダー v0.2.0 の組み合わせに影響を及ぼします。
Calico をデプロイしたあとに、このパッチを適用して問題を回避します:

```bash
kubectl --kubeconfig=./capi-quickstart.kubeconfig \
  -n kube-system patch daemonset calico-node \
  --type=strategic --patch='
spec:
  template:
    spec:
      containers:
      - name: calico-node
        env:
        - name: FELIX_IGNORELOOSERPF
          value: "true"
'
```
</aside>
{{#/tab }}
{{#/tabs }}

しばらくすると、コントロール プレーンが起動し、状態が `Ready` になっているはずです。
`kubectl get nodes` を使ってステータスを確認しましょう:

```bash
kubectl --kubeconfig=./capi-quickstart.kubeconfig get nodes
```

仕上げに、シングル ノードの _MachineDeployment_ を作成します。

{{#tabs name:"tab-usage-machinedeployment" tabs:"AWS,Azure,Docker,GCP,vSphere,OpenStack"}}
{{#tab AWS}}

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: MachineDeployment
metadata:
  name: capi-quickstart-worker
  labels:
    cluster.x-k8s.io/cluster-name: capi-quickstart
    # これより先のラベルは例示目的のものです。
    # より意味のあるものを追加したり変更したりしてください。
    # これらの値を spec.selector.matchLabels および spec.template.metadata.labels と一致させます。
    nodepool: nodepool-0
spec:
  replicas: 1
  selector:
    matchLabels:
      cluster.x-k8s.io/cluster-name: capi-quickstart
      nodepool: nodepool-0
  template:
    metadata:
      labels:
        cluster.x-k8s.io/cluster-name: capi-quickstart
        nodepool: nodepool-0
    spec:
      version: v1.15.3
      bootstrap:
        configRef:
          name: capi-quickstart-worker
          apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
          kind: KubeadmConfigTemplate
      infrastructureRef:
        name: capi-quickstart-worker
        apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
        kind: AWSMachineTemplate
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: AWSMachineTemplate
metadata:
  name: capi-quickstart-worker
spec:
  template:
    spec:
      instanceType: t3.large
      # この IAM プロファイルは前提条件の一部です
      iamInstanceProfile: "nodes.cluster-api-provider-aws.sigs.k8s.io"
      # この値を AWS アカウントに存在する有効な SSH キーペアに変更します
      sshKeyName: default
---
apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
kind: KubeadmConfigTemplate
metadata:
  name: capi-quickstart-worker
spec:
  template:
    spec:
      # これらの値に関するより詳細な情報は
      # Kubeadm ブートストラップ プロバイダーのドキュメントを参照してください
      joinConfiguration:
        nodeRegistration:
          name: '{{ ds.meta_data.local_hostname }}'
          kubeletExtraArgs:
            cloud-provider: aws
```

{{#/tab }}
{{#tab Azure}}

<aside class="note warning">

<h1>必要なアクション</h1>

これらの例には、リソースを作成する前に置き換える必要のある環境変数が含まれています。
以下の YAML の内容を `machinedeployment.yaml` に貼り付けた後、次のコマンドを実行して必要な変数を注入し、リソースを作成できます:

```bash
export AZURE_SUBSCRIPTION_ID_B64="$(echo -n "$AZURE_SUBSCRIPTION_ID" | base64 | tr -d '\n')"
export AZURE_TENANT_ID_B64="$(echo -n "$AZURE_TENANT_ID" | base64 | tr -d '\n')"
export AZURE_CLIENT_ID_B64="$(echo -n "$AZURE_CLIENT_ID" | base64 | tr -d '\n')"
export AZURE_CLIENT_SECRET_B64="$(echo -n "$AZURE_CLIENT_SECRET" | base64 | tr -d '\n')"
SSH_KEY_FILE="capi-quickstart.ssh-key"
ssh-keygen -t rsa -b 2048 -f "${SSH_KEY_FILE}" -N '' 1>/dev/null
export SSH_PUBLIC_KEY=$(cat "${SSH_KEY_FILE}.pub" | base64 | tr -d '\r\n')
```

```bash
cat machinedeployment.yaml \
  | envsubst \
  | kubectl create -f -
```

</aside>

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: MachineDeployment
metadata:
  name: capi-quickstart-node
  labels:
    cluster.x-k8s.io/cluster-name: capi-quickstart
    # これより先のラベルは例示目的のものです。
    # より意味のあるものを追加したり変更したりしてください。
    # これらの値を spec.selector.matchLabels および spec.template.metadata.labels と一致させます。
    nodepool: nodepool-0
spec:
  replicas: 1
  selector:
    matchLabels:
      cluster.x-k8s.io/cluster-name: capi-quickstart
      nodepool: nodepool-0
  template:
    metadata:
      labels:
        cluster.x-k8s.io/cluster-name: capi-quickstart
        nodepool: nodepool-0
    spec:
      version: v1.16.1
      bootstrap:
        configRef:
          name: capi-quickstart-node
          apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
          kind: KubeadmConfigTemplate
      infrastructureRef:
        name: capi-quickstart-node
        apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
        kind: AzureMachineTemplate
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: AzureMachineTemplate
metadata:
  name: capi-quickstart-node
spec:
  template:
    spec:
      location: southcentralus
      vmSize: Standard_B2ms
      image:
        publisher: "cncf-upstream"
        offer: "capi"
        sku: "k8s-1dot16-ubuntu-1804"
        version: "latest"
      osDisk:
        osType: "Linux"
        diskSizeGB: 30
        managedDisk:
          storageAccountType: "Premium_LRS"
      sshPublicKey: ${SSH_PUBLIC_KEY}
---
apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
kind: KubeadmConfigTemplate
metadata:
  name: capi-quickstart-node
spec:
  template:
    spec:
      joinConfiguration:
        nodeRegistration:
          name: '{{ ds.meta_data.local_hostname }}'
          kubeletExtraArgs:
            cloud-provider: azure
            cloud-config: /etc/kubernetes/azure.json
      files:
      - path: /etc/kubernetes/azure.json
        owner: "root:root"
        permissions: "0644"
        content: |
          {
            "cloud": "AzurePublicCloud",
            "tenantId": "${AZURE_TENANT_ID}",
            "subscriptionId": "${AZURE_SUBSCRIPTION_ID}",
            "aadClientId": "${AZURE_CLIENT_ID}",
            "aadClientSecret": "${AZURE_CLIENT_SECRET}",
            "resourceGroup": "capi-quickstart",
            "securityGroupName": "capi-quickstart-controlplane-nsg",
            "location": "${AZURE_LOCATION}",
            "vmType": "standard",
            "vnetName": "capi-quickstart",
            "vnetResourceGroup": "capi-quickstart",
            "subnetName": "capi-quickstart-controlplane-subnet",
            "routeTableName": "capi-quickstart-node-routetable",
            "userAssignedID": "capi-quickstart",
            "loadBalancerSku": "standard",
            "maximumLoadBalancerRuleCount": 250,
            "useManagedIdentityExtension": false,
            "useInstanceMetadata": true
          }
```

{{#/tab }}
{{#tab Docker}}

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: MachineDeployment
metadata:
  name: capi-quickstart-worker
  labels:
    cluster.x-k8s.io/cluster-name: capi-quickstart
    # これより先のラベルは例示目的のものです。
    # より意味のあるものを追加したり変更したりしてください。
    # これらの値を spec.selector.matchLabels および spec.template.metadata.labels と一致させます。
    nodepool: nodepool-0
spec:
  replicas: 1
  selector:
    matchLabels:
      cluster.x-k8s.io/cluster-name: capi-quickstart
      nodepool: nodepool-0
  template:
    metadata:
      labels:
        cluster.x-k8s.io/cluster-name: capi-quickstart
        nodepool: nodepool-0
    spec:
      version: v1.15.3
      bootstrap:
        configRef:
          name: capi-quickstart-worker
          apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
          kind: KubeadmConfigTemplate
      infrastructureRef:
        name: capi-quickstart-worker
        apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
        kind: DockerMachineTemplate
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: DockerMachineTemplate
metadata:
  name: capi-quickstart-worker
spec:
  template:
    spec: {}
---
apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
kind: KubeadmConfigTemplate
metadata:
  name: capi-quickstart-worker
spec:
  template:
    spec:
      # これらの値に関するより詳細な情報は
      # Kubeadm ブートストラップ プロバイダーのドキュメントを参照してください
      joinConfiguration:
        nodeRegistration:
          kubeletExtraArgs:
            eviction-hard: nodefs.available<0%,nodefs.inodesFree<0%,imagefs.available<0%
      clusterConfiguration:
        controllerManager:
          extraArgs:
            enable-hostpath-provisioner: "true"
```
{{#/tab }}
{{#tab GCP}}

<aside class="note warning">

<h1>必要なアクション</h1>

以下のインスタンス タイプとゾーンを自由に更新してください。

</aside>

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: MachineDeployment
metadata:
  name: capi-quickstart-worker
  labels:
    cluster.x-k8s.io/cluster-name: capi-quickstart
    # これより先のラベルは例示目的のものです。
    # より意味のあるものを追加したり変更したりしてください。
    # これらの値を spec.selector.matchLabels および spec.template.metadata.labels と一致させます。
    nodepool: nodepool-0
spec:
  replicas: 1
  selector:
    matchLabels:
      cluster.x-k8s.io/cluster-name: capi-quickstart
      nodepool: nodepool-0
  template:
    metadata:
      labels:
        cluster.x-k8s.io/cluster-name: capi-quickstart
        nodepool: nodepool-0
    spec:
      version: v1.15.3
      bootstrap:
        configRef:
          name: capi-quickstart-worker
          apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
          kind: KubeadmConfigTemplate
      infrastructureRef:
        name: capi-quickstart-worker
        apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
        kind: GCPMachineTemplate
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: GCPMachineTemplate
metadata:
  name: capi-quickstart-worker
spec:
  template:
    spec:
      instanceType: n1-standard-2
      zone: us-central1-a
---
apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
kind: KubeadmConfigTemplate
metadata:
  name: capi-quickstart-worker
spec:
  template:
    spec:
      # これらの値に関するより詳細な情報は
      # Kubeadm ブートストラップ プロバイダーのドキュメントを参照してください
     joinConfiguration:
        nodeRegistration:
          name: '{{ ds.meta_data.local_hostname }}'
          kubeletExtraArgs:
            cloud-provider: gce
```

{{#/tab }}
{{#tab vSphere}}

<aside class="note warning">

<h1>必要なアクション</h1>

このクイック スタートでは、あなたの環境に基づいて置き換える必要のある以下の vSphere 環境を想定しています。

| プロパティ       | 値                    | 説明                                                                                                                                                                     |
|----------------|--------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| vCenter Server | 10.0.0.1                 | vCenter サーバーの IP アドレス、または完全修飾ドメイン名 (FQDN)                                                                                                              |
| Datacenter     | SDDC-Datacenter          | VM がデプロイされるデータセンター                                                                                                                                          |
| Datastore      | DefaultDatastore         | VM で使用するデータストア                                                                                                                                                |
| Resource Pool  | '*/Resources'            | VM が配置されるリソース プール。インベントリ パスにアスタリスク (*) を使用する場合、全体をシングル クオートで囲う必要がある点に注意してください                                            |
| VM Network     | vm-network-1             | VM で使用する VM ネットワーク                                                                                                                                            |
| VM Folder      | vm                       | VM が配置される VM フォルダー                                                                                                                                            |
| VM Template    | ubuntu-1804-kube-v1.16.2 | VM で使用する VM テンプレート                                                                                                                                            |

</aside>

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: MachineDeployment
metadata:
  name: capi-quickstart-worker
  labels:
    cluster.x-k8s.io/cluster-name: capi-quickstart
    # これより先のラベルは例示目的のものです。
    # より意味のあるものを追加したり変更したりしてください。
    # これらの値を spec.selector.matchLabels および spec.template.metadata.labels と一致させます。
    nodepool: nodepool-0
spec:
  replicas: 1
  selector:
    matchLabels:
      cluster.x-k8s.io/cluster-name: capi-quickstart
      nodepool: nodepool-0
  template:
    metadata:
      labels:
        cluster.x-k8s.io/cluster-name: capi-quickstart
        nodepool: nodepool-0
    spec:
      version: v1.16.2
      bootstrap:
        configRef:
          apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
          kind: KubeadmConfigTemplate
          name: capi-quickstart-worker
      infrastructureRef:
        apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
        kind: VSphereMachineTemplate
        name: capi-quickstart-worker
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: VSphereMachineTemplate
metadata:
  name: capi-quickstart-md-0
  namespace: default
spec:
  template:
    spec:
      datacenter: SDDC-Datacenter
      diskGiB: 50
      memoryMiB: 2048
      network:
        devices:
        - dhcp4: true
          dhcp6: false
          networkName: vm-network-1
      numCPUs: 2
      template: ubuntu-1804-kube-v1.16.2
---
apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
kind: KubeadmConfigTemplate
metadata:
  name: capi-quickstart-md-0
  namespace: default
spec:
  template:
    spec:
      joinConfiguration:
        nodeRegistration:
          criSocket: /var/run/containerd/containerd.sock
          kubeletExtraArgs:
            cloud-provider: external
          name: '{{ ds.meta_data.local_hostname }}'
      preKubeadmCommands:
      - hostname "{{ ds.meta_data.local_hostname }}"
      - echo "::1         ipv6-localhost ipv6-loopback" >/etc/hosts
      - echo "127.0.0.1   localhost {{ ds.meta_data.local_hostname }}" >>/etc/hosts
      - echo "{{ ds.meta_data.local_hostname }}" >/etc/hostname
```

{{#/tab }}
{{#tab OpenStack}}

<aside class="note warning">

<h1>必要なアクション</h1>

これらの例には、リソースを作成する前に置き換える必要のある環境変数が含まれています。

</aside>

```yaml
apiVersion: cluster.x-k8s.io/v1alpha3
kind: MachineDeployment
metadata:
  name: capi-quickstart-worker
  labels:
    cluster.x-k8s.io/cluster-name: capi-quickstart
    # これより先のラベルは例示目的のものです。
    # より意味のあるものを追加したり変更したりしてください。
    # これらの値を spec.selector.matchLabels および spec.template.metadata.labels と一致させます。
    nodepool: nodepool-0
spec:
  replicas: 1
  selector:
    matchLabels:
      cluster.x-k8s.io/cluster-name: capi-quickstart
      nodepool: nodepool-0
  template:
    metadata:
      labels:
        cluster.x-k8s.io/cluster-name: capi-quickstart
        nodepool: nodepool-0
    spec:
      version: v1.15.3
      bootstrap:
        configRef:
          apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
          kind: KubeadmConfigTemplate
          name: capi-quickstart-worker
      infrastructureRef:
        apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
        kind: OpenStackMachineTemplate
        name: capi-quickstart-worker
---
apiVersion: infrastructure.cluster.x-k8s.io/v1alpha3
kind: OpenStackMachineTemplate
metadata:
  name: capi-quickstart-worker
spec:
  template:
    spec:
      availabilityZone: nova
      cloudName: ${OPENSTACK_CLOUD}
      cloudsSecret:
        name: cloud-config
      flavor: m1.medium
      image: ${IMAGE_NAME}
---
apiVersion: bootstrap.cluster.x-k8s.io/v1alpha3
kind: KubeadmConfigTemplate
metadata:
  name: capi-quickstart-worker
spec:
  template:
    spec:
      # これらの値に関するより詳細な情報は
      # Kubeadm ブートストラップ プロバイダーのドキュメントを参照してください
      joinConfiguration:
        nodeRegistration:
          name: '{{ local_hostname }}'
          criSocket: "/var/run/containerd/containerd.sock"
          kubeletExtraArgs:
            cloud-config: /etc/kubernetes/cloud.conf
            cloud-provider: openstack
      files:
      - path: /etc/kubernetes/cloud.conf
        owner: root
        permissions: "0600"
        encoding: base64
        # このファイルは OpenStack クラウド プロバイダーの形式である必要があります
        content: |-
          ${OPENSTACK_CLOUD_CONFIG_B64ENCODED}
      - path: /etc/certs/cacert
        owner: root
        permissions: "0600"
        content: |
          ${OPENSTACK_CLOUD_CACERT_B64ENCODED}
      users:
      - name: capo
        sudo: "ALL=(ALL) NOPASSWD:ALL"
        sshAuthorizedKeys:
        - "${SSH_AUTHORIZED_KEY}"
```

{{#/tab }}
{{#/tabs }}

<!-- links -->
[target cluster]: ../reference/glossary.md#target-cluster
