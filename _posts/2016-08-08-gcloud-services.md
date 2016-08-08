---
layout: post
title:  "Google Cloud Services"
date:   2016-08-04 23:50:00 +0900
tags: [google cloud]
---

여기서는 공통적으로 많이 사용되는 서비스들에 대해 다룬다. 모든 서비스들의 리스트를 확인하려면 [상품과 서비스 페이지]를 참고 한다.

아래의 서비스 종류들에 대해 다룰 예정이다.

* 컴퓨팅과 호스팅 서비스
* 저장소 서비스
* 네트워킹 서비스
* 빅데이터 서비스

## 컴퓨팅과 호스팅 서비스

클라우드 플랫폼은 컴퓨팅과 호스팅 서비스를 위한 몇가지 옵션을 제공한다. 사용자는 관리되는 어플리케이션 플랫폼으로 작업을 하거나 유연함을 얻기 위한 leverage container 기술을 사용하거나 사용자가 소유한 클라우드 기반의 인프라를 빌드 하여 작업을 하는 것을 선택할 수 있다. 한쪽에서는 리소스 관리에 대한 책임은 모두 사용자가, 다른 한쪽에서는 이 책임들은 모두 구글이 맡아 하는 것을 생각해 볼 수 있다.

![](http://yonghochoi.github.io/images/google-cloud/ops-continuum.png)

### 어플리케이션 플랫폼

구글 앱 엔진은 클라우드 플랫폼의 PaaS(Platform As A Service)이다. 앱 엔진으로 구글은 사용자를 위한 모든 리소스 관리를 담당한다. 예를 들면, 사용자의 어플리케이션이 웹사이트로의 트래픽이 증가하여 컴퓨팅 리소스를 더 많이 요구하는 경우, 구글은 자동으로 리소스를 제공하여 시스템을 확장한다. 시스템 소프트웨어에 보안 업데이트가 필요한 경우에도 역시 자동으로 처리 된다.

앱엔진에서 사용자의 앱을 빌드할 때 다음과 같은 것들을 할 수 있다.

*  제공되는 언어(Python 2.7, Java 7, PHP, Go)로 앱 엔진의 [standard environment]에서 동작 중에 사용자의 앱을 빌드할 수 있다.

* 제공 되는 언어(Python 2.7/3.4, Java 8, Node.js, Ruby)로 앱 엔진의 [flexible environment]에서 동작 중에 사용자의 앱을 빌드 하거나 지원되는 언어 외의 다른 언어의 구현을 선택하여 사용할 수 있는 [custom runtimes]를 사용할 수 있다.

* 구글에서 사용자를 위해 앱 호스팅, 스케일링, 모니터링, 인프라스트럭쳐를 관리한다.

* 클라우드 플랫폼에서의 앱엔진 환경인 것 처럼 사용자의 로컬 장비에서 [App Engine SDK]를 사용하여 개발과 테스트를 할 수 있다.

* standard와 flexible environment에서 지원하도록 설계된 [저장 기술]을 쉽게 사용할 수 있다.
  * [구글 클라우드 SQL]은 사용자의 MySQL 데이터베이스이고, [앱 엔진 Datastore]는 스키마가 없는 NoSQL datastore이다. [구글 클라우드 저장소]는 사용자의 대용량 파일을 위한 공간을 제공한다.
  * standard environment에서는 Redis, MongoDB, PostgreSQL, Cassandra, Hadoop과 같은 다양한 서드파티 데이터베이스를 선택할 수 있다.
  * flexible environment에서는 구글 앱 엔진 인스턴스에서 접근이 가능한 데이터베이스라면 어떠한 서드파티 데이터베이스든 쉽게 사용할 수 있다.
  * 그 외 환경에서는 Compute 엔진 위에서 이 서드파티 데이터베이스들이 호스팅 될 수 있다. 다른 클라우드 프로바이더나 on-premise로 호스트되거나 서드파티 제조사로부터 관리되는 것으로부터 호스팅 된다.

* standard environment의 [Cloud Endpoints]를 사용할 수 있다. 이는 다른 어플리케이션으로부터 간편하게 데이터에 접근하여 사용할 수 있는 API들과 클라이언트 라이브러리이다.

* email과 사용자관리와 같은 관리가 필요한 서비스들이 내장되어 있다.

* 기존의 보안 설계와 개발 과정을 보완하여 보안 취약점을 식별해내는 [Cloud Security Scanner]를 사용할 수 있다.

* 맥이나 윈도우 환경에서 앱 엔진 GUI 툴을 사용하거나 커맨드 라인을 사용해서 사용자의 앱을 배포할 수 있다.

* standard environment는 Central US나 Western Europe regions에서 사용자의 앱이 실행된다.

앱 엔진 요소들의 모든 리스트와 설명을 보고 싶다면 [App Engine Documentation]을 참고 한다.

[상품과 서비스 페이지]: https://cloud.google.com/products/
[standard environment]: https://cloud.google.com/appengine/docs/about-the-standard-environment
[flexible environment]: https://cloud.google.com/appengine/docs/flexible/
[custom environment]: https://cloud.google.com/appengine/docs/flexible/custom-runtimes/
[App Engine SDK]: https://cloud.google.com/appengine/downloads
[저장 기술]: https://cloud.google.com/appengine/docs/about-the-standard-environment#storage
[구글 클라우드 SQL]: https://cloud.google.com/appengine/docs/about-the-standard-environment#cloudsql
[앱 엔진 Datastore]: https://cloud.google.com/appengine/docs/about-the-standard-environment#datastore
[구글 클라우드 저장소]: https://cloud.google.com/appengine/docs/about-the-standard-environment#gcslibrary
[Cloud Endpoints]: https://cloud.google.com/appengine/docs/about-the-standard-environment#Endpoints
[Cloud Security Scanner]: https://cloud.google.com/security-scanner/
[App Engine Documentation]: https://cloud.google.com/appengine/docs
