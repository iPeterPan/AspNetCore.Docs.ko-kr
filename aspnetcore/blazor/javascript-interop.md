---
title: JavaScript interop Blazor ASP.NET Core
author: guardrex
description: Blazor apps에서 JavaScript의 .NET 및 .NET 메서드에서 JavaScript 함수를 호출 하는 방법에 대해 알아봅니다.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/18/2019
no-loc:
- Blazor
- SignalR
uid: blazor/javascript-interop
ms.openlocfilehash: 4edef123bc1fe41845b8060b9c3b8e77ffd2969d
ms.sourcegitcommit: c81ef12a1b6e6ac838e5e07042717cf492e6635b
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/29/2020
ms.locfileid: "76885474"
---
# <a name="aspnet-core-opno-locblazor-javascript-interop"></a>JavaScript interop Blazor ASP.NET Core

[Javier Calvarro e](https://github.com/javiercn), [Daniel Roth](https://github.com/danroth27)및 [luke latham 문자](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

Blazor 앱은 JavaScript 코드에서 .NET 및 .NET 메서드의 JavaScript 함수를 호출할 수 있습니다.

[예제 코드 살펴보기 및 다운로드](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/)([다운로드 방법](xref:index#how-to-download-a-sample))

## <a name="invoke-javascript-functions-from-net-methods"></a>.NET 메서드에서 JavaScript 함수 호출

.NET 코드에서 JavaScript 함수를 호출 해야 하는 경우가 있습니다. 예를 들어 JavaScript 호출은 JavaScript 라이브러리의 브라우저 기능 또는 기능을 앱에 노출할 수 있습니다. 이 시나리오를 *JavaScript 상호 운용성* (*JS interop*) 이라고 합니다.

.NET에서 JavaScript를 호출 하려면 `IJSRuntime` 추상화를 사용 합니다. JS interop 호출을 실행 하려면 구성 요소에 `IJSRuntime` 추상화를 삽입 합니다. `InvokeAsync<T>` 메서드는 원하는 수의 JSON serialize 가능 인수와 함께 호출 하려는 JavaScript 함수에 대 한 식별자를 사용 합니다. 함수 식별자는 전역 범위 (`window`)를 기준으로 합니다. `window.someScope.someFunction`를 호출 하려는 경우에는 식별자가 `someScope.someFunction`됩니다. 호출 되기 전에 함수를 등록할 필요가 없습니다. 반환 형식 `T` JSON serializable 이어야 합니다. `T`는 반환 되는 JSON 형식에 가장 잘 매핑되는 .NET 형식과 일치 해야 합니다.

렌더링을 사용 하도록 설정 된 Blazor Server 앱의 경우 초기 렌더링을 수행 하는 동안 JavaScript를 호출할 수 없습니다. JavaScript interop 호출은 브라우저와의 연결이 설정 될 때까지 지연 되어야 합니다. 자세한 내용은 [Blazor 앱이 사전 렌더링 되는 경우 검색](#detect-when-a-blazor-app-is-prerendering) 섹션을 참조 하세요.

다음 예제는 실험적 JavaScript 기반 디코더 인 [Textdecoder](https://developer.mozilla.org/docs/Web/API/TextDecoder)를 기반으로 합니다. 이 예제에서는 C# 메서드에서 JavaScript 함수를 호출 하는 방법을 보여 줍니다. JavaScript 함수는 C# 메서드에서 바이트 배열을 받아 배열을 디코딩하고 표시를 위해 구성 요소에 텍스트를 반환 합니다.

*Wwwroot/index.html* (Blazor WebAssembly) 또는 *Pages/_Host. Cshtml* (Blazor Server)의 `<head>` 요소 내에서 `TextDecoder`를 사용 하 여 전달 된 배열을 디코딩하고 디코딩된 값을 반환 하는 JavaScript 함수를 제공 합니다.

[!code-html[](javascript-interop/samples_snapshot/index-script-convertarray.html)]

앞의 예제에 표시 된 코드와 같은 JavaScript 코드는 스크립트 파일에 대 한 참조를 사용 하 여 JavaScript 파일 ( *.js*)에서 로드할 수도 있습니다.

```html
<script src="exampleJsInterop.js"></script>
```

다음 구성 요소:

* 구성 요소 단추 (**배열 변환**)를 선택한 경우 `JSRuntime`를 사용 하 여 `convertArray` JavaScript 함수를 호출 합니다.
* JavaScript 함수를 호출한 후에는 전달 된 배열이 문자열로 변환 됩니다. 문자열은 표시를 위해 구성 요소로 반환 됩니다.

[!code-razor[](javascript-interop/samples_snapshot/call-js-example.razor?highlight=2,34-35)]

## <a name="use-of-ijsruntime"></a>IJSRuntime 사용

`IJSRuntime` 추상화를 사용 하려면 다음 방법 중 하나를 채택 합니다.

* Razor 구성 요소 (*razor*)에 `IJSRuntime` 추상화를 삽입 합니다.

  [!code-razor[](javascript-interop/samples_snapshot/inject-abstraction.razor?highlight=1)]

  *Wwwroot/index.html* 의 `<head>` 요소 (Blazor Weasembomom) 또는 *Pages/_Host Cshtml* (Blazor Server) 내에서 `handleTickerChanged` JavaScript 함수를 제공 합니다. 함수는 `IJSRuntime.InvokeVoidAsync`를 사용 하 여 호출 되 고 값을 반환 하지 않습니다.

  [!code-html[](javascript-interop/samples_snapshot/index-script-handleTickerChanged1.html)]

* `IJSRuntime` 추상화를 클래스 ( *.cs*)에 삽입 합니다.

  [!code-csharp[](javascript-interop/samples_snapshot/inject-abstraction-class.cs?highlight=5)]

  *Wwwroot/index.html* 의 `<head>` 요소 (Blazor Weasembomom) 또는 *Pages/_Host Cshtml* (Blazor Server) 내에서 `handleTickerChanged` JavaScript 함수를 제공 합니다. 함수는 `JSRuntime.InvokeAsync`를 사용 하 여 호출 되 고 값을 반환 합니다.

  [!code-html[](javascript-interop/samples_snapshot/index-script-handleTickerChanged2.html)]

* [BuildRenderTree](xref:blazor/components#manual-rendertreebuilder-logic)를 사용 하 여 동적 콘텐츠를 생성 하려면 `[Inject]` 특성을 사용 합니다.

  ```razor
  [Inject]
  IJSRuntime JSRuntime { get; set; }
  ```

이 항목과 함께 제공 되는 클라이언트 쪽 샘플 앱에서는 사용자 입력을 받고 환영 메시지를 표시 하기 위해 DOM과 상호 작용 하는 두 가지 JavaScript 함수를 앱에서 사용할 수 있습니다.

* `showPrompt` &ndash;에서 사용자 입력을 수락 하는 프롬프트 (사용자 이름)를 생성 하 고 이름을 호출자에 게 반환 합니다.
* `displayWelcome` &ndash;는 `welcome``id`를 사용 하 여 호출자의 시작 메시지를 DOM 개체에 할당 합니다.

*wwwroot/exampleJsInterop.js*:

[!code-javascript[](./common/samples/3.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=2-7)]

JavaScript 파일을 참조 하는 `<script>` 태그를 *wwwroot/index.html* 파일 (Blazor Weasembmboms) 또는 *Pages/_Host cshtml* 파일 (Blazor Server)에 저장 합니다.

*wwwroot/index.html* (Blazor):

[!code-html[](./common/samples/3.x/BlazorWebAssemblySample/wwwroot/index.html?highlight=22)]

*Pages/_Host* (Blazor 서버):

[!code-cshtml[](./common/samples/3.x/BlazorServerSample/Pages/_Host.cshtml?highlight=35)]

`<script>` 태그를 동적으로 업데이트할 수 없으므로 구성 요소 파일에 `<script>` 태그를 넣지 마세요.

.NET 메서드는 `IJSRuntime.InvokeAsync<T>`를 호출 하 여 *exampleJsInterop* 파일의 JavaScript 함수를 사용 하 여 상호 운용 합니다.

`IJSRuntime` 추상화는 Blazor 서버 시나리오를 허용 하는 비동기입니다. 앱이 Blazor Weasembomapp이 고 JavaScript 함수를 동기적으로 호출 하려는 경우에는 다운 캐스트를 `IJSInProcessRuntime` 하 고 대신 `Invoke<T>`를 호출 합니다. 대부분의 JS interop 라이브러리는 비동기 Api를 사용 하 여 모든 시나리오에서 라이브러리를 사용할 수 있도록 하는 것이 좋습니다.

샘플 앱에는 JS interop를 시연 하는 구성 요소가 포함 되어 있습니다. 구성 요소:

* JavaScript 프롬프트를 통해 사용자 입력을 받습니다.
* 처리를 위해 구성 요소에 텍스트를 반환 합니다.
* 는 DOM과 상호 작용 하 여 시작 메시지를 표시 하는 두 번째 JavaScript 함수를 호출 합니다.

*Pages/JSInterop*:

```razor
@page "/JSInterop"
@using BlazorSample.JsInteropClasses
@inject IJSRuntime JSRuntime

<h1>JavaScript Interop</h1>

<h2>Invoke JavaScript functions from .NET methods</h2>

<button type="button" class="btn btn-primary" @onclick="TriggerJsPrompt">
    Trigger JavaScript Prompt
</button>

<h3 id="welcome" style="color:green;font-style:italic"></h3>

@code {
    public async Task TriggerJsPrompt()
    {
        // showPrompt is implemented in wwwroot/exampleJsInterop.js
        var name = await JSRuntime.InvokeAsync<string>(
                "exampleJsFunctions.showPrompt",
                "What's your name?");
        // displayWelcome is implemented in wwwroot/exampleJsInterop.js
        await JSRuntime.InvokeVoidAsync(
                "exampleJsFunctions.displayWelcome",
                $"Hello {name}! Welcome to Blazor!");
    }
}
```

1. 구성 요소의 **트리거 Javascript 프롬프트** 단추를 선택 하 여 `TriggerJsPrompt`를 실행 하면 *wwwroot/exampleJsInterop* 파일에 제공 된 javascript `showPrompt` 함수를 호출 합니다.
1. `showPrompt` 함수는 사용자 입력 (사용자 이름)을 허용 하며,이는 HTML로 인코딩하고 구성 요소로 반환 됩니다. 이 구성 요소는 `name`지역 변수에 사용자 이름을 저장 합니다.
1. `name`에 저장 된 문자열은 환영 메시지에 통합 됩니다 .이 메시지는 환영 메시지를 머리글 태그에 렌더링 하는 `displayWelcome`JavaScript 함수에 전달 됩니다.

## <a name="call-a-void-javascript-function"></a>Void JavaScript 함수 호출

[Void (0)/void 0](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/void) 또는 [undefined](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/undefined) 를 반환 하는 JavaScript 함수는 `IJSRuntime.InvokeVoidAsync`를 사용 하 여 호출 됩니다.

## <a name="detect-when-a-opno-locblazor-app-is-prerendering"></a>Blazor 앱이 사전 렌더링 되는 경우 검색
 
[!INCLUDE[](~/includes/blazor-prerendering.md)]

## <a name="capture-references-to-elements"></a>요소에 대 한 참조 캡처

일부 JS interop 시나리오에서는 HTML 요소에 대 한 참조가 필요 합니다. 예를 들어, UI 라이브러리에서 초기화를 위해 요소 참조를 요구 하거나, `focus` 또는 `play`와 같은 요소에서 명령 같은 Api를 호출 해야 할 수 있습니다.

다음 방법을 사용 하 여 구성 요소의 HTML 요소에 대 한 참조를 캡처합니다.

* HTML 요소에 `@ref` 특성을 추가 합니다.
* `@ref` 특성의 값과 이름이 일치 하는 `ElementReference` 형식의 필드를 정의 합니다.

다음 예에서는 `username` `<input>` 요소에 대 한 참조를 캡처하는 방법을 보여 줍니다.

```razor
<input @ref="username" ... />

@code {
    ElementReference username;
}
```

> [!WARNING]
> Blazor와 상호 작용 하지 않는 빈 요소의 내용은 요소 참조를 사용 하 여 변경할 수 있습니다. 이 시나리오는 타사 API가 요소에 콘텐츠를 제공 하는 경우에 유용 합니다. Blazor는 요소와 상호 작용 하지 않으므로 요소와 DOM의 Blazor표시 간에 충돌이 발생할 가능성이 없습니다.
>
> 다음 예제에서는 Blazor DOM과 상호 작용 하 여이 요소의 목록 항목 (`<li>`)을 채우기 때문에 순서가 지정 되지 않은 목록 (`ul`)의 *콘텐츠를 변경할 수 있습니다* .
>
> ```razor
> <ul ref="MyList">
>     @foreach (var item in Todos)
>     {
>         <li>@item.Text</li>
>     }
> </ul>
> ```
>
> JS interop에서 요소 `MyList`의 내용을 capture 하 고 Blazor 요소에 차이을 적용 하려고 하면 차이는 DOM과 일치 하지 않습니다.

.NET 코드와 관련 하 여 `ElementReference` 불투명 핸들입니다. `ElementReference`로 수행할 수 있는 *유일한* 작업은 JS interop를 통해 JavaScript 코드에 전달 하는 것입니다. 이렇게 하면 JavaScript 쪽 코드가 일반적인 DOM Api와 함께 사용할 수 있는 `HTMLElement` 인스턴스를 수신 합니다.

예를 들어 다음 코드는 요소에 포커스를 설정할 수 있도록 하는 .NET 확장 메서드를 정의 합니다.

*exampleJsInterop.js*:

```javascript
window.exampleJsFunctions = {
  focusElement : function (element) {
    element.focus();
  }
}
```

값을 반환 하지 않는 JavaScript 함수를 호출 하려면 `IJSRuntime.InvokeVoidAsync`을 사용 합니다. 다음 코드는 캡처된 `ElementReference`로 이전 JavaScript 함수를 호출 하 여 사용자 이름 입력에 포커스를 설정 합니다.

[!code-razor[](javascript-interop/samples_snapshot/component1.razor?highlight=1,3,11-12)]

확장 메서드를 사용 하려면 `IJSRuntime` 인스턴스를 수신 하는 정적 확장 메서드를 만듭니다.

```csharp
public static async Task Focus(this ElementReference elementRef, IJSRuntime jsRuntime)
{
    await jsRuntime.InvokeVoidAsync(
        "exampleJsFunctions.focusElement", elementRef);
}
```

`Focus` 메서드는 개체에서 직접 호출 됩니다. 다음 예에서는 `Focus` 메서드를 `JsInteropClasses` 네임 스페이스에서 사용할 수 있다고 가정 합니다.

[!code-razor[](javascript-interop/samples_snapshot/component2.razor?highlight=1-4,12)]

> [!IMPORTANT]
> `username` 변수는 구성 요소가 렌더링 된 후에만 채워집니다. JavaScript 코드에 채워지지 않은 `ElementReference` 전달 되 면 JavaScript 코드는 `null`값을 받습니다. 구성 요소에서 요소에 대 한 초기 포커스를 설정 하기 위해 렌더링을 완료 한 후 요소 참조를 조작 하려면 [OnAfterRenderAsync 또는 OnAfterRender 구성 요소 수명 주기 메서드](xref:blazor/lifecycle#after-component-render)를 사용 합니다.

제네릭 형식으로 작업 하 고 값을 반환 하는 경우 [\<t >](xref:System.Threading.Tasks.ValueTask`1)를 사용 합니다.

```csharp
public static ValueTask<T> GenericMethod<T>(this ElementReference elementRef, 
    IJSRuntime jsRuntime)
{
    return jsRuntime.InvokeAsync<T>(
        "exampleJsFunctions.doSomethingGeneric", elementRef);
}
```

`GenericMethod`는 형식을 사용 하 여 개체에서 직접 호출 됩니다. 다음 예에서는 `GenericMethod` `JsInteropClasses` 네임 스페이스에서 사용할 수 있다고 가정 합니다.

[!code-razor[](javascript-interop/samples_snapshot/component3.razor?highlight=17)]

## <a name="invoke-net-methods-from-javascript-functions"></a>JavaScript 함수에서 .NET 메서드 호출

### <a name="static-net-method-call"></a>정적 .NET 메서드 호출

JavaScript에서 정적 .NET 메서드를 호출 하려면 `DotNet.invokeMethod` 또는 `DotNet.invokeMethodAsync` 함수를 사용 합니다. 호출할 정적 메서드의 식별자, 함수를 포함 하는 어셈블리의 이름 및 인수를 전달 합니다. 비동기 버전은 Blazor 서버 시나리오를 지원 하기 위해 선호 됩니다. JavaScript에서 .NET 메서드를 호출 하려면 .NET 메서드가 public, static 및 `[JSInvokable]` 특성을 가져야 합니다. 기본적으로 메서드 식별자는 메서드 이름 이지만 `JSInvokableAttribute` 생성자를 사용 하 여 다른 식별자를 지정할 수 있습니다. Open 제네릭 메서드를 호출 하는 것은 현재 지원 되지 않습니다.

샘플 앱에는 `int`C# s의 배열을 반환 하는 메서드가 포함 되어 있습니다. `JSInvokable` 특성이 메서드에 적용 됩니다.

*Pages/JsInterop*:

```razor
<button type="button" class="btn btn-primary"
        onclick="exampleJsFunctions.returnArrayAsyncJs()">
    Trigger .NET static method ReturnArrayAsync
</button>

@code {
    [JSInvokable]
    public static Task<int[]> ReturnArrayAsync()
    {
        return Task.FromResult(new int[] { 1, 2, 3 });
    }
}
```

클라이언트에 제공 된 JavaScript는 .Net C# 메서드를 호출 합니다.

*wwwroot/exampleJsInterop.js*:

[!code-javascript[](./common/samples/3.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=8-14)]

**.Net 정적 메서드 ReturnArrayAsync 트리거** 단추를 선택한 경우 브라우저의 웹 개발자 도구에서 콘솔 출력을 검사 합니다.

콘솔 출력은 다음과 같습니다.

```console
Array(4) [ 1, 2, 3, 4 ]
```

네 번째 배열 값은 `ReturnArrayAsync`에서 반환 된 배열 (`data.push(4);`)으로 푸시됩니다.

### <a name="instance-method-call"></a>인스턴스 메서드 호출

JavaScript에서 .NET 인스턴스 메서드를 호출할 수도 있습니다. JavaScript에서 .NET 인스턴스 메서드를 호출 하려면 다음을 수행 합니다.

* `DotNetObjectReference` 인스턴스에 래핑하여 .NET 인스턴스를 JavaScript에 전달 합니다. .NET 인스턴스는 JavaScript에 대 한 참조로 전달 됩니다.
* `invokeMethod` 또는 `invokeMethodAsync` 함수를 사용 하 여 인스턴스에서 .NET 인스턴스 메서드를 호출 합니다. JavaScript에서 다른 .NET 메서드를 호출할 때 .NET 인스턴스도 인수로 전달 될 수도 있습니다.

> [!NOTE]
> 샘플 앱은 클라이언트 쪽 콘솔에 메시지를 기록 합니다. 샘플 앱에서 보여 주는 다음 예제에서는 브라우저의 개발자 도구에서 브라우저의 콘솔 출력을 검사 합니다.

**HelloHelper .net 인스턴스 메서드 SayHello** 단추를 선택 하면 `ExampleJsInterop.CallHelloHelperSayHello`가 호출 되 고 메서드에 `Blazor`이름이 전달 됩니다.

*Pages/JsInterop*:

```razor
<button type="button" class="btn btn-primary" @onclick="TriggerNetInstanceMethod">
    Trigger .NET instance method HelloHelper.SayHello
</button>

@code {
    public async Task TriggerNetInstanceMethod()
    {
        var exampleJsInterop = new ExampleJsInterop(JSRuntime);
        await exampleJsInterop.CallHelloHelperSayHello("Blazor");
    }
}
```

`CallHelloHelperSayHello`는 `HelloHelper`의 새 인스턴스와 `sayHello` JavaScript 함수를 호출 합니다.

*JsInteropClasses/ExampleJsInterop.cs*:

[!code-csharp[](./common/samples/3.x/BlazorWebAssemblySample/JsInteropClasses/ExampleJsInterop.cs?name=snippet1&highlight=10-16)]

*wwwroot/exampleJsInterop.js*:

[!code-javascript[](./common/samples/3.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=15-18)]

이 이름은 `HelloHelper.Name` 속성을 설정 하는 `HelloHelper`의 생성자에 전달 됩니다. JavaScript 함수 `sayHello` 실행 되 면 `HelloHelper.SayHello` JavaScript 함수를 통해 콘솔에 기록 되는 `Hello, {Name}!` 메시지를 반환 합니다.

*JsInteropClasses/HelloHelper.cs*:

[!code-csharp[](./common/samples/3.x/BlazorWebAssemblySample/JsInteropClasses/HelloHelper.cs?name=snippet1&highlight=5,10-11)]

브라우저의 웹 개발자 도구에 있는 콘솔 출력:

```console
Hello, Blazor!
```

## <a name="share-interop-code-in-a-class-library"></a>클래스 라이브러리의 interop 코드 공유

JS interop 코드를 클래스 라이브러리에 포함 하 여 NuGet 패키지의 코드를 공유할 수 있습니다.

클래스 라이브러리는 빌드된 어셈블리에 포함 된 JavaScript 리소스를 처리 합니다. JavaScript 파일은 *wwwroot* 폴더에 배치 됩니다. 도구는 라이브러리가 빌드될 때 리소스를 포함 하는 작업을 담당 합니다.

빌드된 NuGet 패키지는 NuGet 패키지를 참조 하는 것과 동일한 방식으로 앱의 프로젝트 파일에서 참조 됩니다. 패키지가 복원 된 후에는 앱 코드가 JavaScript를로 호출할 수 있습니다 C#.

자세한 내용은 <xref:blazor/class-libraries>를 참조하세요.

## <a name="harden-js-interop-calls"></a>JS interop 호출 강화

JS interop는 네트워킹 오류로 인해 실패할 수 있으며 신뢰할 수 없는 것으로 처리 되어야 합니다. 기본적으로 Blazor Server 앱은 1 분 후 서버에서 JS interop 호출을 시간 제한 합니다. 앱에서 10 초 등의 적극적 시간 제한을 허용할 수 있는 경우 다음 방법 중 하나를 사용 하 여 시간 제한을 설정 합니다.

* `Startup.ConfigureServices`에서 전역적으로 시간 제한을 지정 합니다.

  ```csharp
  services.AddServerSideBlazor(
      options => options.JSInteropDefaultCallTimeout = TimeSpan.FromSeconds({SECONDS}));
  ```

* 구성 요소 코드에서 호출 마다 단일 호출에서 제한 시간을 지정할 수 있습니다.

  ```csharp
  var result = await JSRuntime.InvokeAsync<string>("MyJSOperation", 
      TimeSpan.FromSeconds({SECONDS}), new[] { "Arg1" });
  ```

리소스 소모에 대 한 자세한 내용은 <xref:security/blazor/server>를 참조 하세요.

## <a name="additional-resources"></a>추가 자료

* [InteropComponent 예 (dotnet/AspNetCore GitHub 리포지토리, 3.1 릴리스 분기)](https://github.com/dotnet/AspNetCore/blob/release/3.1/src/Components/test/testassets/BasicTestApp/InteropComponent.razor)
