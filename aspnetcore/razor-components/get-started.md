---
title: Razor 구성 요소를 사용 하 여 시작
author: guardrex
description: 만들기 Razor 구성 요소 프로젝트를 수정 하 여 Razor 구성 요소를 사용 하 여 시작 하는 방법에 알아봅니다.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 04/07/2019
uid: razor-components/get-started
ms.openlocfilehash: 151e58497b0f22fa7c5a9bde1f665eeb73fd5dc3
ms.sourcegitcommit: 6bde1fdf686326c080a7518a6725e56e56d8886e
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/08/2019
ms.locfileid: "59515654"
---
# <a name="get-started-with-razor-components"></a>Razor 구성 요소를 사용 하 여 시작

작성자: [Daniel Roth](https://github.com/danroth27) 및 [Luke Latham](https://github.com/guardrex)

# [<a name="visual-studio"></a>Visual Studio](#tab/visual-studio)

필수 구성 요소:

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.0.md)]

Visual Studio에서 첫 번째 Razor 구성 요소 프로젝트를 만들려면:

1. 최신 설치 [.NET Core 3.0 미리 보기 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0) 릴리스 합니다.
1. 미리 보기 Sdk를 사용 하도록 Visual Studio를 사용 하도록 설정 합니다.
   1. 오픈 **도구가** > **옵션** 메뉴 모음에서.
   1. 엽니다는 **프로젝트 및 솔루션** 노드. 엽니다는 **.NET Core** 탭 합니다.
   1. 확인란 **.NET Core SDK의 미리 보기를 사용 하 여**입니다. **확인**을 선택합니다.
1. 새 프로젝트를 만듭니다.
1. **새 ASP.NET Core 웹 응용 프로그램**을 선택합니다. **새로 만들기**를 선택합니다.
1. 에 이름을 제공 합니다 **프로젝트 이름** 필드입니다. 확인 합니다 **위치** 항목이 올바른 또는 프로젝트의 위치를 지정 합니다. **만들기**를 선택합니다.
1. 했는지 **.NET Core** 하 고 **ASP.NET Core 3.0** 맨 위에 있는 선택 됩니다.
1. 선택 된 **Razor 구성 요소** 템플릿과 선택 **만들기**합니다.
1. **F5** 키를 눌러 앱을 실행합니다.

지금까지 지금까지 첫 번째 구성 요소 Razor 앱을 실행 했습니다!

<!--

# [Visual Studio Code](#tab/visual-studio-code)

Prerequisites:

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.0.md)]

To create your first Razor Components project in Visual Studio Code:

1. Execute the following command from a command shell:

   ```console
   dotnet new razorcomponents -o WebApplication1
   ```

1. Open the *WebApplication1* folder in Visual Studio Code.

1. Add a *.vscode* folder.

1. Add a *tasks.json* file to the *.vscode* folder with the following content:

   [!code-json[](get-started/samples_snapshot/3.x/tasks.json)]

1. Add a *launch.json* file to the *.vscode* folder with the following content:

   [!code-json[](get-started/samples_snapshot/3.x/launch.json)]

1. Execute the app using the Visual Studio Code debugger.

1. In a browser, navigate to `https://localhost:5001`.

Congratulations! You just ran your first Razor Components app!

# [Visual Studio for Mac](#tab/visual-studio-mac)

.NET Core 3.0 will be supported with Visual Studio for Mac version 8.0 or later. Visual Studio for Mac version 8.0 Preview isn't available at this time.

Use the [.NET Core CLI version of this topic](xref:razor-components/get-started?tabs=netcore-cli) on macOS.

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.0.md)]

To create your first project Razor Components project in Visual Studio for Mac:

1. Select **File** > **New Solution** or **New Project**.
1. In the sidebar, select **.NET Core** > **App**.
1. Select **ASP.NET Core Razor Components** and select **Next**.
1. The **Target Framework** defaults to **.NET Core 3.0**. Select **Next**.
1. In the **Project Name** field, enter `WebApplication1`. Select **Create**.
1. Select **Run** > **Run Without Debugging** to run the app *without the debugger*. Running with the debugger isn't supported at this time.

Congratulations! You just ran your first Razor Components app!
-->

# [<a name="net-core-cli"></a>.NET Core CLI](#tab/netcore-cli/)

필수 구성 요소:

