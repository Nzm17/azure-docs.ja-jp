---
title: Azure VMware Solution のデプロイの計画
description: この記事では、Azure VMware Solution のデプロイ ワークフローの概要について説明します。  最終的な結果として、仮想マシン (VM) の作成と移行に向けて環境の準備が整います。
ms.topic: tutorial
ms.date: 10/16/2020
ms.openlocfilehash: 2cc4d40fd8088a632e0c24e3c4b770ebdc9de2e8
ms.sourcegitcommit: 67b44a02af0c8d615b35ec5e57a29d21419d7668
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/06/2021
ms.locfileid: "97912735"
---
# <a name="planning-the-azure-vmware-solution-deployment"></a>Azure VMware Solution のデプロイの計画

この記事では、デプロイ中に使用されるデータを特定して収集するための計画プロセスについて説明します。 デプロイを計画する際は、収集した情報をデプロイ中に参照しやすいよう必ず文書化してください。

このクイック スタートのプロセスにより、仮想マシン (VM) の作成と移行のための運用準備ができた環境が得られます。 

>[!IMPORTANT]
>Azure VMware Solution リソースを作成する前に、「[Azure VMware Solution リソースを有効にする方法](enable-azure-vmware-solution.md)」に従い、サポート チケットを提出してホストの割り当てを依頼してください。 サポート チームがリクエストを受け取った後、リクエストが確認され、ホストが割り当てられるまでに、最大 5 営業日かかります。 Azure VMware Solution の既存のプライベート クラウドがあり、さらに多くのホストを割り当てる必要がある場合は、同じプロセスを実行します。 


## <a name="subscription"></a>サブスクリプション

Azure VMware Solution のデプロイに使用する予定のサブスクリプションを特定します。  新しいサブスクリプションを作成することも、既存のものを再利用することもできます。

>[!NOTE]
>サブスクリプションは、マイクロソフト エンタープライズ契約またはクラウド ソリューション プロバイダー Azure プランに関連付けられている必要があります。 詳細については、「[Azure VMware Solution リソースを有効にする方法](enable-azure-vmware-solution.md)」を参照してください。

## <a name="resource-group"></a>Resource group

Azure VMware Solution のために使用するリソース グループを特定します。  一般に、リソース グループは Azure VMware Solution 専用に作成されますが、既存のリソース グループを使用することもできます。

## <a name="region"></a>リージョン

