---
title: ASP.NET Core의 보기 구성 요소
author: rick-anderson
description: ASP.NET Core에서 보기 구성 요소가 사용되는 방법 및 이를 앱에 추가하는 방법을 알아봅니다.
ms.author: riande
ms.custom: mvc
ms.date: 12/18/2019
uid: mvc/views/view-components
ms.openlocfilehash: 8df070b580b6125db6616b33a81f3ffebad69f3a
ms.sourcegitcommit: 7dfe6cc8408ac6a4549c29ca57b0c67ec4baa8de
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 01/09/2020
ms.locfileid: "75828921"
---
# <a name="view-components-in-aspnet-core"></a>ASP.NET Core의 보기 구성 요소

작성자: [Rick Anderson](https://twitter.com/RickAndMSFT)

[예제 코드 살펴보기 및 다운로드](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/views/view-components/sample) ([다운로드 방법](xref:index#how-to-download-a-sample))

## <a name="view-components"></a>보기 구성 요소

보기 구성 요소는 부분 보기와 유사하지만 훨씬 강력합니다. 보기 구성 요소는 모델 바인딩을 사용하지 않으며 호출 시 제공하는 데이터에만 의존합니다. 이 문서는 컨트롤러 및 보기를 사용하여 작성되었지만, 보기 구성 요소는 Razor Pages와 함께 사용할 수도 있습니다.

보기 구성 요소는 다음과 같은 특징을 갖고 있습니다.

* 전체 응답보다는 청크를 렌더링합니다.
* 컨트롤러 및 보기 간에서 발견할 수 있는 것과 동일한 문제의 분리 및 테스트 가능성의 이점을 포함합니다.
* 매개 변수 및 비즈니스 논리를 포함할 수 있습니다.
* 일반적으로 레이아웃 페이지에서 호출됩니다.

보기 구성 요소는 다음과 같이 부분 보기로는 너무 복잡한 재사용 가능한 렌더링 논리를 갖는 모든 곳에서 사용할 수 있습니다.

* 동적 탐색 메뉴
* 태그 클라우드(여기서는 데이터베이스를 쿼리합니다.)
* 로그인 패널
* 쇼핑 카트
* 최근에 게시된 문서
* 일반적인 블로그의 사이드바 콘텐츠
* 모든 페이지에 렌더링되고 사용자의 로그인 상태에 따라 로그아웃 또는 로그인 링크를 표시하는 로그인 패널

보기 구성 요소는 클래스(일반적으로 [ViewComponent](/dotnet/api/microsoft.aspnetcore.mvc.viewcomponent)에서 파생됨)와 보기 구성 요소가 반환하는 결과(일반적으로 보기)의 두 부분으로 구성됩니다. 컨트롤러와 마찬가지로 보기 구성 요소는 POCO일 수 있지만 대부분의 개발자는 `ViewComponent`에서 파생하여 사용 가능한 메서드와 속성을 활용하려고 합니다.

보기 구성 요소가 앱의 사양을 충족하는지 고려할 때 Razor 구성 요소를 대신 사용하는 방안도 고려해보세요. Razor 구성 요소 역시 태그와 C# 코드를 결합하여 다시 사용할 수 있는 UI 단위를 생성합니다. Razor 구성 요소는 클라이언트 쪽 UI 논리 및 컴퍼지션을 제공할 때 개발자 생산성을 위해 설계되었습니다. 자세한 내용은 <xref:blazor/components>를 참조하세요.

## <a name="creating-a-view-component"></a>보기 구성 요소 만들기

이 섹션에서는 보기 구성 요소를 만들기 위한 높은 수준의 요구 사항이 포함되어 있습니다. 이 문서의 뒷부분에서는 각 단계를 자세히 검토하고 보기 구성 요소를 만들어봅니다.

### <a name="the-view-component-class"></a>보기 구성 요소 클래스

다음 중 한 가지 방법을 사용하여 보기 구성 요소 클래스를 만들 수 있습니다.

* *ViewComponent*에서 파생
* `[ViewComponent]` 특성을 사용하여 클래스를 데코레이팅하거나 `[ViewComponent]` 특성이 사용된 클래스에서 파생
* 이름이 *ViewComponent* 접미사로 끝나는 클래스 만들기

컨트롤러와 마찬가지로 보기 구성 요소는 공용, 비중첩 및 비추상 클래스이어야 합니다. 보기 구성 요소의 이름은 "ViewComponent" 접미사가 제거된 클래스 이름입니다. 또한 `ViewComponentAttribute.Name` 속성을 사용하여 명시적으로 지정할 수도 있습니다.

보기 구성 요소 클래스는 다음과 같은 특징을 갖고 있습니다.

* 생성자 [종속성 주입](../../fundamentals/dependency-injection.md)을 완벽하게 지원합니다.

* 컨트롤러 수명 주기를 따르지 않습니다. 즉, 뷰 구성 요소에 [필터](../controllers/filters.md)를 사용할 수 없습니다.

### <a name="view-component-methods"></a>보기 구성 요소 메서드

보기 구성 요소는 `Task<IViewComponentResult>`를 반환하는 `InvokeAsync` 메서드 또는 `IViewComponentResult`를 반환하는 동기 `Invoke` 메서드에서 해당 논리를 정의합니다. 매개 변수는 모델 바인딩이 아니라 보기 구성 요소 호출에서 직접 가져옵니다. 보기 구성 요소는 요청을 직접 처리하지 않습니다. 일반적으로 보기 구성 요소는 모델을 초기화하고 `View` 메서드를 호출하여 보기에 전달합니다. 요약해보면 보기 구성 요소 메서드는 다음과 같습니다.

* `Task<IViewComponentResult>`를 반환하는 `InvokeAsync` 메서드 또는 `IViewComponentResult`를 반환하는 동기 `Invoke` 메서드를 정의합니다.
* 일반적으로 모델을 초기화하고 `ViewComponent` `View` 메서드를 호출하여 이를 보기에 전달합니다.
* 매개 변수는 HTTP가 아닌 호출 메서드에서 가져옵니다. 모델 바인딩이 없습니다.
* HTTP 엔드포인트로 직접 연결할 수 없습니다. (일반적으로 보기의) 코드에서 호출됩니다. 보기 구성 요소는 요청을 처리하지 않습니다.
* 현재 HTTP 요청의 세부 정보가 아닌 시그니처에 오버로드됩니다.

### <a name="view-search-path"></a>보기 검색 경로

런타임은 다음 경로에서 보기를 검색합니다.

* /Views/{Controller Name}/Components/{View Component Name}/{View Name}
* /Views/Shared/Components/{View Component Name}/{View Name}
* /Pages/Shared/Components/{View Component Name}/{View Name}

검색 경로는 컨트롤러 + 보기 및 Razor Pages를 사용하는 프로젝트에 적용됩니다.

보기 구성 요소에 대한 기본 보기 이름은 *Default*로 이는 일반적으로 보기 파일의 이름이 *Default.cshtml*로 지정됨을 의미합니다. 보기 구성 요소 결과를 만들거나 `View` 메서드를 호출할 때 다른 보기 이름을 지정할 수 있습니다.

보기 파일 이름을 *Default.cshtml*로 지정하고 *Views/Shared/Components/{View Component Name}/{View Name}* 경로를 사용하는 것이 좋습니다. 예제에 사용된 `PriorityList` 보기 구성 요소는 보기 구성 요소 보기로 *Views/Shared/Components/PriorityList/Default.cshtml*을 사용합니다.

### <a name="customize-the-view-search-path"></a>보기 검색 경로 사용자 지정

보기 검색 경로를 사용자 지정하려면 Razor의 <xref:Microsoft.AspNetCore.Mvc.Razor.RazorViewEngineOptions.ViewLocationFormats> 컬렉션을 수정합니다. 예를 들어 "/Components/{View Component Name}/{View Name}" 경로 내에서 보기를 검색하려면 컬렉션에 새 항목을 추가합니다.

[!code-cs[](view-components/samples_snapshot/2.x/Startup.cs?name=snippet_ViewLocationFormats&highlight=4)]

이전 코드에서 "{0}" 자리 표시자는 "Components/{View Component Name}/{View Name}" 경로를 나타냅니다.

## <a name="invoking-a-view-component"></a>보기 구성 요소 호출

보기 구성 요소를 사용하려면 보기에서 다음을 호출합니다.

```cshtml
@await Component.InvokeAsync("Name of view component", {Anonymous Type Containing Parameters})
```

매개 변수는 `InvokeAsync` 메서드에 전달됩니다. 이 문서에서 개발하는 `PriorityList` 보기 구성 요소는 *Views/ToDo/Index.cshtml* 보기 파일에서 호출됩니다. 다음에서 `InvokeAsync` 메서드는 두 매개 변수를 사용하여 호출됩니다.

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexFinal.cshtml?range=35)]

::: moniker range=">= aspnetcore-1.1"

## <a name="invoking-a-view-component-as-a-tag-helper"></a>태그 도우미로 보기 구성 요소 호출

ASP.NET Core 1.1 이상에서는 [태그 도우미](xref:mvc/views/tag-helpers/intro)로 보기 구성 요소를 호출할 수 있습니다.

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexTagHelper.cshtml?range=37-38)]