* [.NET core SDK 3.0 미리 보기](https://dotnet.microsoft.com/download/dotnet-core/3.0)

1. 명령 셸에서 첫 번째 Razor 구성 요소 프로젝트를 만들려면:

   ```console
   dotnet new razorcomponents -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

1. 브라우저에서 `https://localhost:5001`로 이동합니다.

지금까지 지금까지 첫 번째 구성 요소 Razor 앱을 실행 했습니다!

---

## <a name="razor-components-project"></a>Razor 구성 요소 프로젝트

Razor 구성 요소는 Razor 구문을 사용 하 여 작성 되지만 Razor 페이지 및 MVC 보기와 다르게 컴파일됩니다. 합니다 *.razor* 파일 확장명은 Razor 구성 요소를 지정 하는 데 사용 됩니다. Razor 페이지 및 MVC 뷰를 사용 하 여 계속 합니다 *.cshtml* 파일 확장명입니다.

> [!NOTE]
> 사용 하 여 razor 구성 요소를 작성할 수 있습니다 합니다 *.cshtml* 파일 확장명으로 사용 하 여 Razor 구성 요소 파일과 해당 파일 식별 됩니다는 `_RazorComponentInclude` MSBuild 속성입니다. 예를 들어, Razor 구성 요소 템플릿을 사용 하 여 만든 앱을 지정 하는 모든 *.cshtml* 아래에 있는 파일을 *구성 요소* Razor 구성 요소로 간주 해야 폴더:
>
> ```xml
> <_RazorComponentInclude>Components\**\*.cshtml</_RazorComponentInclude>
> ```

앱을 실행 하는 경우 여러 페이지 세로 막대의 탭에서 사용할 수 있습니다.

* 홈
* 카운터
* 데이터 가져오기

카운터 페이지에서 **Click me** 단추를 선택하여 페이지 새로 고침 없이 카운터를 증분합니다. 웹 페이지의 카운터를 증분하려면 일반적으로 JavaScript를 작성해야 하지만, Razor 구성 요소는 C#를 사용하여 더 나은 접근 방식을 제공합니다.

*WebApplication1/Components/Pages/Counter.razor*:

[!code-cshtml[](get-started/samples_snapshot/3.x/Counter1.razor)]

에 대 한 요청 `/counter` 에 지정 된 대로 브라우저에서을 `@page` 맨 위에 있는 지시문 하면 카운터 구성 요소를 해당 콘텐츠를 렌더링 합니다. 구성 요소는 메모리 내 표현을 유연 하 고 효율적인 방식으로 UI를 업데이트 하는 데 사용할 수 있는 렌더링 트리를 렌더링 합니다.

각 시간 합니다 **Click me** 단추를 선택 합니다.

* `onclick` 이벤트가 발생 합니다.
* `IncrementCount` 메서드가 호출됩니다.
* `currentCount` 증분됩니다.
* 구성 요소를 다시 렌더링 됩니다.

런타임에 이전 내용으로 새 콘텐츠를 비교 하 여 변경 된 내용이만 문서 개체 모델 (DOM)에 적용 됩니다.

HTML과 유사한 구문을 사용 하는 다른 구성 요소는 구성 요소를 추가 합니다. 구성 요소 매개 변수는 특성 또는 자식 콘텐츠를 사용 하 여 지정 됩니다. 예를 들어, 카운터 구성 요소 수에 추가할 앱의 홈 페이지를 추가 하 여를 `<Counter />` 요소 인덱스 구성 요소입니다.

*WebApplication1/Components/Pages/Index.razor*:

[!code-cshtml[](get-started/samples_snapshot/3.x/Index1.razor?highlight=7)]

앱을 실행합니다. 홈 페이지에는 자체 카운터가 있습니다.

매개 변수를 카운터 구성 요소를 추가 하려면 구성 요소의 업데이트 `@functions` 블록:

* 에 대 한 속성을 추가 `IncrementAmount` 데코 레이트 된 `[Parameter]` 특성입니다.
* `currentCount` 값을 늘릴 때 `IncrementAmount`를 사용하도록 `IncrementCount` 메서드를 변경합니다.

*WebApplication1/Components/Pages/Counter.razor*:

[!code-cshtml[](get-started/samples_snapshot/3.x/Counter2.razor?highlight=4,8)]

특성을 사용하여 Home 구성 요소의 `<Counter>` 요소에 `IncrementAmount` 매개 변수를 지정합니다.

*WebApplication1/Components/Pages/Index.razor*:

[!code-cshtml[](get-started/samples_snapshot/3.x/Index2.razor)]

앱을 실행합니다. 홈 페이지에는 각 시간을 10 씩 증가 하는 자체 카운터가 합니다 **Click me** 단추를 선택 합니다.

## <a name="next-steps"></a>다음 단계

<xref:tutorials/first-razor-components-app>