---
title: ASP.NET Core SignalR의 인증 및 권한 부여
author: bradygaster
description: ASP.NET Core SignalR에서 인증 및 권한 부여를 사용 하는 방법에 대해 알아봅니다.
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 12/05/2019
no-loc:
- SignalR
uid: signalr/authn-and-authz
ms.openlocfilehash: 091cc9b2adc1f6a8fac79519884695d1c1725d2a
ms.sourcegitcommit: c0b72b344dadea835b0e7943c52463f13ab98dd1
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 12/06/2019
ms.locfileid: "74880413"
---
# <a name="authentication-and-authorization-in-aspnet-core-opno-locsignalr"></a>ASP.NET Core SignalR의 인증 및 권한 부여

작성자: [Andrew Stanton-Nurse](https://twitter.com/anurse)

[예제 코드 살펴보기 및 다운로드](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/authn-and-authz/sample/) [(다운로드 방법)](xref:index#how-to-download-a-sample)

## <a name="authenticate-users-connecting-to-a-opno-locsignalr-hub"></a>SignalR 허브에 연결 하는 사용자 인증

SignalR를 [ASP.NET Core 인증과](xref:security/authentication/identity) 함께 사용 하 여 사용자를 각 연결에 연결할 수 있습니다. 허브에서는 [HubConnectionContext](/dotnet/api/microsoft.aspnetcore.signalr.hubconnectioncontext.user) 속성을 사용 하 여 인증 데이터에 액세스할 수 있습니다. 인증을 통해 허브는 사용자와 연결 된 모든 연결에서 메서드를 호출할 수 있습니다. 자세한 내용은 [SignalR에서 사용자 및 그룹 관리 ](xref:signalr/groups)를 참조 하세요. 단일 사용자에게 다수의 연결을 연결할 수 있습니다.

다음은 SignalR 및 ASP.NET Core 인증을 사용 하는 `Startup.Configure`의 예입니다.

::: moniker range=">= aspnetcore-3.0"

```csharp
public void Configure(IApplicationBuilder app)
{
    ...

    app.UseStaticFiles();

    app.UseRouting();

    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHub<ChatHub>("/chat");
        endpoints.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
    });
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

```csharp
public void Configure(IApplicationBuilder app)
{
    ...

    app.UseStaticFiles();

    app.UseAuthentication();

    app.UseSignalR(hubs =>
    {
        hubs.MapHub<ChatHub>("/chat");
    });

    app.UseMvc(routes =>
    {
        routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
    });
}
```

> [!NOTE]
> SignalR를 등록 하 고 인증 미들웨어를 ASP.NET Core 하는 순서는 중요 합니다. `HttpContext`에 대 한 사용자가 SignalR 있도록 `UseSignalR` 하기 전에 항상 `UseAuthentication`를 호출 합니다.

::: moniker-end

### <a name="cookie-authentication"></a>쿠키 인증

브라우저 기반 앱에서 쿠키 인증을 사용 하면 기존 사용자 자격 증명이 SignalR 연결로 자동으로 전달 됩니다. 브라우저 클라이언트를 사용하면 추가적인 구성이 필요 없습니다. 사용자가 앱에 로그인 하면 SignalR 연결이 자동으로이 인증을 상속 합니다.

쿠키는 액세스 토큰을 전송하는 브라우저 고유의 방식이지만 비 브라우저 클라이언트에서도 쿠키를 전송할 수 있습니다. [.Net 클라이언트](xref:signalr/dotnet-client)를 사용 하는 경우 `.WithUrl` 호출에서 `Cookies` 속성을 구성 하 여 쿠키를 제공할 수 있습니다. 그러나 .NET 클라이언트에서 쿠키 인증을 사용 하려면 앱이 쿠키에 대 한 인증 데이터를 교환 하는 API를 제공 해야 합니다.

### <a name="bearer-token-authentication"></a>전달자 토큰 인증

클라이언트는 쿠키를 사용하는 대신 액세스 토큰을 제공할 수도 있습니다. 서버는 토큰의 유효성을 검사하고 이를 이용해서 사용자를 식별합니다. 이 유효성 검사는 연결이 설정될 때만 수행됩니다. 서버는 연결이 유지되는 동안 토큰이 해지되었는지를 확인하기 위해 자동으로 유효성을 재검사하지 않습니다.

서버에서 전달자 토큰 인증은 [JWT 전달자 미들웨어](/dotnet/api/microsoft.extensions.dependencyinjection.jwtbearerextensions.addjwtbearer)를 사용하여 구성합니다.

JavaScript 클라이언트에서는 [accessTokenFactory](xref:signalr/configuration#configure-bearer-authentication) 옵션을 통해서 토큰을 제공할 수 있습니다.

[!code-typescript[Configure Access Token](authn-and-authz/sample/wwwroot/js/chat.ts?range=52-55)]

.NET 클라이언트에는 토큰을 구성 하는 데 사용할 수 있는 유사한 [AccessTokenProvider](xref:signalr/configuration#configure-bearer-authentication) 속성이 있습니다.

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub", options =>
    { 
        options.AccessTokenProvider = () => Task.FromResult(_myAccessToken);
    })
    .Build();
```

> [!NOTE]
> 사용자가 제공 하는 액세스 토큰 함수는 SignalR에서 수행 하는 **모든** HTTP 요청 전에 호출 됩니다. 연결을 활성 상태로 유지하기 위해 토큰을 갱신해야 할 경우 (연결 중 만료될 수 있으므로), 이 함수 내에서 작업을 수행하고 갱신된 토큰을 반환해야 합니다.

표준 웹 API에서 전달자 토큰은 HTTP 헤더로 전송됩니다. 그러나 일부 전송을 사용 하는 경우 SignalR는 브라우저에서 이러한 헤더를 설정할 수 없습니다. WebSockets 및 서버-전송 이벤트를 사용할 경우 토큰은 쿼리 문자열 매개 변수로 전송됩니다. 서버에서이를 지원 하려면 추가 구성이 필요 합니다.

[!code-csharp[Configure Server to accept access token from Query String](authn-and-authz/sample/Startup.cs?name=snippet)]

> [!NOTE]
> 쿼리 문자열은 브라우저 API 제한으로 인해 Websocket 및 서버에서 전송한 이벤트에 연결할 때 브라우저에서 사용 됩니다. HTTPS를 사용 하는 경우 쿼리 문자열 값은 TLS 연결을 통해 보안이 유지 됩니다. 그러나 많은 서버에서 쿼리 문자열 값을 기록 합니다. 자세한 내용은 [ASP.NET Core SignalR의 보안 고려 사항 ](xref:signalr/security)을 참조 하세요. SignalR는 헤더를 사용 하 여 토큰 (예: .NET 및 Java 클라이언트)을 지 원하는 환경에서 토큰을 전송 합니다.

### <a name="cookies-vs-bearer-tokens"></a>쿠키 대 전달자 토큰 

쿠키는 브라우저에만 적용 됩니다. 다른 종류의 클라이언트에서 전송 하면 전달자 토큰 전송과 비교 하 여 복잡성이 증가 합니다. 따라서 앱이 브라우저 클라이언트에서 사용자를 인증 해야 하는 경우를 제외 하 고는 쿠키 인증을 권장 하지 않습니다. 브라우저 클라이언트 이외의 클라이언트도 사용할 경우 전달자 토큰 인증이 권장되는 방식입니다.

### <a name="windows-authentication"></a>Windows 인증

앱에서 [Windows 인증](xref:security/authentication/windowsauth) 을 구성 하는 경우 해당 id를 사용 하 여 허브를 보호할 수 SignalR. 그러나 메시지를 개별 사용자에 게 보내려면 사용자 지정 사용자 ID 공급자를 추가 해야 합니다. Windows 인증 시스템이 "이름 식별자" 클레임을 제공 하지 않습니다. SignalR는 클레임을 사용 하 여 사용자 이름을 결정 합니다.

`IUserIdProvider`를 구현하는 새 클래스를 추가하고 사용자로부터 식별자로 사용할 클레임 중 하나를 가져옵니다. 예를 들어, "Name" 클레임(`[Domain]\[Username]` 형태의 Windows 사용자 이름)을 사용하려면 다음과 같은 클래스를 생성합니다.

[!code-csharp[Name based provider](authn-and-authz/sample/nameuseridprovider.cs?name=NameUserIdProvider)]

`ClaimTypes.Name`하는 대신 `User`의 모든 값 (예: Windows SID 식별자 등)을 사용할 수 있습니다.

> [!NOTE]
> 선택한 값은 시스템의 모든 사용자 간에 고유해야 합니다. 그렇지 않으면 특정 사용자를 대상으로 한 메시지가 다른 사용자에게 전달될 수 있습니다.

`Startup.ConfigureServices` 메서드에이 구성 요소를 등록 합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ... other services ...

    services.AddSignalR();
    services.AddSingleton<IUserIdProvider, NameUserIdProvider>();
}
```

.NET 클라이언트에서는 [UseDefaultCredentials](/dotnet/api/microsoft.aspnetcore.http.connections.client.httpconnectionoptions.usedefaultcredentials) 속성을 설정해서 Windows 인증을 활성화시켜야 합니다.

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/myhub", options =>
    {
        options.UseDefaultCredentials = true;
    })
    .Build();
```

Windows 인증은 Microsoft Internet Explorer 또는 Microsoft Edge를 사용하는 경우에만 브라우저 클라이언트에서 지원됩니다.

### <a name="use-claims-to-customize-identity-handling"></a>클레임을 사용 하 여 id 처리 사용자 지정

사용자를 인증 하는 앱은 사용자 클레임에서 사용자 Id SignalR 파생 시킬 수 있습니다. SignalR 사용자 Id를 만드는 방법을 지정 하려면 `IUserIdProvider`를 구현 하 고 구현을 등록 합니다.

샘플 코드는 클레임을 사용 하 여 사용자의 이메일 주소를 식별 속성으로 선택 하는 방법을 보여 줍니다. 

> [!NOTE]
> 선택한 값은 시스템의 모든 사용자 간에 고유해야 합니다. 그렇지 않으면 특정 사용자를 대상으로 한 메시지가 다른 사용자에게 전달될 수 있습니다.

[!code-csharp[Email provider](authn-and-authz/sample/EmailBasedUserIdProvider.cs?name=EmailBasedUserIdProvider)]

계정 등록은 ASP.NET id 데이터베이스에 `ClaimsTypes.Email` 유형의 클레임을 추가 합니다.

[!code-csharp[Adding the email to the ASP.NET identity claims](authn-and-authz/sample/pages/account/Register.cshtml.cs?name=AddEmailClaim)]

이 구성 요소를 `Startup.ConfigureServices`에 등록 합니다.

```csharp
services.AddSingleton<IUserIdProvider, EmailBasedUserIdProvider>();
```

## <a name="authorize-users-to-access-hubs-and-hub-methods"></a>허브 및 허브 메서드 접근에 대한 사용자 권한 부여

기본적으로, 인증되지 않은 사용자도 허브의 모든 메서드를 호출할 수 있습니다. 인증을 요구 하려면 [권한 부여](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute) 특성을 허브에 적용 합니다.

[!code-csharp[Restrict a hub to only authorized users](authn-and-authz/sample/Hubs/ChatHub.cs?range=8-10,32)]

`[Authorize]` 특성의 생성자 인수 및 속성을 이용해서 특정 [권한 부여 정책](xref:security/authorization/policies)을 만족하는 사용자만 접근할 수 있도록 제한할 수 있습니다. 예를 들어 `MyAuthorizationPolicy`라는 사용자 지정 권한 부여 정책이 존재할 경우, 다음과 같은 코드를 사용해서 해당 정책을 만족하는 사용자만 허브에 접근할 수 있도록 만들 수 있습니다.

```csharp
[Authorize("MyAuthorizationPolicy")]
public class ChatHub : Hub
{
}
```

각각의 허브 메서드에도 `[Authorize]` 특성을 적용할 수 있습니다. 현재 사용자가 메서드에 적용된 정책을 만족하지 않으면 호출자에게 오류가 반환됩니다.

```csharp
[Authorize]
public class ChatHub : Hub
{
    public async Task Send(string message)
    {
        // ... send a message to all users ...
    }

    [Authorize("Administrators")]
    public void BanUser(string userName)
    {
        // ... ban a user from the chat room (something only Administrators can do) ...
    }
}
```

::: moniker range=">= aspnetcore-3.0"

### <a name="use-authorization-handlers-to-customize-hub-method-authorization"></a>권한 부여 처리기를 사용 하 여 허브 메서드 권한 부여 사용자 지정

SignalR는 허브 메서드가 권한 부여를 요구할 때 권한 부여 처리기에 사용자 지정 리소스를 제공 합니다. 리소스가 `HubInvocationContext`의 인스턴스인 경우 `HubInvocationContext`는 `HubCallerContext`, 호출 되는 허브 메서드의 이름 및 허브 메서드에 대 한 인수를 포함 합니다.

Azure Active Directory를 통해 여러 조직 로그인을 허용 하는 대화방의 예를 생각해 보세요. Microsoft 계정 있는 사용자는 채팅에 로그인 할 수 있지만 소유 조직의 구성원만 사용자를 금지 하거나 사용자의 채팅 기록을 볼 수 있습니다. 또한 특정 사용자의 특정 기능을 제한 해야 할 수도 있습니다. ASP.NET Core 3.0의 업데이트 된 기능을 사용 하는 것이 가능 합니다. `DomainRestrictedRequirement` 사용자 지정 `IAuthorizationRequirement`으로 사용 되는 방법을 확인 합니다. `HubInvocationContext` 리소스 매개 변수가 전달 되 고 있으므로 내부 논리는 허브가 호출 되는 컨텍스트를 검사 하 고 사용자가 개별 허브 메서드를 실행 하도록 허용 하는 결정을 내릴 수 있습니다.

```csharp
[Authorize]
public class ChatHub : Hub
{
    public void SendMessage(string message)
    {
    }

    [Authorize("DomainRestricted")]
    public void BanUser(string username)
    {
    }

    [Authorize("DomainRestricted")]
    public void ViewUserHistory(string username)
    {
    }
}

public class DomainRestrictedRequirement : 
    AuthorizationHandler<DomainRestrictedRequirement, HubInvocationContext>, 
    IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context,
        DomainRestrictedRequirement requirement, 
        HubInvocationContext resource)
    {
        if (IsUserAllowedToDoThis(resource.HubMethodName, context.User.Identity.Name) && 
            context.User.Identity.Name.EndsWith("@microsoft.com"))
        {
            context.Succeed(requirement);
        }
        return Task.CompletedTask;
    }

    private bool IsUserAllowedToDoThis(string hubMethodName,
        string currentUsername)
    {
        return !(currentUsername.Equals("asdf42@microsoft.com") && 
            hubMethodName.Equals("banUser", StringComparison.OrdinalIgnoreCase));
    }
}
```

`Startup.ConfigureServices`에서 사용자 지정 `DomainRestrictedRequirement` 요구 사항을 매개 변수로 제공 하 여 `DomainRestricted` 정책을 만드는 새 정책을 추가 합니다.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ... other services ...

    services
        .AddAuthorization(options =>
        {
            options.AddPolicy("DomainRestricted", policy =>
            {
                policy.Requirements.Add(new DomainRestrictedRequirement());
            });
        });
}
```

앞의 예제에서 `DomainRestrictedRequirement` 클래스는 해당 요구 사항에 대 한 `IAuthorizationRequirement` 및 자체 `AuthorizationHandler`입니다. 이러한 두 구성 요소를 별도의 클래스로 분할 하 여 문제를 구분할 수 있습니다. 예제의 방법의 장점은 요구 사항과 처리기가 동일 하기 때문에 시작 하는 동안 `AuthorizationHandler`를 주입할 필요가 없다는 점입니다.

::: moniker-end

## <a name="additional-resources"></a>추가 자료

* [ASP.NET Core에서 전달자 토큰 인증](https://blogs.msdn.microsoft.com/webdev/2016/10/27/bearer-token-authentication-in-asp-net-core/)
* [리소스 기반 권한 부여](xref:security/authorization/resourcebased)
