---
title: ASP.NET Core에서 Azure Active Directory B2C를 사용 하는 클라우드 인증
author: camsoper
description: ASP.NET Core를 사용 하 여 Azure Active Directory B2C 인증을 설정 하는 방법을 알아봅니다.
ms.author: casoper
ms.custom: mvc
ms.date: 01/21/2019
uid: security/authentication/azure-ad-b2c
ms.openlocfilehash: 136fa47788456492a9a7fe6d9d9e5996c13e8c20
ms.sourcegitcommit: eca76bd065eb94386165a0269f1e95092f23fa58
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/24/2020
ms.locfileid: "76727282"
---
# <a name="cloud-authentication-with-azure-active-directory-b2c-in-aspnet-core"></a>ASP.NET Core에서 Azure Active Directory B2C를 사용 하는 클라우드 인증

작성자: [Cam Soper](https://twitter.com/camsoper)

[Azure Active Directory B2C](/azure/active-directory-b2c/active-directory-b2c-overview) (Azure AD B2C)은 웹 및 모바일 앱에 대 한 클라우드 id 관리 솔루션입니다. 서비스는 클라우드 및 온-프레미스에서 호스트 되는 앱에 대 한 인증을 제공 합니다. 인증 유형 포함할 개별 계정, 소셜 네트워크 계정 및 enterprise 계정 페더레이션 합니다. 또한 Azure AD B2C는 최소 구성으로 다단계 인증을 제공할 수 있습니다.

> [!TIP]
> Azure Active Directory (Azure AD) 및 Azure AD B2C는 별개 제품입니다. Azure AD 테 넌 트 조직을 나타내고 Azure AD B2C 테 넌 트를 신뢰 당사자 응용 프로그램을 사용 하 여 사용할 id의 컬렉션을 나타냅니다. 자세히 알아보려면 [Azure AD B2C: faq (질문과 대답)](/azure/active-directory-b2c/active-directory-b2c-faqs)를 참조 하세요.

이 자습서에서는 다음 방법을 알아봅니다.

> [!div class="checklist"]
> * Azure Active Directory B2C 테넌트 만들기
> * Azure AD B2C에 앱 등록
> * Visual Studio를 사용 하 여 인증을 위해 Azure AD B2C 테 넌 트를 사용 하도록 구성 된 ASP.NET Core 웹 앱 만들기
> * Azure AD B2C 테 넌 트의 동작을 제어 하는 정책 구성

## <a name="prerequisites"></a>필수 구성 요소

다음은이 연습에 필요한:

* [Microsoft Azure 구독](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)
* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019)

## <a name="create-the-azure-active-directory-b2c-tenant"></a>Azure Active Directory B2C 테 넌 트 만들기

[설명서에 설명 된 대로](/azure/active-directory-b2c/active-directory-b2c-get-started)Azure Active Directory B2C 테 넌 트를 만듭니다. 메시지가 표시 되 면 Azure 구독을 사용 하 여 테 넌 트 연결는 선택 사항이 자습서에 대 한 합니다.

## <a name="register-the-app-in-azure-ad-b2c"></a>Azure AD B2C에 앱 등록

