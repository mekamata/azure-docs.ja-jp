---
title: AWS やその他のプラットフォームから Azure の Managed Disks に移行する | Microsoft Docs
description: AWS などの他のクラウドやその他の仮想化プラットフォームからアップロードされた VHD を使用して Azure で VM を作成し、Azure Managed Disks を活用します。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 10/07/2017
ms.author: cynthn
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 6d9fbfd07de9a5d536cf458dc478aade851d4b23
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/16/2018
---
# <a name="migrate-from-amazon-web-services-aws-and-other-platforms-to-managed-disks-in-azure"></a>アマゾン ウェブ サービス (AWS) やその他のプラットフォームから Azure の Managed Disks に移行する

AWS やオンプレミスの仮想化ソリューションから Azure に VHD ファイルをアップロードして、Managed Disks を利用する VM を作成できます。 Azure Managed Disks を使用すると、Azure IaaS VM のストレージ アカウントを管理する必要がなくなります。 必要なディスクの種類 (Premium または Standard) とサイズを指定すれば、ディスクの作成と管理は Azure によって行われます。 

一般化された VHD と特殊化された VHD のいずれもアップロードできます。 
- **一般化した VMD**: Sysprep を使用してすべての個人アカウント情報が削除されています。 
- **特殊化された VHD**: ユーザー アカウント、アプリケーション、その他の状態データが元の VM から保持されます。 

> [!IMPORTANT]
> VHD を Azure にアップロードする前に、「[Windows VHD の Azure へのアップロードの準備](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)」に従う必要があります
>
>


| シナリオ                                                                                                                         | ドキュメント                                                                                                                       |
|----------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| 管理ディスクを使用して Azure VM に移行する既存の AWS EC2 インスタンスがある                              | [PowerShell を使用してアマゾン ウェブ サービス (AWS) から Azure に Windows VM を移行する](aws-to-azure.md)                           |
| 複数の Azure VM を作成するためのイメージとして使用する他の仮想化プラットフォームの VM がある。 | [汎用化した VHD をアップロードして Azure で新しい VM を作成する](upload-generalized-managed.md) |
| Azure で再作成する一意にカスタマイズされた VM がある。                                                      | [特殊化されたディスクからの Windows VM の作成](create-vm-specialized.md)         |


## <a name="overview-of-managed-disks"></a>Managed Disks の概要

Azure Managed Disks は、ストレージ アカウントを管理する必要をなくして VM 管理をシンプルにします。 Managed Disks では、可用性セット内の VM の信頼性の向上というメリットもあります。 単一障害点を避けるために、可用性セット内の異なる VM のディスクは相互に十分に分離されます。 可用性セット内の異なる VM のディスクは異なるストレージ スケール ユニット (スタンプ) に自動的に配置されるため、ハードウェアとソフトウェアの障害を原因とする単一のストレージ スケール ユニット障害の影響が限定されます。 ニーズに基づいて、2 種類のストレージ オプションから選ぶことができます。 
 
- [Premium Managed Disks](premium-storage.md) は、ソリッド ステート ドライブ (SSD) ベースのストレージ メディアで、I/O を集中的に行うワークロードを実行している仮想マシンに、高パフォーマンスで待ち時間の短いディスク サポートを提供します。 Premium Managed Disks に移行すると、これらのディスクの速度とパフォーマンスを最大限に高めることができます。  

- [Standard Managed Disks](standard-storage.md) はハード ディスク ドライブ (HDD) ベースのストレージ メディアを使用し、パフォーマンス変動の影響を受けにくい開発テストやアクセス頻度の少ないワークロードに最適です。  

## <a name="plan-for-the-migration-to-managed-disks"></a>Managed Disks への移行の計画

このセクションでは、VM とディスクの種類に関する最適な決定を行います。

