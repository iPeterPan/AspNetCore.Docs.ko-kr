---
title: HTTPS를 통해 Docker를 사용 하 여 ASP.NET Core 이미지 호스팅
author: rick-anderson
description: HTTPS를 통해 Docker를 사용 하 여 ASP.NET Core 이미지를 호스트 하는 방법을 알아봅니다.
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/05/2019
no-loc:
- Let's Encrypt
uid: security/docker-https
ms.openlocfilehash: 2f338e8883ca926c0f9a7ab339f58b088151cc87
ms.sourcegitcommit: 235623b6e5a5d1841139c82a11ac2b4b3f31a7a9
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 02/08/2020
ms.locfileid: "77089203"
---
# <a name="hosting-aspnet-core-images-with-docker-over-https"></a>HTTPS를 통해 Docker를 사용 하 여 ASP.NET Core 이미지 호스팅

작성자: [Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core는 [기본적으로 HTTPS를](/aspnet/core/security/enforcing-ssl)사용 합니다. [HTTPS](https://en.wikipedia.org/wiki/HTTPS) 는 [인증서](https://en.wikipedia.org/wiki/Public_key_certificate) 를 신뢰, id 및 암호화에 의존 합니다.

이 문서에서는 HTTPS를 사용 하 여 미리 작성 된 컨테이너 이미지를 실행 하는 방법을 설명 합니다.

개발 시나리오는 [HTTPS를 통해 Docker를 사용 하 여 ASP.NET Core 응용 프로그램 개발](https://github.com/dotnet/dotnet-docker/blob/master/samples/run-aspnetcore-https-development.md) 을 참조 하세요.

이 샘플에는 docker [클라이언트](https://www.docker.com/products/docker)의 [docker 17.06](https://docs.docker.com/release-notes/docker-ce) 이상이 필요 합니다.

## <a name="prerequisites"></a>사전 요구 사항

이 문서의 지침 중 일부에는 [.Net Core 2.2 SDK](https://www.microsoft.com/net/download) 이상이 필요 합니다.

## <a name="certificates"></a>인증서

도메인에 대 한 [프로덕션 호스팅을](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/) 위해서는 [인증 기관의](https://wikipedia.org/wiki/Certificate_authority) 인증서가 필요 합니다. [Let's Encrypt](https://letsencrypt.org/) 은 무료 인증서를 제공 하는 인증 기관입니다.

이 문서에서는 `localhost`에 대해 미리 작성 된 이미지를 호스팅하기 위해 [자체 서명 된 개발 인증서](https://en.wikipedia.org/wiki/Self-signed_certificate) 를 사용 합니다. 지침은 프로덕션 인증서를 사용 하는 것과 유사 합니다.

프로덕션 인증서의 경우:

* `dotnet dev-certs` 도구는 필요 하지 않습니다.
* 지침에 사용 되는 위치에 인증서를 저장할 필요가 없습니다. 사이트 디렉터리 내에 인증서를 저장 하지 않는 것이 좋지만 모든 위치는 작동 해야 합니다.

다음 섹션에 포함 된 지침은 Docker의 `-v` 명령줄 옵션을 사용 하 여 컨테이너에 인증서를 탑재 합니다. *Dockerfile*에서 `COPY` 명령을 사용 하 여 컨테이너 이미지에 인증서를 추가할 수 있지만 권장 되지는 않습니다. 다음과 같은 이유로 인증서를 이미지로 복사 하는 것은 권장 되지 않습니다.

* 개발자 인증서를 사용 하 여 테스트 하는 데 동일한 이미지를 사용 하는 것은 어렵습니다.
* 프로덕션 인증서를 사용 하 여 호스트 하는 데 동일한 이미지를 사용 하는 것은 어렵습니다.
* 인증서 공개에는 상당한 위험이 있습니다.

## <a name="running-pre-built-container-images-with-https"></a>HTTPS를 사용 하 여 미리 작성 된 컨테이너 이미지 실행

운영 체제 구성에 대해 다음 지침을 따르십시오.

### <a name="windows-using-linux-containers"></a>Linux 컨테이너를 사용 하는 Windows

인증서 생성 및 로컬 컴퓨터 구성:

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

위의 명령에서 `{ password here }`을 암호로 바꿉니다.

HTTPS에 대해 구성 된 ASP.NET Core를 사용 하 여 컨테이너 이미지를 실행 합니다.

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

암호는 인증서에 사용 된 암호와 일치 해야 합니다.

### <a name="macos-or-linux"></a>macOS 또는 Linux

인증서 생성 및 로컬 컴퓨터 구성:

```dotnetcli
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

`dotnet dev-certs https --trust`은 macOS 및 Windows 에서만 지원 됩니다. 배포판에서 지 원하는 방식으로 Linux에서 인증서를 신뢰 해야 합니다. 브라우저에서 인증서를 신뢰 해야 할 수도 있습니다.

위의 명령에서 `{ password here }`을 암호로 바꿉니다.

HTTPS에 대해 구성 된 ASP.NET Core를 사용 하 여 컨테이너 이미지를 실행 합니다.

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v ${HOME}/.aspnet/https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

암호는 인증서에 사용 된 암호와 일치 해야 합니다.

### <a name="windows-using-windows-containers"></a>Windows 컨테이너를 사용 하는 windows

인증서 생성 및 로컬 컴퓨터 구성:

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

위의 명령에서 `{ password here }`을 암호로 바꿉니다.

HTTPS에 대해 구성 된 ASP.NET Core를 사용 하 여 컨테이너 이미지를 실행 합니다.

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=\https\aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:C:\https\ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

암호는 인증서에 사용 된 암호와 일치 해야 합니다.
