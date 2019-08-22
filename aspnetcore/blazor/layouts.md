---
title: ASP.NET Core Blazor 레이아웃
author: guardrex
description: Blazor apps에 대해 재사용 가능한 레이아웃 구성 요소를 만드는 방법에 대해 알아봅니다.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 07/02/2019
uid: blazor/layouts
ms.openlocfilehash: 2d652e149381f0a93e3135da978ab5737d47c6f1
ms.sourcegitcommit: 0b9e767a09beaaaa4301915cdda9ef69daaf3ff2
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 07/03/2019
ms.locfileid: "68948223"
---
# <a name="aspnet-core-blazor-layouts"></a><span data-ttu-id="48141-103">ASP.NET Core Blazor 레이아웃</span><span class="sxs-lookup"><span data-stu-id="48141-103">ASP.NET Core Blazor layouts</span></span>

<span data-ttu-id="48141-104">[Rainer Stropek](https://www.timecockpit.com)</span><span class="sxs-lookup"><span data-stu-id="48141-104">By [Rainer Stropek](https://www.timecockpit.com)</span></span>

<span data-ttu-id="48141-105">메뉴, 저작권 메시지 및 회사 로고와 같은 일부 앱 요소는 일반적으로 앱의 전체 레이아웃에 포함 되며 앱의 모든 구성 요소에서 사용 됩니다.</span><span class="sxs-lookup"><span data-stu-id="48141-105">Some app elements, such as menus, copyright messages, and company logos, are usually part of app's overall layout and used by every component in the app.</span></span> <span data-ttu-id="48141-106">이러한 요소의 코드를 앱의 모든 구성 요소에 복사 하는 것은 요소 중 하나&mdash;에 업데이트가 필요한 경우 모든 구성 요소를 업데이트 해야 하기 때문에 효율적이 지 않습니다.</span><span class="sxs-lookup"><span data-stu-id="48141-106">Copying the code of these elements into all of the components of an app isn't an efficient approach&mdash;every time one of the elements requires an update, every component must be updated.</span></span> <span data-ttu-id="48141-107">이러한 중복은 유지 관리 하기 어렵고 시간이 지남에 따라 일관 되지 않은 콘텐츠가 발생할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="48141-107">Such duplication is difficult to maintain and can lead to inconsistent content over time.</span></span> <span data-ttu-id="48141-108">*레이아웃* 은이 문제를 해결 합니다.</span><span class="sxs-lookup"><span data-stu-id="48141-108">*Layouts* solve this problem.</span></span>

<span data-ttu-id="48141-109">기술적으로 레이아웃은 또 다른 구성 요소입니다.</span><span class="sxs-lookup"><span data-stu-id="48141-109">Technically, a layout is just another component.</span></span> <span data-ttu-id="48141-110">레이아웃은 Razor 템플릿 또는 C# 코드에서 정의 되며 [데이터 바인딩](xref:blazor/components#data-binding), [종속성 주입](xref:blazor/dependency-injection)및 기타 구성 요소 시나리오를 사용할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="48141-110">A layout is defined in a Razor template or in C# code and can use [data binding](xref:blazor/components#data-binding), [dependency injection](xref:blazor/dependency-injection), and other component scenarios.</span></span>

<span data-ttu-id="48141-111">*구성* 요소를 *레이아웃*으로 전환 하려면 다음을 수행 합니다.</span><span class="sxs-lookup"><span data-stu-id="48141-111">To turn a *component* into a *layout*, the component:</span></span>

* <span data-ttu-id="48141-112">레이아웃 내의 `LayoutComponentBase`렌더링 된 콘텐츠에 대 `Body` 한 속성을 정의 하는에서 상속 됩니다.</span><span class="sxs-lookup"><span data-stu-id="48141-112">Inherits from `LayoutComponentBase`, which defines a `Body` property for the rendered content inside the layout.</span></span>
* <span data-ttu-id="48141-113">Razor 구문 `@Body` 를 사용 하 여 레이아웃 태그에서 콘텐츠가 렌더링 되는 위치를 지정 합니다.</span><span class="sxs-lookup"><span data-stu-id="48141-113">Uses the Razor syntax `@Body` to specify the location in the layout markup where the content is rendered.</span></span>

<span data-ttu-id="48141-114">다음 코드 샘플은 레이아웃 구성 요소인 *Mainlayout razor*의 razor 템플릿을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="48141-114">The following code sample shows the Razor template of a layout component, *MainLayout.razor*.</span></span> <span data-ttu-id="48141-115">레이아웃은를 `LayoutComponentBase` 상속 하 고 `@Body` 탐색 표시줄과 바닥글 사이를 설정 합니다.</span><span class="sxs-lookup"><span data-stu-id="48141-115">The layout inherits `LayoutComponentBase` and sets the `@Body` between the navigation bar and the footer:</span></span>

[!code-cshtml[](layouts/sample_snapshot/3.x/MainLayout.razor?highlight=1,13)]

## <a name="specify-a-layout-in-a-component"></a><span data-ttu-id="48141-116">구성 요소에 레이아웃 지정</span><span class="sxs-lookup"><span data-stu-id="48141-116">Specify a layout in a component</span></span>

<span data-ttu-id="48141-117">Razor 지시어 `@layout` 를 사용 하 여 레이아웃을 구성 요소에 적용 합니다.</span><span class="sxs-lookup"><span data-stu-id="48141-117">Use the Razor directive `@layout` to apply a layout to a component.</span></span> <span data-ttu-id="48141-118">컴파일러는 구성 `@layout` 요소 클래스 `LayoutAttribute`에 적용 되는를로 변환 합니다.</span><span class="sxs-lookup"><span data-stu-id="48141-118">The compiler converts `@layout` into a `LayoutAttribute`, which is applied to the component class.</span></span>

<span data-ttu-id="48141-119">다음 구성 요소인 *masterlist. razor*의 내용은의 `MainLayout` `@Body`위치에 있는에 삽입 됩니다.</span><span class="sxs-lookup"><span data-stu-id="48141-119">The content of the following component, *MasterList.razor*, is inserted into the `MainLayout` at the position of `@Body`:</span></span>

[!code-cshtml[](layouts/sample_snapshot/3.x/MasterList.razor?highlight=1)]

## <a name="centralized-layout-selection"></a><span data-ttu-id="48141-120">중앙 레이아웃 선택</span><span class="sxs-lookup"><span data-stu-id="48141-120">Centralized layout selection</span></span>

<span data-ttu-id="48141-121">앱의 모든 폴더에는 선택적으로 *_Imports. razor*라는 템플릿 파일이 포함 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="48141-121">Every folder of an app can optionally contain a template file named *_Imports.razor*.</span></span> <span data-ttu-id="48141-122">컴파일러는 동일한 폴더에 있는 모든 Razor 템플릿의 imports 파일에 지정 된 지시문을 포함 하 고 모든 하위 폴더에서 재귀적으로 포함 됩니다.</span><span class="sxs-lookup"><span data-stu-id="48141-122">The compiler includes the directives specified in the imports file in all of the Razor templates in the same folder and recursively in all of its subfolders.</span></span> <span data-ttu-id="48141-123">따라서을 포함 `@layout MainLayout` 하는 *Imports. razor* 파일은 폴더의 모든 구성 요소가를 사용 `MainLayout`하는지 확인 합니다.</span><span class="sxs-lookup"><span data-stu-id="48141-123">Therefore, an *_Imports.razor* file containing `@layout MainLayout` ensures that all of the components in a folder use `MainLayout`.</span></span> <span data-ttu-id="48141-124">폴더 및 하위 폴더에 있는 모든 `@layout MainLayout` *razor* 파일을 반복적으로 추가할 필요는 없습니다.</span><span class="sxs-lookup"><span data-stu-id="48141-124">There's no need to repeatedly add `@layout MainLayout` to all of the *.razor* files within the folder and subfolders.</span></span> <span data-ttu-id="48141-125">`@using`지시문은 동일한 방식으로 구성 요소에도 적용 됩니다.</span><span class="sxs-lookup"><span data-stu-id="48141-125">`@using` directives are also applied to components in the same way.</span></span>

<span data-ttu-id="48141-126">다음 가져오기-razor 파일 가져오기 *(_s)* :</span><span class="sxs-lookup"><span data-stu-id="48141-126">The following *_Imports.razor* file imports:</span></span>

* <span data-ttu-id="48141-127">`MainLayout`.</span><span class="sxs-lookup"><span data-stu-id="48141-127">`MainLayout`.</span></span>
* <span data-ttu-id="48141-128">동일한 폴더와 하위 폴더에 있는 모든 Razor 구성 요소입니다.</span><span class="sxs-lookup"><span data-stu-id="48141-128">All Razor components in the same folder and any subfolders.</span></span>
* <span data-ttu-id="48141-129">`BlazorApp1.Data` 네임스페이스.</span><span class="sxs-lookup"><span data-stu-id="48141-129">The `BlazorApp1.Data` namespace.</span></span>
 
[!code-cshtml[](layouts/sample_snapshot/3.x/_Imports.razor)]

<span data-ttu-id="48141-130">*_Imports. razor* 파일은 [razor 뷰 및 페이지에 대 한 _ViewImports 파일과](xref:mvc/views/layout#importing-shared-directives) 비슷하지만 특히 razor 구성 요소 파일에 적용 됩니다.</span><span class="sxs-lookup"><span data-stu-id="48141-130">The *_Imports.razor* file is similar to the [_ViewImports.cshtml file for Razor views and pages](xref:mvc/views/layout#importing-shared-directives) but applied specifically to Razor component files.</span></span>

<span data-ttu-id="48141-131">Blazor 템플릿에서는 레이아웃 선택을 위해 Imports. razor 파일을 사용 합니다 *(_d).*</span><span class="sxs-lookup"><span data-stu-id="48141-131">The Blazor templates use *_Imports.razor* files for layout selection.</span></span> <span data-ttu-id="48141-132">Blazor 템플릿에서 만든 앱은 프로젝트 및 *Pages* 폴더의 루트에 *_imports. razor* 파일이 포함 되어 있습니다.</span><span class="sxs-lookup"><span data-stu-id="48141-132">An app created from a Blazor template contains the *_Imports.razor* file in the root of the project and in the *Pages* folder.</span></span>

## <a name="nested-layouts"></a><span data-ttu-id="48141-133">중첩 레이아웃</span><span class="sxs-lookup"><span data-stu-id="48141-133">Nested layouts</span></span>

<span data-ttu-id="48141-134">앱은 중첩 된 레이아웃으로 구성 될 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="48141-134">Apps can consist of nested layouts.</span></span> <span data-ttu-id="48141-135">구성 요소는 다른 레이아웃을 참조 하는 레이아웃을 참조할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="48141-135">A component can reference a layout which in turn references another layout.</span></span> <span data-ttu-id="48141-136">예를 들어 중첩 레이아웃은 다중 수준 메뉴 구조를 만드는 데 사용 됩니다.</span><span class="sxs-lookup"><span data-stu-id="48141-136">For example, nesting layouts are used to create a multi-level menu structure.</span></span>

<span data-ttu-id="48141-137">다음 예제에서는 중첩 된 레이아웃을 사용 하는 방법을 보여 줍니다.</span><span class="sxs-lookup"><span data-stu-id="48141-137">The following example shows how to use nested layouts.</span></span> <span data-ttu-id="48141-138">*EpisodesComponent* 파일은 표시할 구성 요소입니다.</span><span class="sxs-lookup"><span data-stu-id="48141-138">The *EpisodesComponent.razor* file is the component to display.</span></span> <span data-ttu-id="48141-139">구성 요소는 다음 `MasterListLayout`을 참조 합니다.</span><span class="sxs-lookup"><span data-stu-id="48141-139">The component references the `MasterListLayout`:</span></span>

[!code-cshtml[](layouts/sample_snapshot/3.x/EpisodesComponent.razor?highlight=1)]

<span data-ttu-id="48141-140">*Masterlistlayout. razor* 파일은를 `MasterListLayout`제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="48141-140">The *MasterListLayout.razor* file provides the `MasterListLayout`.</span></span> <span data-ttu-id="48141-141">레이아웃은 렌더링 되는 다른 `MasterLayout`레이아웃를 참조 합니다.</span><span class="sxs-lookup"><span data-stu-id="48141-141">The layout references another layout, `MasterLayout`, where it's rendered.</span></span> <span data-ttu-id="48141-142">`EpisodesComponent`가 표시 되 `@Body` 는 위치에 렌더링 됩니다.</span><span class="sxs-lookup"><span data-stu-id="48141-142">`EpisodesComponent` is rendered where `@Body` appears:</span></span>

[!code-cshtml[](layouts/sample_snapshot/3.x/MasterListLayout.razor?highlight=1,9)]

<span data-ttu-id="48141-143">마지막으로 *masterlayout* 에는머리글,주메뉴,바닥글등의최상위레이아웃요소가포함`MasterLayout` 됩니다.</span><span class="sxs-lookup"><span data-stu-id="48141-143">Finally, `MasterLayout` in *MasterLayout.razor* contains the top-level layout elements, such as the header, main menu, and footer.</span></span> <span data-ttu-id="48141-144">`MasterListLayout`가 표시`@Body` 되 면`EpisodesComponent` 이 렌더링 됩니다.</span><span class="sxs-lookup"><span data-stu-id="48141-144">`MasterListLayout` with `EpisodesComponent` are rendered where `@Body` appears:</span></span>

[!code-cshtml[](layouts/sample_snapshot/3.x/MasterLayout.razor?highlight=6)]

## <a name="additional-resources"></a><span data-ttu-id="48141-145">추가 자료</span><span class="sxs-lookup"><span data-stu-id="48141-145">Additional resources</span></span>

* <xref:mvc/views/layout>