태그 도우미에 대한 파스칼식 클래스 및 메서드 매개 변수는 [케밥식](https://stackoverflow.com/questions/11273282/whats-the-name-for-dash-separated-case/12273101)으로 번역됩니다. 보기 구성 요소를 호출하는 태그 도우미는 `<vc></vc>` 요소를 사용합니다. 보기 구성 요소는 다음과 같이 지정됩니다.

```cshtml
<vc:[view-component-name]
  parameter1="parameter1 value"
  parameter2="parameter2 value">
</vc:[view-component-name]>
```

보기 구성 요소를 태그 도우미로 사용하려면 `@addTagHelper` 지시문을 사용하여 보기 구성 요소가 포함된 어셈블리를 등록합니다. 보기 구성 요소가 `MyWebApp`이라는 어셈블리에 있는 경우 다음 지시문을 *_ViewImports.cshtml* 파일에 추가합니다.

```cshtml
@addTagHelper *, MyWebApp
```

보기 구성 요소를 참조하는 모든 파일에 보기 구성 요소를 태그 도우미로 등록할 수 있습니다. 태그 도우미를 등록하는 방법에 대한 자세한 내용은 [태그 도우미 범위 관리](xref:mvc/views/tag-helpers/intro#managing-tag-helper-scope)를 참조하세요.

이 자습서에 사용된 `InvokeAsync` 메서드:

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexFinal.cshtml?range=35)]