管理されていないディスクから管理ディスクへの移行を計画している場合は、[仮想マシンの共同作成者](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)ロールが割り当てられているユーザーは VM のサイズを変更できないことに注意してください (それらは事前に変換されている可能性があるためです)。 管理ディスクを持つ VM が OS ディスク上での Microsoft.Compute/disks/write 権限をユーザーに要求するのは、これが理由です。

### <a name="location"></a>場所

Azure Managed Disks を使用できる場所を選びます。 Premium Managed Disks に移行する場合は、移行を計画しているリージョンで Premium Storage が使用可能であることも確認します。 使用できる場所に関する最新情報については、「[リージョン別の利用可能な製品](https://azure.microsoft.com/regions/#services)」をご覧ください。

### <a name="vm-sizes"></a>VM サイズ

Premium Managed Disks に移行する場合は、VM が配置されているリージョンで利用できる Premium Storage 対応サイズに合うように VM のサイズを更新する必要があります。 Premium Storage で対応できる VM サイズをご確認ください。 Azure VM のサイズの仕様は、「 [仮想マシンのサイズ](sizes.md)」に記載されています。
Premium Storage で動作する仮想マシンのパフォーマンス特性を確認し、ワークロードに最適な VM を選択してください。 ディスク トラフィックが流れるのに十分な帯域幅が VM で利用できることを確認します。

### <a name="disk-sizes"></a>ディスク サイズ

**Premium Managed Disks**

VM で使える Premium 管理ディスクには 3 種類あり、それぞれに特定の IOPS とスループットの制限があります。 VM の Premium ディスクの種類を選ぶ場合は、容量、パフォーマンス、スケーラビリティ、最大負荷に関するアプリケーションのニーズに基づいて、これらの制限を考慮してください。

| Premium ディスクの種類  | P10               | P20               | P30               |
|---------------------|-------------------|-------------------|-------------------|
| ディスク サイズ           | 128 GB            | 512 GB            | 1024 GB (1 TB)    |
| ディスクあたりの IOPS       | 500               | 2300              | 5000              |
| ディスクあたりのスループット | 100 MB/秒 | 150 MB/秒 | 200 MB/秒 |

**Standard Managed Disks**

VM で使用できる Standard Managed Disks は 5 種類あります。 それぞれ容量は異なりますが、IOPS とスループットの制限は同じです。 アプリケーションの容量のニーズに基づいて Standard Managed Disks の種類を選択してください。

| Standard ディスクの種類  | S4               | S6               | S10              | S20              | S30              |
|---------------------|------------------|------------------|------------------|------------------|------------------|
| ディスク サイズ           | 30 GB            | 64 GB            | 128 GB           | 512 GB           | 1024 GB (1 TB)   |
| ディスクあたりの IOPS       | 500              | 500              | 500              | 500              | 500              |
| ディスクあたりのスループット | 60 MB/秒 | 60 MB/秒 | 60 MB/秒 | 60 MB/秒 | 60 MB/秒 |

### <a name="disk-caching-policy"></a>ディスク キャッシュ ポリシー 

**Premium Managed Disks**

既定では、ディスクのキャッシュ ポリシーは、すべてのPremium データ ディスクに対して「*読み取り専用*」、VM にアタッチされた Premium オペレーティング システム ディスクに対して「*読み取り/書き込み*」です。 アプリケーションの IO パフォーマンスを最適化するには、この構成をお勧めします。 書き込み量の多いディスクや書き込み専用のディスク (SQL Server ログ ファイルなど) の場合は、ディスク キャッシュを無効にすることで、アプリケーションのパフォーマンスを向上できる場合があります。

### <a name="pricing"></a>価格

[Managed Disks の価格](https://azure.microsoft.com/en-us/pricing/details/managed-disks/)をご確認ください。 Premium Managed Disks の価格は、Premium 非管理対象ディスクと同じです。 一方、Standard Managed Disks の価格は、Standard Unmanaged Disks と異なります。


## <a name="next-steps"></a>次の手順

- VHD を Azure にアップロードする前に、「[Windows VHD の Azure へのアップロードの準備](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)」に従う必要があります
