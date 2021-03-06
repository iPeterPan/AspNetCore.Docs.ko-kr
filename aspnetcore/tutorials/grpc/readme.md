---
page_type: sample
description: 이 자습서는 ASP.NET Core에서 gRPC 서비스 및 gRPC 클라이언트를 만드는 방법을 보여줍니다.
languages:
- csharp
products:
- dotnet-core
- aspnet-core
- vs
urlFragment: create-grpc-client
ms.openlocfilehash: b9feb9eed62177358fffc0d7da582f625a431e32
ms.sourcegitcommit: 9e85c2562df5e108d7933635c830297f484bb775
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 11/04/2019
ms.locfileid: "73463051"
---
# <a name="create-a-grpc-client-and-server-in-aspnet-core-30-using-visual-studio"></a>Visual Studio를 사용하여 ASP.NET Core 3.0에서 gRPC 클라이언트 및 서버 만들기

이 자습서에서는 .NET Core [gRPC](https://grpc.io/docs/guides/) 클라이언트와 ASP.NET Core gRPC 서버를 만드는 방법을 보여줍니다.

자습서를 마치고 나면 gRPC Greeter 서비스와 통신하는 gRPC 클라이언트를 갖게 됩니다.

이 자습서에서는 다음을 수행합니다.

* gRPC 서버를 만듭니다.
* gRPC 클라이언트를 만듭니다.
* gRPC Greeter 서비스를 사용하여 gRPC 클라이언트 서비스를 테스트합니다.

## <a name="create-a-grpc-service"></a>gRPC 서비스 만들기

* Visual Studio **파일** 메뉴에서 **새로 만들기** > **프로젝트**를 선택합니다.
* **새 프로젝트 만들기** 대화 상자에서 **ASP.NET Core 웹 애플리케이션**을 선택합니다.
* **다음** 선택
* 프로젝트 이름을 **GrpcGreeter**로 지정합니다. 코드를 복사하여 붙여넣을 때 네임스페이스가 일치하도록 프로젝트 이름을 *GrpcGreeter*로 지정해야 합니다.
* **만들기**를 선택합니다.
* **새 ASP.NET Core 웹 애플리케이션 만들기** 대화 상자에서 다음을 수행합니다.
  * 드롭다운 메뉴에서 **.NET Core** 및 **ASP.NET Core 3.0**을 선택합니다. 
  * **gRPC 서비스** 템플릿을 선택합니다.
  * **만들기**를 선택합니다.

### <a name="run-the-service"></a>서비스 실행

* `Ctrl+F5`를 눌러 디버거 없이 gRPC 서비스를 실행합니다.

  Visual Studio는 명령 프롬프트에서 서비스를 실행합니다.

로그는 `https://localhost:5001`에서 서비스가 수신 대기 중임을 보여줍니다.

```console
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
```

> [!NOTE]
> gRPC 템플릿은 [TLS(전송 계층 보안)](https://tools.ietf.org/html/rfc5246)를 사용하도록 구성됩니다. gRPC 클라이언트는 HTTPS를 사용하여 서버를 호출해야 합니다.
>
> macOS는 TLS를 사용하는 ASP.NET Core gRPC를 지원하지 않습니다. macOS에서 gRPC 서비스를 성공적으로 실행하려면 추가 구성이 필요합니다. 자세한 내용은 [macOS의 gRPC 및 ASP.NET Core](xref:grpc/aspnetcore#grpc-and-aspnet-core-on-macos)를 참조하세요.

### <a name="examine-the-project-files"></a>프로젝트 파일 검토

*GrpcGreeter* 프로젝트 파일:

* *greet.proto*: *Protos/greet.proto* 파일은 `Greeter` gRPC를 정의하고 gRPC 서버 자산을 생성하는 데 사용됩니다. 자세한 내용은 [gRPC 소개](xref:grpc/index)를 참조하세요.
* *Services* 폴더: `Greeter` 서비스의 구현을 포함합니다.
* *appSettings.json*: Kestrel에서 사용하는 프로토콜과 같은 구성 데이터를 포함합니다. 자세한 내용은 <xref:fundamentals/configuration/index>을 참조하세요.
* *Program.cs*: gRPC 서비스의 진입점을 포함합니다. 자세한 내용은 <xref:fundamentals/host/generic-host>을 참조하세요.
* *Startup.cs*: 앱 동작을 구성하는 코드를 포함합니다. 자세한 내용은 [앱 시작](xref:fundamentals/startup)을 참조하세요.

## <a name="create-the-grpc-client-in-a-net-console-app"></a>.NET 콘솔 앱에서 gRPC 클라이언트 만들기

* Visual Studio의 두 번째 인스턴스를 엽니다.
* 메뉴 모음에서 **파일** > **새로 만들기** > **프로젝트**를 선택합니다.
* **새 프로젝트 만들기** 대화 상자에서 **콘솔 앱(.NET Core)** 을 선택합니다.
* **다음** 선택
* **이름** 텍스트 상자에 "GrpcGreeterClient"를 입력합니다.
* **만들기**를 선택합니다.

### <a name="add-required-packages"></a>필수 패키지 추가

gRPC 클라이언트 프로젝트에는 다음 패키지가 필요합니다.

* [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client)는 .NET Core 클라이언트를 포함합니다.
* [Google.Protobuf](https://www.nuget.org/packages/Google.Protobuf/)는 C#용 protobuf 메시지 API를 포함합니다.
* [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/)는 protobuf 파일에 대한 C# 도구 지원을 포함합니다. 도구 패키지는 런타임에는 필요하지 않으므로 종속성은 `PrivateAssets="All"`로 표시됩니다.

PMC(패키지 관리자 콘솔) 또는 NuGet 패키지 관리를 사용하여 패키지를 설치합니다.

#### <a name="pmc-option-to-install-packages"></a>PMC를 사용한 패키지 설치

* Visual Studio에서 **도구** > **NuGet 패키지 관리자** > **패키지 관리자 콘솔**을 선택합니다.
* **패키지 관리자 콘솔** 창에서 *GrpcGreeterClient.csproj* 파일이 있는 디렉터리로 이동합니다.
* 다음 명령을 실행합니다.

 ```powershell
Install-Package Grpc.Net.Client
Install-Package Google.Protobuf
Install-Package Grpc.Tools
```

#### <a name="manage-nuget-packages-option-to-install-packages"></a>NuGet 패키지 관리를 사용한 패키지 설치

* **솔루션 탐색기** > **NuGet 패키지 관리**에서 프로젝트를 마우스 오른쪽 단추로 클릭
* **찾아보기** 탭을 선택합니다.
* 검색 상자에 **Grpc.Net.Client**를 입력합니다.
* **찾아보기** 탭에서 **Grpc.Net.Client** 패키지를 선택하고 **설치**를 선택합니다.
* `Google.Protobuf` 및 `Grpc.Tools`에 대해 반복하세요.

### <a name="add-greetproto"></a>greet.proto 추가

* gRPC 클라이언트 프로젝트에서 **Protos** 폴더를 만듭니다.
* gRPC Greeter 서비스에서 **Protos\greet.proto** 파일을 gRPC 클라이언트 프로젝트로 복사합니다.
* *GrpcGreeterClient.csproj* 프로젝트 파일을 편집합니다.

  프로젝트를 마우스 오른쪽 단추로 클릭하고 **프로젝트 파일 편집**을 선택합니다.

* **greet.proto** 파일을 참조하는 `<Protobuf>` 요소를 포함하는 항목 그룹을 추가합니다.

  ```xml
  <ItemGroup>
    <Protobuf Include="Protos\greet.proto" GrpcServices="Client" />
  </ItemGroup>
  ```

### <a name="create-the-greeter-client"></a>Greeter 클라이언트 만들기

프로젝트를 빌드하여 `GrpcGreeter` 네임스페이스에 형식을 만듭니다. `GrpcGreeter` 형식은 빌드 프로세스에 의해 자동으로 생성됩니다.

gRPC 클라이언트 *Program.cs* 파일을 다음 코드로 수정합니다.

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;
using GrpcGreeter;
using Grpc.Net.Client;

namespace GrpcGreeterClient
{
    class Program
    {
        static async Task Main(string[] args)
        {
            // The port number(5001) must match the port of the gRPC server.
            var channel = GrpcChannel.ForAddress("https://localhost:5001");
            var client = new Greeter.GreeterClient(channel);
            var reply = await client.SayHelloAsync(
                              new HelloRequest { Name = "GreeterClient" });
            Console.WriteLine("Greeting: " + reply.Message);
            Console.WriteLine("Press any key to exit...");
            Console.ReadKey();
        }
    }
}
```

*Program.cs*는 gRPC 클라이언트의 진입점 및 논리를 포함합니다.

Greeter 클라이언트는 다음에 의해 생성됩니다.

* gRPC 서비스에 대한 연결을 만들기 위한 정보가 포함된 `GrpcChannel` 인스턴스화.
* `GrpcChannel`을 사용하여 Greeter 클라이언트를 구성합니다.

## <a name="test-the-grpc-client-with-the-grpc-greeter-service"></a>gRPC Greeter 서비스를 사용하여 gRPC 클라이언트 테스트

* Greeter 서비스에서 `Ctrl+F5`를 눌러 디버거 없이 서버를 시작합니다.
* `GrpcGreeterClient` 프로젝트에서 `Ctrl+F5`를 눌러 디버거 없이 클라이언트를 시작합니다.

클라이언트가 자신의 이름 “GreeterClient”가 포함된 메시지와 함께 인사말을 서비스에 보냅니다. 서비스는 "Hello GreeterClient"라는 메시지를 응답으로 보냅니다. "Hello GreeterClient" 응답이 명령 프롬프트에 표시됩니다.

```console
Greeting: Hello GreeterClient
Press any key to exit...
```

gRPC 서비스는 명령 프롬프트에 기록된 로그에 성공적인 호출의 세부 정보를 기록합니다.

```console
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: C:\GH\aspnet\docs\4\Docs\aspnetcore\tutorials\grpc\grpc-start\sample\GrpcGreeter
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/2 POST https://localhost:5001/Greet.Greeter/SayHello application/grpc
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 78.32260000000001ms 200 application/grpc
```

### <a name="docs-help--next-steps-for-grpc"></a>gRPC에 대한 문서 도움말 및 다음 단계

* [ASP.NET Core에서 gRPC 소개](https://docs.microsoft.com/aspnet/core/grpc/index?view=aspnetcore-3.0)
* [C#을 사용하는 gRPC 서비스](https://docs.microsoft.com/aspnet/core/grpc/basics?view=aspnetcore-3.0)
* [C Core에서 ASP.NET Core로 gRPC 서비스 마이그레이션](https://docs.microsoft.com/aspnet/core/grpc/migration?view=aspnetcore-3.0)