태그 도우미 태그:

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexTagHelper.cshtml?range=37-38)]

위의 예제에서 `PriorityList` 보기 구성 요소는 `priority-list`가 됩니다. 보기 구성 요소에 대한 매개 변수는 케밥식의 특성으로 전달됩니다.

::: moniker-end

### <a name="invoking-a-view-component-directly-from-a-controller"></a>컨트롤러에서 보기 구성 요소 직접 호출

일반적으로 보기 구성 요소는 보기에서 호출되지만 컨트롤러 메서드에서 직접 호출할 수도 있습니다. 보기 구성 요소는 컨트롤러 같은 엔드포인트를 정의하지 않지만 `ViewComponentResult`의 내용을 반환하는 컨트롤러 동작을 쉽게 구현할 수 있습니다.

이 예제에서는 보기 구성 요소가 컨트롤러에서 직접 호출됩니다.

[!code-csharp[](view-components/sample/ViewCompFinal/Controllers/ToDoController.cs?name=snippet_IndexVC)]

## <a name="walkthrough-creating-a-simple-view-component"></a>연습: 간단한 뷰 구성 요소 만들기

시작 코드를 [다운로드](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/views/view-components/sample), 빌드 및 테스트합니다. *ToDo* 항목의 목록을 표시하는 `ToDo` 컨트롤러가 포함된 간단한 프로젝트입니다.

