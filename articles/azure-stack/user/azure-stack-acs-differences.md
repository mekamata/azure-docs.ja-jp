---
title: 'Azure Stack Storage : 違いと考慮事項'
description: Azure Stack のデプロイに関する考慮事項と一緒に、Azure Stack Storage と Azure Storage の相違点について説明します。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
ms.reviwer: xiaofmao
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 02/21/2017
ms.author: jeffgilb
ms.openlocfilehash: 7c4f030018f388302c3b60a41086bbd97c86513d
ms.sourcegitcommit: 168426c3545eae6287febecc8804b1035171c048
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/08/2018
---
# <a name="azure-stack-storage-differences-and-considerations"></a>Azure Stack Storage : 違いと考慮事項

*適用先: Azure Stack 統合システムと Azure Stack Development Kit*

Azure Stack Storage は、Microsoft Azure Stack 内のストレージ クラウド サービスのセットです。 Azure Stack Storage では、Azure と整合性のあるセマンティクスによって BLOB、テーブル、キュー、アカウント管理機能が提供されます。

この記事では、Azure Stack Storage と Azure Storage について明らかになっている相違点をまとめています。 また、Azure Stack をデプロイするときに留意するその他の考慮事項についてもまとめています。 Azure Stack と Azure の違いの概要については、[重要な考慮事項](azure-stack-considerations.md)のトピックを参照してください。

## <a name="cheat-sheet-storage-differences"></a>チート シート: ストレージの相違点

| Feature | Azure (グローバル) | Azure Stack |
| --- | --- | --- |
|File Storage|クラウド ベースの SMB ファイル共有のサポート|まだサポートされていません
|Azure Storage Service Encryption for Data at Rest|256 ビット AES 暗号化|BitLocker 128 ビット AES 暗号化
|ストレージ アカウントの種類|汎用アカウントと Azure Blob ストレージ アカウント|汎用目的のみ。
|レプリケーション オプション|ローカル冗長ストレージ、geo 冗長ストレージ、読み取りアクセス geo 冗長ストレージ、およびゾーン冗長ストレージ|ローカル冗長ストレージ。
|Premium Storage|完全にサポートされます|プロビジョニング可能ですがパフォーマンス制限や保証がありません。
|管理ディスク|Premium および標準がサポートされます|まだサポートされていません。
|BLOB 名|1,024 文字 (2,048 バイト)|880 文字 (1,760 バイト)
|ブロック BLOB の最大サイズ|4.75 TB (100 MB X 50,000 ブロック)|1802 update 以降のバージョンでは、4.75 TB (100 MB x 50,000 ブロック)。 それより前のバージョンでは 50,000 X 4 MB (約 195 GB)。
|ページ BLOB のスナップショット コピー|実行中の VM にアタッチされている Azure の管理対象外 VM ディスクのバックアップはサポートされています|まだサポートされていません。
|ページ BLOB の増分スナップショットのコピー|Premium および標準の Azure ページ BLOB がサポートされます|まだサポートされていません。
|Blob Storage のストレージ層|ホット ストレージ層、クール ストレージ層、アーカイブ ストレージ層。|まだサポートされていません。
Blob Storage の論理的な削除|プレビュー|まだサポートされていません。
|ページ BLOB の最大サイズ|8 TB|1 TB (テラバイト)
|ページ BLOB のページ サイズ|512 バイト|4 KB
|テーブルのパーティション キーと行キーのサイズ|1,024 文字 (2,048 バイト)|400 文字 (800 バイト)

### <a name="metrics"></a>メトリック
ストレージ メトリックにもいくつかの相違点があります。
* ストレージ メトリックのトランザクション データでは、内部と外部のネットワーク帯域幅が区別されません。
* ストレージ メトリックのトランザクション データには、マウントされたディスクへの仮想マシンのアクセスは含まれません。

## <a name="api-version"></a>API バージョン
Azure Stack Storage では以下のバージョンがサポートされます。