새로 만든 Azure AD B2C 테 넌 트에서 설명서의 **웹 앱 등록** 섹션 아래 [에 있는 단계](/azure/active-directory-b2c/tutorial-register-applications#register-a-web-application) 를 사용 하 여 앱을 등록 합니다. **웹 앱 클라이언트 암호 만들기** 섹션에서 중지 합니다. 클라이언트 암호를이 자습서에 필요 하지 않습니다. 

다음 값을 사용합니다.

| 설정                       | 값                     | 참고 사항                                                                                                                                                                                              |
|-------------------------------|---------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Name**                      | *&lt;앱 이름&gt;*        | 소비자에 게 앱을 설명 하는 앱의 **이름을** 입력 합니다.                                                                                                                                 |
| **웹앱/웹 API 포함** | 예                       |                                                                                                                                                                                                    |
| **암시적 흐름 허용**       | 예                       |                                                                                                                                                                                                    |
| **회신 URL**                 | `https://localhost:44300/signin-oidc` | 회신 URL은 Azure AD B2C에서 앱이 요청한 토큰을 반환하는 엔드포인트입니다. Visual Studio는 사용할 회신 URL을 제공 합니다. 이제 `https://localhost:44300/signin-oidc`를 입력 하 여 양식을 완성 합니다. |
| **앱 ID URI**                | 비워 둠               | 이 자습서에 필요 하지 않습니다.                                                                                                                                                                    |
| **네이티브 클라이언트 포함**     | 아니요                        |                                                                                                                                                                                                    |

> [!WARNING]
> Localhost가 아닌 회신 URL을 설정 하는 경우 [회신 url 목록에서 허용 되는 항목에 대 한 제약 조건을](/azure/active-directory-b2c/tutorial-register-applications#register-a-web-application)알고 있어야 합니다. 

앱이 등록 되 면 테 넌 트의 앱 목록이 표시 됩니다. 방금 등록 한 앱을 선택 합니다. **응용 프로그램 ID** 필드의 오른쪽에 있는 **복사** 아이콘을 선택 하 여 클립보드에 복사 합니다.

지금은 Azure AD B2C 테 넌 트에서 더 이상 구성할 수 없지만 브라우저 창은 열어 둡니다. ASP.NET Core 앱을 만든 후에 더 많은 구성이 있습니다.

## <a name="create-an-aspnet-core-app-in-visual-studio"></a>Visual Studio에서 ASP.NET Core 앱 만들기

Visual Studio 웹 응용 프로그램 템플릿은 인증에 Azure AD B2C 테 넌 트를 사용 하도록 구성할 수 있습니다.

Visual Studio에서 다음을 수행합니다.

1. 새 ASP.NET Core 웹 애플리케이션을 만듭니다. 
2. 템플릿 목록에서 **웹 응용 프로그램** 을 선택 합니다.
3. **인증 변경** 단추를 선택 합니다.
    
    ![인증 변경 단추](./azure-ad-b2c/_static/changeauth.png)

4. **인증 변경** 대화 상자에서 **개별 사용자 계정**을 선택한 다음 드롭다운에서 **클라우드의 기존 사용자 저장소에 연결** 을 선택 합니다. 
    
    ![인증 대화 상자 변경](./azure-ad-b2c/_static/changeauthdialog.png)

5. 다음 값으로 양식을 완성 합니다.
    
    | 설정                       | 값                                                 |
    |-------------------------------|-------------------------------------------------------|
    | **도메인 이름**               | *B2C 테 넌 트의 도메인 이름을 &lt;&gt;*          |
    | **애플리케이션 ID**            | *클립보드의 응용 프로그램 ID를 붙여 &lt;&gt;* |
    | **콜백 경로**             | *기본값을 사용 &lt;&gt;*                       |
    | **등록 또는 로그인 정책** | `B2C_1_SiUpIn`                                        |
    | **암호 재설정 정책**     | `B2C_1_SSPR`                                          |
    | **프로필 정책 편집**       | *&lt;비워 둠&gt;*                                 |
    
    회신 uri 옆의 **복사** 링크를 **선택 하 여** 회신 uri를 클립보드로 복사 합니다. **확인** 을 선택 하 여 **인증 변경** 대화 상자를 닫습니다. **확인** 을 선택 하 여 웹 앱을 만듭니다.

## <a name="finish-the-b2c-app-registration"></a>B2C 앱 등록을 완료 합니다.

B2C 앱 속성이 계속 열려 있는 브라우저 창으로 돌아갑니다. 이전에 지정한 임시 **회신 URL** 을 Visual Studio에서 복사한 값으로 변경 합니다. 창 맨 위에 있는 **저장** 을 선택 합니다.

> [!TIP]
> 회신 URL을 복사 하지 않은 경우 웹 프로젝트 속성의 디버그 탭에서 HTTPS 주소를 사용 하 고 *appsettings*의 **callbackpath** 값을 추가 합니다.

## <a name="configure-policies"></a>정책 구성

Azure AD B2C 설명서의 단계를 사용 하 여 [등록 또는 로그인 정책을 만든](/azure/active-directory-b2c/active-directory-b2c-reference-policies#user-flow-versions)다음 [암호 재설정 정책을 만듭니다](/azure/active-directory-b2c/active-directory-b2c-reference-policies#user-flow-versions). **Id 공급자**, **등록 특성**및 **응용 프로그램 클레임**에 대 한 설명서에 제공 된 예제 값을 사용 합니다. 설명서에 설명 된 대로 **지금 실행** 단추를 사용 하 여 정책을 테스트 하는 것은 선택 사항입니다.

> [!WARNING]
> Visual Studio의 **인증 변경** 대화 상자에서 정책 이름이 사용 되었으므로 정책 이름이 설명서에 설명 된 것과 정확 하 게 일치 하는지 확인 합니다. 정책 이름은 *appsettings*에서 확인할 수 있습니다.

## <a name="configure-the-underlying-openidconnectoptionsjwtbearercookie-options"></a>기본 OpenIdConnectOptions/JwtBearer/Cookie 옵션 구성

기본 옵션을 직접 구성 하려면 `Startup.ConfigureServices`에서 적절 한 체계 상수를 사용 합니다.

```csharp
services.Configure<OpenIdConnectOptions>(
    AzureAD[B2C]Defaults.OpenIdScheme, options => 
    {
        // Omitted for brevity
    });

services.Configure<CookieAuthenticationOptions>(
    AzureAD[B2C]Defaults.CookieScheme, options => 
    {
        // Omitted for brevity
    });

services.Configure<JwtBearerOptions>(
    AzureAD[B2C]Defaults.JwtBearerAuthenticationScheme, options => 
    {
        // Omitted for brevity
    });
```

## <a name="run-the-app"></a>앱 실행

Visual Studio에서 **f5** 키를 눌러 앱을 빌드하고 실행 합니다. 웹 앱이 시작 되 면 **동의** 를 선택 하 여 쿠키 사용을 승인한 다음 (메시지가 표시 되는 경우) **로그인**을 선택 합니다.

![앱에 로그인](./azure-ad-b2c/_static/signin.png)

브라우저가 Azure AD B2C 테 넌 트로 리디렉션됩니다. 기존 계정 (정책 테스트를 만든 경우)으로 로그인 하거나 **지금 등록** 을 선택 하 여 새 계정을 만듭니다. **암호를 잊으셨나요?** 링크를 사용 하 여 잊어버린 암호를 다시 설정 합니다.

![Azure AD B2C 로그인](./azure-ad-b2c/_static/b2csts.png)

성공적으로 로그인 하면 브라우저가 웹 앱으로 리디렉션됩니다.

![성공](./azure-ad-b2c/_static/success.png)

## <a name="next-steps"></a>다음 단계

본 자습서에서는 다음 작업에 관한 방법을 학습했습니다.

> [!div class="checklist"]
> * Azure Active Directory B2C 테넌트 만들기
> * Azure AD B2C에 앱 등록
> * Visual Studio를 사용 하 여 인증을 위해 Azure AD B2C 테 넌 트를 사용 하도록 구성 된 ASP.NET Core 웹 응용 프로그램 만들기
> * Azure AD B2C 테 넌 트의 동작을 제어 하는 정책 구성

ASP.NET Core 앱이 인증을 위해 Azure AD B2C를 사용 하도록 구성 되었으므로 [권한 부여 특성](xref:security/authorization/simple) 을 사용 하 여 앱을 보호할 수 있습니다. 다음을 학습 하 여 앱을 계속 개발 합니다.

* [Azure AD B2C 사용자 인터페이스를 사용자 지정](/azure/active-directory-b2c/active-directory-b2c-reference-ui-customization)합니다.
* [암호 복잡성 요구 사항을 구성](/azure/active-directory-b2c/active-directory-b2c-reference-password-complexity)합니다.
* [Multi-factor authentication을 사용 하도록 설정](/azure/active-directory-b2c/active-directory-b2c-reference-mfa)합니다.
* [Microsoft](/azure/active-directory-b2c/active-directory-b2c-setup-msa-app), [Facebook](/azure/active-directory-b2c/active-directory-b2c-setup-fb-app), [Google](/azure/active-directory-b2c/active-directory-b2c-setup-goog-app), [Amazon](/azure/active-directory-b2c/active-directory-b2c-setup-amzn-app), [Twitter](/azure/active-directory-b2c/active-directory-b2c-setup-twitter-app)등의 추가 id 공급자를 구성 합니다.
* [AZURE AD Graph API를 사용](/azure/active-directory-b2c/active-directory-b2c-devquickstarts-graph-dotnet) 하 여 Azure AD B2C 테 넌 트에서 그룹 멤버 자격과 같은 추가 사용자 정보를 검색 합니다.
* [Azure AD B2C를 사용 하 여 ASP.NET Core WEB API를 보호](https://azure.microsoft.com/resources/samples/active-directory-b2c-dotnetcore-webapi/)합니다.
* [Azure AD B2C를 사용 하 여 .net 웹 앱에서 .net WEB API를 호출](/azure/active-directory-b2c/active-directory-b2c-devquickstarts-web-api-dotnet)합니다.