![ToDo 목록](view-components/_static/2dos.png)

### <a name="add-a-viewcomponent-class"></a>ViewComponent 클래스 추가

*ViewComponents* 폴더를 만들고 다음 `PriorityListViewComponent` 클래스를 추가합니다.

[!code-csharp[](view-components/sample/ViewCompFinal/ViewComponents/PriorityListViewComponent1.cs?name=snippet1)]

코드에 대한 참고 사항:

* 보기 구성 요소 클래스는 프로젝트의 **모든** 폴더에 포함될 수 있습니다.
* 클래스 이름 PriorityList**ViewComponent**는 **ViewComponent** 접미사로 끝나기 때문에 런타임이 보기에서 클래스 구성 요소를 참조할 때 "PriorityList" 문자열을 사용합니다. 나중에 보다 자세히 설명합니다.
* `[ViewComponent]` 특성은 보기 구성 요소를 참조하는 데 사용되는 이름을 변경할 수 있습니다. 예를 들어 클래스의 이름을 `XYZ`로 지정하고 `ViewComponent` 특성을 적용할 수 있습니다.

  ```csharp
  [ViewComponent(Name = "PriorityList")]
     public class XYZ : ViewComponent
     ```

* 위의 `[ViewComponent]` 특성은 구성 요소와 연관된 보기를 찾을 때 `PriorityList` 이름을 사용하고, 보기에서 클래스 구성 요소를 참조할 때 "PriorityList" 문자열을 사용하도록 보기 구성 요소 선택기에게 지시합니다. 나중에 보다 자세히 설명합니다.
* 이 구성 요소에서는 [종속성 주입](../../fundamentals/dependency-injection.md)을 사용하여 데이터 컨텍스트를 사용할 수 있도록 합니다.
* `InvokeAsync`는 보기에서 호출할 수 있는 메서드를 노출하며 임의의 개수의 인수를 사용할 수 있습니다.
* `InvokeAsync` 메서드는 `isDone` 및 `maxPriority` 매개 변수를 만족하는 `ToDo` 항목 집합을 반환합니다.

### <a name="create-the-view-component-razor-view"></a>보기 구성 요소 Razor 보기 만들기

* *Views/Shared/Components* 폴더를 만듭니다. 이 폴더의 이름은 *Components*로 **지정되어야만** 합니다.

* *Views/Shared/Components/PriorityList* 폴더를 만듭니다. 이 폴더 이름은 보기 구성 요소 클래스의 이름 또는 클래스 이름에서 접미사를 뺀 이름과 일치해야 합니다(규칙을 준수하고 클래스 이름에 *ViewComponent* 접미사를 사용한 경우). `ViewComponent` 특성을 사용한 경우 클래스 이름은 특성 지정과 일치해야 합니다.

* *Views/Shared/Components/PriorityList/Default.cshtml* Razor 보기를 만듭니다.


  [!code-cshtml[](view-components/sample/ViewCompFinal/Views/Shared/Components/PriorityList/Default1.cshtml)]

   Razor 뷰는 `TodoItem` 목록을 가져와 표시합니다. 뷰 구성 요소 `InvokeAsync` 메서드가 뷰의 이름을 전달하지 않은 경우(샘플에서처럼) 규칙에 따라 뷰 이름으로 *Default*가 사용됩니다. 자습서의 뒷부분에서 뷰 이름을 전달하는 방법을 보여 줍니다. 특정 컨트롤러에 대한 기본 스타일 지정을 재정의하려면 컨트롤러 관련 뷰 폴더에 뷰를 추가합니다(예: *Views/ToDo/Components/PriorityList/Default.cshtml)* .

    보기 구성 요소가 컨트롤러에 관한 것이면 컨트롤러 관련 폴더에 추가할 수 있습니다(*Views/ToDo/Components/PriorityList/Default.cshtml*).

