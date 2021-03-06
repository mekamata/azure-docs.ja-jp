---
title: "Azure Service Bus の Premium および Standard メッセージング価格レベルの概要 | Microsoft Docs"
description: "Service Bus の Premium および Standard メッセージング レベル"
services: service-bus-messaging
documentationcenter: .net
author: djrosanova
manager: timlt
editor: 
ms.assetid: e211774d-821c-4d79-8563-57472d746c58
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 11/10/2017
ms.author: sethm
ms.openlocfilehash: 613bb074063e436cdbd54fe5aee9c49109a2d8f2
ms.sourcegitcommit: 6a22af82b88674cd029387f6cedf0fb9f8830afd
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/11/2017
---
# <a name="service-bus-premium-and-standard-messaging-tiers"></a>Service Bus の Premium および Standard メッセージング レベル

Service Bus メッセージングには、キューやトピックなどのエンティティが含まれており、エンタープライズ メッセージング機能と、クラウド スケールの豊富な発行/サブスクライブ セマンティクスが結合されます。 Service Bus メッセージングは、多くの高度なクラウド ソリューションで、通信のバックボーンとして使用されます。

Service Bus メッセージングに *Premium* レベルを導入して、ミッション クリティカルなアプリケーションのスケール、パフォーマンス、および可用性に関する顧客の一般的な要求に対処しています。 機能セットとほぼ同じですが、Service Bus メッセージングのこれら 2 つのレベルは、さまざまなユース ケースに応えるように設計されています。

次の表に、大まかな違いをいくつか示します。

| プレミアム | Standard |
| --- | --- |
| 高スループット |変わりやすいスループット |
| 予測可能なパフォーマンス |変わりやすい待機時間 |
| 固定価格 |従量性の変わりやすい料金 |
| ワークロードをスケールアップおよびスケールダウンする機能 |該当なし |
| 最大 1 MB のメッセージ サイズ |最大 256 KB のメッセージ サイズ |

**Service Bus Premium メッセージング**では、各顧客のワークロードが分離した状態で実行されるように、CPU とメモリのレベルでリソースが分離されます。 このリソースのコンテナーを、*メッセージング ユニット*と呼びます。 各 Premium 名前空間には、1 つ以上のメッセージング ユニットが割り当てられます。 Service Bus の Premium 名前空間ごとに、1 個、2 個、または 4 個のメッセージング ユニットを購入できます。 1 つのワークロードまたはエンティティは、複数のメッセージング ユニットにまたがることができます。課金は 24 時間単位、すなわち日単位ですが、メッセージング ユニットの数は自由に変更できます。 その結果、Service Bus ベースのソリューションのパフォーマンスは、予測可能で反復可能になります。

このパフォーマンスは、より予測可能かつ利用可能なだけでなく、より高速です。 Service Bus Premium メッセージングは、[Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/) に導入されたストレージ エンジンが基盤になっています。 Premium メッセージングでのピークのパフォーマンスは、Standard レベルよりもはるかに高速です。

## <a name="premium-messaging-technical-differences"></a>Premium メッセージングの技術的な相違点

次のセクションでは、Premium メッセージング レベルと Standard メッセージング レベルのいくつかの違いについて説明します。

### <a name="partitioned-queues-and-topics"></a>パーティション分割されたキューとトピック

パーティション分割されたキューとトピックは Premium メッセージングでサポートされます。実際には、これらのエンティティは常にパーティション分割されます (無効にすることはできません)。 ただし、Premium のパーティション分割されたキューとトピックは、Standard レベルの Service Bus メッセージングと同様には機能しません。 Premium メッセージングは SQL をデータ ストアとして使用しないため、共有プラットフォームに関連するリソース競合が発生する可能性がなくなりました。 その結果、パフォーマンス向上のためのパーティション分割は必要ありません。 さらに、パーティション数は、Standard メッセージングでの 16 から、Premium メッセージングでは 2 に変更されました。 2 個のパーティションによって可用性が確保されます。また、Premium の実行時環境にとって、2 個というのはより適切なパーティション数です。 

Premium メッセージングでは、[MaxSizeInMegabytes](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxsizeinmegabytes#Microsoft_ServiceBus_Messaging_QueueDescription_MaxSizeInMegabytes) でエンティティのサイズを指定すると、サイズが 2 つのパーティションに均等に分割されます。これは、合計サイズが指定されたサイズの 16 倍になる [Standard のパーティション分割されたエンティティ](service-bus-partitioning.md#standard)とは異なります。 

パーティション分割の詳細については、「[パーティション分割されたキューとトピック](service-bus-partitioning.md)」を参照してください。

### <a name="express-entities"></a>エクスプレス エンティティ

完全に分離されたランタイム環境で Premium メッセージングが実行されるため、Premium 名前空間ではエクスプレス エンティティがサポートされません。 エクスプレス機能の詳細については、[QueueDescription.EnableExpress](/dotnet/api/microsoft.servicebus.messaging.queuedescription.enableexpress#Microsoft_ServiceBus_Messaging_QueueDescription_EnableExpress) プロパティを参照してください。

Standard メッセージングで実行しているコードがあり、それを Premium レベルに移植したい場合は、[EnableExpress](/dotnet/api/microsoft.servicebus.messaging.queuedescription.enableexpress#Microsoft_ServiceBus_Messaging_QueueDescription_EnableExpress) プロパティが **false** (既定値) に設定されていることを確認します。

## <a name="get-started-with-premium-messaging"></a>Premium メッセージングを使ってみる

Premium メッセージングは簡単に使い始めることができ、そのプロセスは Standard メッセージングと似ています。 まず、[Azure Portal](https://portal.azure.com) で[名前空間を作成](service-bus-create-namespace-portal.md)します。 **[価格レベルの選択]** で **[Premium]** を選択してください。

![create-premium-namespace][create-premium-namespace]

[Azure Resource Manager テンプレートを使用して Premium 名前空間](https://azure.microsoft.com/en-us/resources/templates/101-servicebus-pn-ar/)を作成することもできます。

## <a name="next-steps"></a>次のステップ

Service Bus メッセージングの詳細については、次のトピックを参照してください。

* [Azure Service Bus Premium メッセージングの概要 (ブログの投稿)](http://azure.microsoft.com/blog/introducing-azure-service-bus-premium-messaging/)
* [Azure Service Bus Premium メッセージングの概要 (Channel9)](https://channel9.msdn.com/Blogs/Subscribe/Introducing-Azure-Service-Bus-Premium-Messaging)
* [Service Bus メッセージングの概要](service-bus-messaging-overview.md)
* [Service Bus キューの使用](service-bus-dotnet-get-started-with-queues.md)

<!--Image references-->

[create-premium-namespace]: ./media/service-bus-premium-messaging/select-premium-tier.png