Azure VMware Solution のデプロイ先リージョンを特定します。  詳細については、[リージョン別の利用可能な製品に関する Azure のガイド](https://azure.microsoft.com/en-us/global-infrastructure/services/?products=azure-vmware)を参照してください。

## <a name="resource-name"></a>リソース名

デプロイ時に使用するリソース名を定義します。  リソース名はわかりやすく説明的な名前であり、その中で Azure VMware Solution プライベート クラウドにタイトルを付けます。

>[!IMPORTANT]
>名前は 40 文字を超えてはいけません。 名前がこの制限を超えていると、プライベート クラウドで使用するパブリック IP アドレスを作成できません。 

## <a name="size-hosts"></a>サイズ ホスト

Azure VMware Solution のデプロイ時に使用するサイズ ホストを特定します。  完全な一覧については、[Azure VMware Solution プライベート クラウドとクラスター](concepts-private-clouds-clusters.md#hosts)に関するドキュメントを参照してください。

## <a name="number-of-hosts"></a>ホストの数

Azure VMware Solution プライベート クラウドにデプロイするホストの数を定義します。  最小ホスト数は 3 で、最大はクラスターあたり 16 個です。  詳細については、[Azure VMware Solution プライベート クラウドとクラスター](concepts-private-clouds-clusters.md#clusters)に関するドキュメントを参照してください。

当初のデプロイ数を超える数にする必要がある場合は、後でいつでもクラスターを拡張できます。

## <a name="vcenter-admin-password"></a>vCenter 管理者パスワード
vCenter 管理者パスワードを定義します。  デプロイ時に vCenter 管理者パスワードを作成します。 パスワードは、vCenter ビルド時の cloudadmin@vsphere.local 管理者アカウントに対するものです。 これを使用して vCenter にサインインします。

## <a name="nsx-t-admin-password"></a>NSX-T 管理者パスワード
NSX-T 管理者パスワードを定義します。  デプロイ時に NSX-T 管理者パスワードを作成します。 パスワードは、NSX のビルド時に、NSX アカウントの管理者ユーザーに割り当てられます。 これを使用して NSX-T Manager にログインします。

## <a name="ip-address-segment"></a>IP アドレス セグメント

デプロイを計画する際の最初の手順は、IP のセグメント化を計画することです。  Azure VMware Solution は、管理者が提供する /22 ネットワークを取り込みます。 次に、それを小さなセグメントに分割してから、それらの IP セグメントを vCenter、VMware HCX、NSX-T、および vMotion のために使用します。

Azure VMware Solution は、内部の ExpressRoute 回線を介して Microsoft Azure Virtual Network に接続します。 ほとんどの場合、ExpressRoute Global Reach 経由でデータ センターに接続します。 

Azure VMware Solution、お使いの既存の Azure 環境、およびオンプレミス環境はすべて、ルートを (一般的には) やり取りします。 それに該当する場合、この手順で定義する/22 CIDR ネットワーク アドレス ブロックは、オンプレミスまたは Azure で利用中のどのブロックとも重複してはいけません。

**例:** 10.0.0.0/22

詳細については、「[ネットワーク計画のチェックリスト](tutorial-network-checklist.md#routing-and-subnet-considerations)」を参照してください。

:::image type="content" source="media/pre-deployment/management-vmotion-vsan-network-ip-diagram.png" alt-text="特定する - IP アドレス セグメント" border="false":::  

## <a name="ip-address-segment-for-virtual-machine-workloads"></a>仮想マシンのワークロードの IP アドレス セグメント

プライベート クラウド内に最初のネットワーク (NSX セグメント) を作成するための IP セグメントを特定します。  言い換えると、Azure VMware Solution に VM をデプロイできるように、Azure VMware Solution 上にネットワーク セグメントを作成する必要があります。   

L2 ネットワークの拡張だけを計画している場合でも、環境を検証するネットワーク セグメントを作成します。

作成されるどの IP セグメントも、Azure とオンプレミスのフットプリント全体で一意である必要があることに注意してください。  

**例:** 10.0.4.0/24

:::image type="content" source="media/pre-deployment/nsx-segment-diagram.png" alt-text="特定する - 仮想マシンのワークロードの IP アドレス セグメント" border="false":::     

## <a name="optional-extend-networks"></a>(省略可能) ネットワークを拡張する

ネットワーク セグメントは、オンプレミスから Azure VMware Solution まで拡張できます。実施する場合は、ここでそれらのネットワークを特定します。  

次の点に留意します。

- オンプレミスからネットワークを拡張する予定の場合、これらのネットワークはオンプレミスの VMware 環境内の [vSphere Distributed Switch (vDS)](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-B15C6A13-797E-4BCB-B9D9-5CBC5A60C3A6.html) に接続する必要があります。  
- 拡張したいネットワークが [vSphere Standard Switch](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.networking.doc/GUID-350344DE-483A-42ED-B0E2-C811EE927D59.html) 上で機能している場合、それらは拡張できません。

## <a name="azure-virtual-network-to-attach-azure-vmware-solution"></a>Azure VMware Solution を接続するための Azure Virtual Network

Azure VMware Solution プライベート クラウドにアクセスするには、Azure VMware Solution に付随する ExpressRoute 回線を、Azure Virtual Network に接続する必要があります。  デプロイ時には、新しい仮想ネットワークを定義すること、または既存のものを選択することが可能です。

Azure VMware Solution からの ExpressRoute 回線は、この手順で定義する Azure Virtual Network の ExpressRoute ゲートウェイに接続します。  

>[!IMPORTANT]
>仮想ネットワークあたり 4 つの ExpressRoute 回路という上限を超えない限り、既存の ExpressRoute ゲートウェイを使用して Azure VMware Solution に接続できます。  ただし、オンプレミスから ExpressRoute 経由で Azure VMware Solution にアクセスするには、ExpressRoute Global Reach が必要です。これは、ExpressRoute ゲートウェイでは、それに接続されている回路との間で推移的なルーティングが提供されないためです。  

Azure VMware Solution からの ExpressRoute 回線を既存の ExpressRoute ゲートウェイに接続する場合は、デプロイ後に実行できます。  

そのため、Azure VMware Solution を既存の ExpressRoute ゲートウェイに接続するか、というのがまとめの質問です。  

* **する** = デプロイ時に使用されない仮想ネットワークを特定します。
* **しない** = 既存の仮想ネットワークを選択するか、デプロイ時に新しく作成します。

どちらの方法でも、この手順で行うことを文書化しておきます。

>[!NOTE]
>この仮想ネットワークは、オンプレミス環境と Azure VMware Solution から認識されるため、この仮想ネットワークで使用する IP セグメントとサブネットが重複しないようにしてください。

:::image type="content" source="media/pre-deployment/azure-vmware-solution-expressroute-diagram.png" alt-text="特定する - Azure VMware Solution を接続するための Azure Virtual Network" border="false":::

## <a name="vmware-hcx-network-segments"></a>VMware HCX のネットワーク セグメント

VMware HCX は、Azure VMware Solution にバンドルされているテクノロジです。 VMware HCX の主なユース ケースは、ワークロードの移行とディザスター リカバリーです。 いずれかの実施を計画する場合は、ここでネットワークを計画するのが最適です。   そうしない場合は、次の手順までスキップして続行できます。

[!INCLUDE [hcx-network-segments](includes/hcx-network-segments.md)]

## <a name="next-steps"></a>次の手順
これで必要な情報を収集して文書化したので、次のセクションに進み、Azure VMware Solution プライベート クラウドを作成します。

> [!div class="nextstepaction"]
> [Azure VMware Solution のデプロイ](deploy-azure-vmware-solution.md)