* 우선 순위 목록 구성 요소에 대한 호출을 포함하는 `div`를 *Views/ToDo/index.cshtml* 파일 맨 아래에 추가합니다.

    [!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexFirst.cshtml?range=34-38)]

`@await Component.InvokeAsync` 태그는 호출하는 보기 구성 요소에 대한 구문을 보여 줍니다. 첫 번째 인수는 실행하거나 호출하려는 구성 요소의 이름입니다. 후속 매개 변수는 구성 요소에 전달됩니다. `InvokeAsync`는 임의 개수의 인수를 사용할 수 있습니다.

앱을 테스트합니다. 다음 이미지는 ToDo 목록 및 우선 순위 항목을 보여 줍니다.

![todo 목록 및 우선 순위 항목](view-components/_static/pi.png)

또한 컨트롤러에서 보기 구성 요소를 직접 호출할 수도 있습니다.

[!code-csharp[](view-components/sample/ViewCompFinal/Controllers/ToDoController.cs?name=snippet_IndexVC)]

![IndexVC 작업에서 우선 순위 항목](view-components/_static/indexvc.png)

### <a name="specifying-a-view-name"></a>보기 이름 지정

복잡한 보기 구성 요소에서는 조건에 따라 기본값이 아닌 보기를 지정해야 할 수도 있습니다. 다음 코드는 `InvokeAsync` 메서드에서 "PVC" 보기를 지정하는 방법을 보여 줍니다. `PriorityListViewComponent` 클래스에서 `InvokeAsync` 메서드를 수정합니다.

[!code-csharp[](../../mvc/views/view-components/sample/ViewCompFinal/ViewComponents/PriorityListViewComponentFinal.cs?highlight=4,5,6,7,8,9&range=28-39)]

*Views/Shared/Components/PriorityList/Default.cshtml* 파일을 *Views/Shared/Components/PriorityList/PVC.cshtml*이라는 보기로 복사합니다. PVC 보기가 사용되었음을 나타내는 제목을 추가합니다.

[!code-cshtml[](../../mvc/views/view-components/sample/ViewCompFinal/Views/Shared/Components/PriorityList/PVC.cshtml?highlight=3)]

*Views/ToDo/Index.cshtml*을 수정합니다.

<!-- Views/ToDo/Index.cshtml is never imported, so change to test tutorial -->

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexFinal.cshtml?range=35)]

앱을 실행하고 PVC 보기를 확인합니다.

![우선 순위 보기 구성 요소](view-components/_static/pvc.png)

PVC 보기가 렌더링되지 않는다면 우선 순위가 4 이상인 보기 구성 요소를 호출하고 있는지 확인해봅니다.

### <a name="examine-the-view-path"></a>보기 경로 검토

* 우선 순위 보기가 반환되지 않도록 우선 순위 매개 변수를 3 이하로 변경합니다.
* 임시로 *Views/ToDo/Components/PriorityList/Default.cshtml*의 이름을 *1Default.cshtml*로 변경합니다.
* 앱을 테스트하면 다음과 같은 오류가 발생합니다.

   ```
   An unhandled exception occurred while processing the request.
   InvalidOperationException: The view 'Components/PriorityList/Default' wasn't found. The following locations were searched:
   /Views/ToDo/Components/PriorityList/Default.cshtml
   /Views/Shared/Components/PriorityList/Default.cshtml
   EnsureSuccessful
   ```

* *Views/ToDo/Components/PriorityList/1Default.cshtml*을 *Views/Shared/Components/PriorityList/Default.cshtml*에 복사합니다.
* *Shared* ToDo 보기 구성 요소 보기에 일부 태그를 추가하여 보기가 *Shared* 폴더에 있음을 표시합니다.
* **Shared** 구성 요소 보기를 테스트합니다.

![공유 구성 요소 보기가 있는 ToDo 출력](view-components/_static/shared.png)

### <a name="avoiding-hard-coded-strings"></a>하드 코드된 문자열 방지