Azure Storage サービスの API:

1802 update 以降:
 - [2017-04-17](https://docs.microsoft.com/en-us/rest/api/storageservices/version-2017-04-17)
 - [2016-05-31](https://docs.microsoft.com/en-us/rest/api/storageservices/version-2016-05-31)
 - [2015-12-11](https://docs.microsoft.com/en-us/rest/api/storageservices/version-2015-12-11)
 - [2015-07-08 ](https://docs.microsoft.com/en-us/rest/api/storageservices/version-2015-07-08)
 - [2015-04-05](https://docs.microsoft.com/en-us/rest/api/storageservices/version-2015-04-05)

以前のバージョン:
 - [2015-04-05](https://docs.microsoft.com/en-us/rest/api/storageservices/version-2015-04-05)


Azure Storage サービスの Management API:

 - [2015-05-01-preview](https://docs.microsoft.com/rest/api/storagerp/?redirectedfrom=MSDN)
 - [2015-06-15](https://docs.microsoft.com/rest/api/storagerp/?redirectedfrom=MSDN)
 - [2016-01-01](https://docs.microsoft.com/rest/api/storagerp/?redirectedfrom=MSDN)

## <a name="sdk-versions"></a>SDK バージョン

Azure Stack Storage では次のクライアント ライブラリがサポートされます。

| クライアント ライブラリ | Azure Stack でサポートされるバージョン | リンク                                                                                                                                                                                                                                                                                                                                     | エンドポイントの指定       |
|----------------|-------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------|
| .NET           | 6.2.0 から 8.7.0          | NuGet パッケージ:<br>https://www.nuget.org/packages/WindowsAzure.Storage/<br> <br>GitHub リリース:<br>https://github.com/Azure/azure-storage-net/releases                                                                                                                                                                                    | app.config ファイル              |
| Java           | 4.1.0 から 6.1.0           | Maven パッケージ:<br>http://mvnrepository.com/artifact/com.microsoft.azure/azure-storage<br> <br>GitHub リリース:<br>https://github.com/Azure/azure-storage-java/releases                                                                                                                                                                    | 接続文字列の設定      |
| Node.js        | 1.1.0 から 2.7.0           | NPM リンク:<br>https://www.npmjs.com/package/azure-storage<br>(たとえば "npm install azure-storage@2.7.0" を実行します)<br> <br>GitHub リリース:<br>https://github.com/Azure/azure-storage-node/releases                                                                                                                                         | サービス インスタンスの宣言 |
| C++            | 2.4.0 から 3.1.0           | NuGet パッケージ:<br>https://www.nuget.org/packages/wastorage.v140/<br> <br>GitHub リリース:<br>https://github.com/Azure/azure-storage-cpp/releases                                                                                                                                                                                          | 接続文字列の設定      |
| PHP            | 0.15.0 から 1.0.0          | GitHub リリース:<br>https://github.com/Azure/azure-storage-php/releases<br> <br>Composer 経由でインストールする (下記参照)                                                                                                                                                                                                                  | 接続文字列の設定      |
| Python         | 0.30.0 から 1.0.0          | GitHub リリース:<br>https://github.com/Azure/azure-storage-python/releases                                                                                                                                                                                                                                                                | サービス インスタンスの宣言 |
| Ruby           | 0.12.1 から 1.0.1          | RubyGems パッケージ:<br>共通:<br>https://rubygems.org/gems/azure-storage-common/<br>BLOB: https://rubygems.org/gems/azure-storage-blob/<br>キュー: https://rubygems.org/gems/azure-storage-queue/<br>テーブル: https://rubygems.org/gems/azure-storage-table/<br> <br>GitHub リリース:<br>https://github.com/Azure/azure-storage-ruby/releases | 接続文字列の設定      |

## <a name="next-steps"></a>次の手順

* [Azure Stack Storage の開発ツールの概要](azure-stack-storage-dev.md)
* [Azure Stack Storage の概要](azure-stack-storage-overview.md)

