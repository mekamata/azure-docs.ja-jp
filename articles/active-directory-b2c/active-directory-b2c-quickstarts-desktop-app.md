---
title: Azure AD B2C 対応デスクトップ アプリの試用
description: Azure Active Directory B2C を使用してユーザー ログインを提供するサンプル ASP.NET デスクトップ アプリを試用するためのクイック スタートです。
services: active-directory-b2c
author: davidmu1
manager: mtillman
editor: ''
ms.service: active-directory-b2c
ms.workload: identity
ms.topic: quickstart
ms.custom: mvc
ms.date: 2/13/2018
ms.author: davidmu
ms.openlocfilehash: 8726a5816623d42deb495aa062ef82f1f95b76b7
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/23/2018
---
# <a name="quickstart-test-drive-an-azure-ad-b2c-enabled-desktop-app"></a>クイック スタート: Azure AD B2C 対応デスクトップ アプリの試用

Azure Active Directory (Azure AD) B2C は、アプリケーション、ビジネス、顧客を保護するためのクラウド ID 管理を提供します。 Azure AD B2C に対応したアプリは、オープンな標準プロトコルを使用し、ソーシャル アカウントやエンタープライズ アカウントで認証を行うことができます。

このクイックスタートでは、Azure AD B2C 対応のサンプル WPF (Windows Presentation Foundation) デスクトップ アプリにソーシャル ID プロバイダーを使ってサインインし、Azure AD B2C で保護された Web API を呼び出します。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>前提条件

* [Visual Studio 2017](https://www.visualstudio.com/downloads/) と **ASP.NET および開発**ワークロード。 
* Facebook、Google、Microsoft、または Twitter のソーシャル アカウント。

## <a name="download-the-sample"></a>サンプルのダウンロード

[ZIP ファイルをダウンロード](https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop/archive/master.zip)するか、GitHub からサンプル Web アプリを複製します。

```
git clone https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop.git
```

## <a name="run-the-app-in-visual-studio"></a>Visual Studio でのアプリの実行

サンプル アプリケーションのプロジェクト フォルダーにある `active-directory-b2c-wpf.sln` ソリューションを Visual Studio で開きます。

**F5** キーを押してアプリケーションをデバッグします。

## <a name="create-an-account"></a>アカウントの作成

**[Sign in]\(サインイン\)** をクリックすると、Azure AD B2C のポリシーに基づく**サインアップまたはサインイン** ワークフローが開始されます。

![サンプル アプリケーション](media/active-directory-b2c-quickstarts-desktop-app/wpf-sample-application.png)

このサンプルは、ソーシャル ID プロバイダーを使用する方法や、メール アドレスを使用してローカル アカウントを作成する方法など、複数のサインアップ方法に対応しています。 このクイック スタートでは、Facebook、Google、Microsoft、または Twitter のいずれかのソーシャル ID プロバイダー アカウントを使用します。 

### <a name="sign-up-using-a-social-identity-provider"></a>ソーシャル ID プロバイダーを使用してサインアップする

このサンプル Web アプリでは、Wingtip Toys という架空のブランドのカスタム ログイン ページが Azure AD B2C により表示されます。 

1. ソーシャル ID プロバイダーを使用してサインアップするには、使用する ID プロバイダーのボタンをクリックします。 

    ![サインインまたはサインアップ用のプロバイダー](media/active-directory-b2c-quickstarts-desktop-app/sign-in-or-sign-up-wpf.png)

    ユーザーは、ソーシャル アカウントの資格情報を使用して認証 (サインイン) を行うと共に、ソーシャル アカウントから情報を読み取ることについての承認をアプリケーションに与えることになります。 アクセスを許可することにより、アプリケーションはソーシャル アカウントからプロファイル情報 (名前やお住まいの都市など) を取得できるようになります。 

2. ID プロバイダーのサインイン プロセスを完了します。 たとえば、Twitter を選択した場合、Twitter の資格情報を入力し、**[サインイン]** をクリックします。

    ![ソーシャル アカウントを使用して認証と承認を行う](media/active-directory-b2c-quickstarts-desktop-app/twitter-authenticate-authorize-wpf.png)

    新しいアカウントのプロファイルの詳細には、ソーシャル アカウントからの情報があらかじめ設定されています。 

3. 必要に応じて詳細を変更し、**[Continue]\(続行\)** をクリックします。 入力した値は、Azure AD B2C ユーザー アカウントのプロファイルに使用されます。

    ![新しいアカウントのサインアップ用プロファイルの詳細](media/active-directory-b2c-quickstarts-desktop-app/new-account-sign-up-profile-details-wpf.png)

    ID プロバイダーを使用する新しい Azure AD B2C ユーザー アカウントが正常に作成されました。 サインイン後に、アクセス トークンが *[Token info]\(トークン情報\)* テキスト ボックスに表示されます。 アクセス トークンは、API リソースにアクセスするときに使用されます。

## <a name="edit-your-profile"></a>プロファイルの編集

Azure Active Directory B2C には、ユーザーが自分のプロファイルを更新することができる機能があります。  このサンプル Web アプリのワークフローには、Azure AD B2C の編集プロファイル ポリシーが使用されます。 

1. 作成したプロファイルを編集するには、**[Edit profile]\(プロファイルの編集\)** をクリックします。

    ![プロファイルを編集する](media/active-directory-b2c-quickstarts-desktop-app/edit-profile-wpf.png)

2. 作成したアカウントに関連付けられている ID プロバイダーを選択します。 たとえば、アカウントの作成時に ID プロバイダーとして Twitter を使用した場合は、関連付けられているプロファイルの詳細を変更するために Twitter を選択します。

3. **表示名**や**都市**を変更し、**[Continue]\(続行\)** をクリックします。

    新しいアクセス トークンが *[Token info]\(トークン情報\)* テキスト ボックスに表示されます。 プロファイルに対する変更を確認する場合は、アクセス トークンをコピーしてトークン デコーダー https://jwt.ms に貼り付けます。

## <a name="access-a-protected-web-api-resource"></a>保護された Web API リソースにアクセスする

**[Call API]\(API の呼び出し\)** をクリックして、Azure AD B2C で保護されたリソース https://fabrikamb2chello.azurewebsites.net/hello に対して要求を送信します。 

![API の呼び出し](media/active-directory-b2c-quickstarts-desktop-app/call-api-wpf.png)

このアプリケーションは、保護された Web API リソースへの要求に Azure AD アクセス トークンを追加します。 Web API からは、アクセス トークンに含まれている表示名が返されます。

Azure AD B2C ユーザー アカウントを使用して、Azure AD B2C で保護された Web API の承認済みの呼び出しを正しく行いました。

## <a name="clean-up-resources"></a>リソースのクリーンアップ

他の Azure AD B2C クイックスタートやチュートリアルを試す場合は、Azure AD B2C テナントを使用できます。 不要になったら、[Azure AD B2C テナントを削除する](active-directory-b2c-faqs.md#how-do-i-delete-my-azure-ad-b2c-tenant)ことができます。

## <a name="next-steps"></a>次の手順

次のステップでは、独自の Azure AD B2C テナントを作成し、そのテナントを使用して実行するようにサンプルを構成します。 

> [!div class="nextstepaction"]
> [Azure Portal で Azure Active Directory B2C テナントを作成する](active-directory-b2c-get-started.md)