컴파일 시간 안전성을 원하는 경우 하드 코드된 보기 구성 요소 이름을 클래스 이름으로 바꿀 수 있습니다. "ViewComponent" 접미사 없이 보기 구성 요소를 만듭니다.

[!code-csharp[](../../mvc/views/view-components/sample/ViewCompFinal/ViewComponents/PriorityList.cs?highlight=10&range=5-35)]

`using` 문을 Razor 보기 파일에 추가하고 `nameof` 연산자를 사용합니다.

[!code-cshtml[](view-components/sample/ViewCompFinal/Views/ToDo/IndexNameof.cshtml?range=1-6,35-)]

## <a name="perform-synchronous-work"></a>동기 작업 수행

비동기 작업을 수행할 필요가 없는 경우 프레임워크에서 동기 `Invoke` 메서드 호출을 처리합니다. 다음 메서드는 동기 `Invoke` 보기 구성 요소를 만듭니다.

```csharp
public class PriorityList : ViewComponent
{
    public IViewComponentResult Invoke(int maxPriority, bool isDone)
    {
        var items = new List<string> { $"maxPriority: {maxPriority}", $"isDone: {isDone}" };
        return View(items);
    }
}
```

보기 구성 요소의 Razor 파일은 `Invoke` 메서드에 전달된 문자열을 나열합니다(*Views/Home/Components/PriorityList/Default.cshtml*).

```cshtml
@model List<string>

<h3>Priority Items</h3>
<ul>
    @foreach (var item in Model)
    {
        <li>@item</li>
    }
</ul>
```

::: moniker range=">= aspnetcore-1.1"

보기 구성 요소는 다음 방법 중 하나를 사용하여 Razor 파일(예: *Views/Home/Index.cshtml*)에서 호출됩니다.

* <xref:Microsoft.AspNetCore.Mvc.IViewComponentHelper>
* [태그 도우미](xref:mvc/views/tag-helpers/intro)

<xref:Microsoft.AspNetCore.Mvc.IViewComponentHelper> 접근 방법을 사용하려면 `Component.InvokeAsync`를 호출합니다.

::: moniker-end

::: moniker range="< aspnetcore-1.1"

보기 구성 요소는 <xref:Microsoft.AspNetCore.Mvc.IViewComponentHelper>를 사용하여 Razor 파일(예: *Views/Home/Index.cshtml*)에서 호출됩니다.

`Component.InvokeAsync`를 호출합니다.

::: moniker-end

```cshtml
@await Component.InvokeAsync(nameof(PriorityList), new { maxPriority = 4, isDone = true })
```

::: moniker range=">= aspnetcore-1.1"

태그 도우미를 사용하려면 `@addTagHelper` 지시문을 사용하여 보기 구성 요소가 포함된 어셈블리를 등록합니다(보기 구성 요소는 `MyWebApp`이라는 어셈블리에 있음).

```cshtml
@addTagHelper *, MyWebApp
```

Razor 태그 파일에서 보기 구성 요소 태그 도우미를 사용합니다.

```cshtml
<vc:priority-list max-priority="999" is-done="false">
</vc:priority-list>
```

::: moniker-end

`PriorityList.Invoke`의 메서드 시그니처는 동기식이지만, Razor는 태그 파일에서 `Component.InvokeAsync`를 사용하여 메서드를 찾고 호출합니다.

## <a name="all-view-component-parameters-are-required"></a>모든 뷰 구성 요소 매개 변수 필요

뷰 구성 요소의 각 매개 변수는 필수 특성입니다. 이 [GitHub 문제](https://github.com/dotnet/AspNetCore/issues/5011)를 참조하세요. 매개 변수가 하나라도 생략되면

* `InvokeAsync` 메서드 시그니처가 일치하지 않게 되므로 메서드가 실행되지 않습니다.
* ViewComponent가 마크업을 렌더링하지 않습니다.
* 오류가 throw되지 않습니다.

## <a name="additional-resources"></a>추가 자료

* [보기에 종속성 주입](xref:mvc/views/dependency-injection)
