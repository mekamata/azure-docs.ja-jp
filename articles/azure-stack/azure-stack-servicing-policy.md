﻿---
title: Azure Stack サービス ポリシー | Microsoft Docs
description: Azure Stack サービス ポリシーの詳細と、サポートを受けられる状態に統合システムを維持する方法を説明します。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: caac3d2f-11cc-4ff2-82d6-52b58fee4c39
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/03/2018
ms.author: mabrigg
ms.openlocfilehash: e37b63580d8cea4b5772bc54f7b2f79980afc0bc
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/05/2018
---
# <a name="azure-stack-servicing-policy"></a>Azure Stack サービス ポリシー
この記事では、Azure Stack 統合システムのサービス ポリシーについて、およびサポートを受けられる状態にシステムを維持する必要があることについて説明します。 

## <a name="update-package-types"></a>更新プログラム パッケージの種類

統合システムの更新プログラム パッケージには、Microsoft ソフトウェアの更新プログラムと、ドライバーやファームウェアなどの OEM (original equipment manufacturer) ハードウェア ベンダーが指定する更新プログラムの、2 種類があります。 これらの更新プログラムは、個別の Azure Stack 更新プログラム パッケージとして配信され、別々に管理されます。

- **Microsoft ソフトウェア更新プログラム**。 Microsoft には、Microsoft ソフトウェア更新プログラム パッケージについて、エンド ツー エンドでサービスを提供し続ける責任があります。 これらのパッケージには、最新の Windows Server のセキュリティ更新プログラム、セキュリティ以外の更新プログラム、Azure Stack の機能の更新プログラムが含まれます。 これらの更新プログラム パッケージは Microsoft から直接ダウンロードできます。
- **OEM ハードウェア ベンダー提供の更新プログラム**。 Azure Stack ハードウェア パートナーには、ハードウェアに関連するファームウェアおよびドライバーの更新プログラム パッケージについて、ガイドを含めエンド ツー エンドでサービスを提供し続ける責任があります。 また、Azure Stack ハードウェア パートナーは、ハードウェア ライフサイクル ホストのソフトウェアとハードウェアすべてに関するガイドを所有して管理します。 OEM ハードウェア ベンダーは、自社のダウンロード サイトでこれらの更新プログラム パッケージを管理します。

## <a name="update-package-release-cadence"></a>更新プログラム パッケージのリリース周期

Microsoft は、毎月ソフトウェア更新プログラム パッケージをリリースします。 ただし、1 か月に複数の更新プログラムをリリースすることや、1 つも更新プログラムをリリースしないこともあります。 OEM ハードウェア ベンダーは、必要に応じて随時更新プログラムをリリースします。

Microsoft 更新プログラム パッケージでは、リリース日を確認しやすくするために次のような命名規則を採用しています。

*MajorProductVersion.MinorProductVersion.YYMMDD.BuildNumber*

たとえば、2017 年 6 月 15 日にリリースされた Microsoft ソフトウェアの更新プログラムは、バージョンが "1.0.170615.1" になります。

## <a name="keep-your-system-under-support"></a>システムがサポートされる状態を維持する
引き続きサポートを受けるには、Azure Stack のデプロイを最新の状態に保つ必要があります。 更新の遅延に対するポリシーとして、Azure Stack が引き続きサポート対象となるには、リリースされた最新バージョンの更新プログラム、またはそれより 2 つ前までのメジャー バージョンの更新プログラムのいずれかが実行されている必要があります。  修正プログラムはメジャー バージョンの更新プログラムとは見なされません。  Azure Stack のクラウドが *2 つ以上前の更新プログラム*の場合は非準拠と見なされ、サポートを受けるには最小限サポートされるバージョンに更新する必要があります。 

たとえば、最新バージョンの更新プログラムが 1805 の場合、その前の 2 つの更新プログラム パッケージは 1804 と 1803 であるため、1804 と 1803 は両方ともサポートされます。 ただし、1802 はサポート外です。 今後 1、2 か月のうちにリリースがないときはこのポリシーが適用されます。 たとえば、最新のリリースが 1805 で 1804 のリリースがなかった場合、その前の 2 つの更新プログラム パッケージ 1803 と 1802 がサポートされます。

Microsoft ソフトウェア更新プログラム パッケージは累積的ではないため、前提として前のバージョンの更新パッケージが必要です。 1 つ以上の更新プログラムを先送りにする場合は、最新のバージョンを取得する際に全体のランタイムを考慮してください。 


## <a name="next-steps"></a>次の手順

- [Azure Stack での更新の管理](azure-stack-updates.md)


