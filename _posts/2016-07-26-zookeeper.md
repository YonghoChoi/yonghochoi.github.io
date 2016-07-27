---
layout: post
title:  "ZooKeeper 개요"
date:   2016-07-26 14:31:00 +0900
tags: [zookeeper]
---

## Fundamentals

### 주키퍼의 아키텍쳐

![](http://yonghochoi.github.io/images/zookeeper/zookeeper-1.png)

* Client : 서버로 접근하여 정보를 얻는 분산 어플리케이션 클러스트 내 노드들 중 하나이다. 특정 시간 주기로 모든 클라이언트는 서버로 메세지를 보내고 이를 통해 서버는 클라이언트가 살아있음을 인지한다. 이와 유사하게 서버도 클라이언트가 연결 될 때 응답을 전송한다. 클라이언트가 연결된 서버로부터 응답을 받지 못하면 자동으로 다른 서버로 메세지를 redirect 한다.
* Server : 서버는 주키퍼 앙상블 내 노드들 중 하나로서 클라이언트에게 서비스를 제공한다. 클라이언트에게 응답 패킷을 전달하므로써 서버가 살아있다는 것을 알린다.
* Ensemble : 주키퍼 서버들의 그룹이다. 하나의 앙상블은 최소한 3개의 노드를 권장한다.
* Leader : 연결된 노드들 중 어떠한 노드가 중단되더라도 자동 복구를 수행하기 위한 서버 노드이다. 리더는 서버가 구동될 때 선출된다.
* Follower : 리더의 명령을 따르는 서버 노드이다.

### 계층에 따른 네임스페이스

주키퍼의 노드를 znode라고 한다. 모든 znode는 하나의 이름으로 식별되고 일련의 "/" 로 구분된다.

![](http://yonghochoi.github.io/images/zookeeper/zookeeper-2.png)

다이어그램을 보면 먼저 "/"로 된 root znode를 가지고 있다. 그 아래 두개의 logical 네임스페이스인 config와 workers가 있다.

config 네임스페이스는 중앙 집중식 설정 관리(centralized configuration management)를 위해 사용되고, workers 네임스페이스는 네이밍을 위해 사용된다.

config 네임스페이스 아래의 각각의 노드들은 1MB까지 데이터를 저장할 수 있다. 이 구조의 주 목적은 동기화된 데이터와 znode의 메타데이터를 저장하기 위한 것이다. 이 구조를 주키퍼 데이터 모델이라고 부른다.

주키퍼 데이터 모델 내 모든 znode는 stat을 가지고 있다. 이 stat은 버전 넘버와 ACL, timestamp, Data의 크기를 포함한 znode의 메타데이터를 제공한다.

* Version number : 모든 znode는 하나의 버전 번호를 가진다. 이는 znode와 관련된 data가 변경될 떄마다 이에 상응하는 버전번호 또한 증가된다는 것을 의미한다. 이 개념은 복수의 주키퍼 클라이언트들이 동일한 znode에 대해 연산을 수행하려 할 때 중요하다.
* ACL(Action Control List) : ACL은 znode에 접근하기 위한 기본적인 인증 메카니즘이다. 이것은 모든 znode의 read와 write 연산을 통제한다.
* Timestamp : 타임스탬프는 znode의 생성과 변경에 대한 경과시간을 의미한다.(millisecond 단위) 주키퍼는 Transaction ID인 zxid로 znode의 모든 변화를 식별한다. zxid는 유일하고 각 transaction을위한 시간을 유지한다. 그래서 하나의 Request부터 다른 Request까지의 경과시간을 쉽게 알아낼 수 있다.
* Data length : 하나의 znode 내에 저장된 데이터의 총량을 의미한다. 최대 1MB 까지 저장할 수 있다.

### Znode의 타입

* Persistence znode : 특정 znode가 생성되고 연결이 끊어져도 znode는 살아있다. 별도로 명시하지 않는 한 모든 znode는 persistent가 default이다.
* Ephemeral znode : 클라이언트가 살아있는 동안만 활성화된다. 주키퍼 앙상블로부터 클라이언트의 연결이 끊어지만 ephemeral znode 역시 자동으로 제거된다. ephemeral znode가 제거되면 다음 적절한 node가 그 위치를 채우게 된다. Ephemeral znode는 리더 선출에서 중요한 역할을 한다.
* Sequential znode : Sequential znode는 persistent가 될 수도 있고, ephemeral이 될 수도 있다. 새로운 znode가 sequntial znode로 생성되었다면 주키퍼는 원래 이름에 10진수를 추가하여 znode의 경로를 set한다. 예를 들어 znode의 경로가 /myapp인 sequential znode가 생성되었다면 주키퍼는 이 경로를 /myapp0000000001로 변경할 것이고 다음 sequence number를 0000000002로 설정할 것이다. 두 sequential znode들이 동시에 생성될 경우라도 주키퍼는 절대 같은 번호를 사용하지 않는다. sequential znode는 Locking과 Synchroniztion에서 중요한 역할을 한다.

### Sessions

세션은 주키퍼의 매우 중요한 operation이다. 하나의 session 내 Request들은 FIFO 순으로 실행된다. 하나의 클라이언트가 처음 서버로 연결될 때 세션은 established 상태가 되고 하나의 session id를 발급받는다.

클라이언트는 유효한 세션을 유지하기 위해 특정 주기에 heartbeat를 전송한다. 주키퍼 앙상블이 서비스가 구동될떄 설정했던 sessiontimeout을 초과하여 클라이언트로부터 heartbeat를 수신받지 못한다면 클라이언트가 죽은것으로 간주한다.

세션 타임아웃은 millisecond 단위이고, 어떤 이유에서 세션이 종료될 때 ephemeral znode 또한 제거된다.

### Watches

주키퍼 앙상블 내 변화에 대한 알림을 클라이언트에게 전달하기 위한 간단한 메커니즘이다. 클라이언트들은 특정 znode를 reading하는 동안 watche를 설정할 수 있다. watches는 onwichiclientregisters 변경에 대해 znode에 등록된 client에게 알림을 전송한다.

znode의 변화들은 znode와 관련된 변경이나 znode의 children내 변경이다. Watches는 변경에 대해 오직 한번만 트리거된다. 세션이 만료되면 클라이언트가 서버로부터 disconnect되고 watches 또한 제거된다.

## Workflow

### 주키퍼 앙상블 내 노드들

* 서버 수는 주로 홀수로 구성
	* 서버 간의 데이터 불일치가 발생하면 데이터 보정이 필요한데 이 때 과반수의 룰을 적용하기 때문에 서버를 홀수로 구성하는 것이 데이터 정합성 측면에서 유리하다.

![](http://yonghochoi.github.io/images/zookeeper/zookeeper-3.png)

* Write : Write 프로세스는 리더노드에서 처리된다. 리더는 모든 znode들로 write request를 전달하고 znode들로부터의 답변을 기다린다. znode들 중 절반이 답변을 했다면 write 프로세스가 성공했다고 판단한다.
* Read : Read는 연결된 특정 znode로부터 내부적으로 수행된다. 그래서 클러스터와 상호작용이 필요없다.
* Replicated Database : 주키퍼 내 데이터를 저장하는데 사용한다. 각 znode는 자신 소유의 database를 가지고 있고 모든 znode는 일관성을 유지하기 위해 항상 같은 데이터를 가진다.
* Leader : 리더는 wrtie request들의 처리를 책임지고 있는 znode이다.
* Follower : 팔로워들은 클라이언트로부터 작성된 request들을 수신하고 리더 znode로 수신받은 request들을 전달한다.
* Request Processor : 리더 노드에만 존재한다. follower 노드로부터 온 write request들을 통제한다.
* Atomic broadcasts : 리더 노드로부터 팔로워노드로 변경된 것들을 브로드캐스팅하는 것을 책임진다.

## Leader Election

리더 선출을 위한 과정은 다음과 같다.

1. 모든 노드들은 ephemeral znode와 같은 경로에 /app/leader_election/guid_의 형태로 하나의 sequential을 생성한다.
2. 주키퍼 앙상블은 경로에 순차적으로 증가하는 10진 정수를 뒤에 붙일 것이고, znode는 /app/leader_election/guid_0000000001, /app/leader_election/guid_0000000002와 같이 만들어질 것이다.
3. 가장 작은 번호로 만들어진 znode가 리더가 되고, 다른 노드들은 팔로워가 된다.
4. 각각의 팔로워들은 자기 자신보다 작고 가장 근접한 번호의 노드를 watch한다. 예를들어 /app/leader_election_guid_0000000008이라면 ...0007을 watch 할 것이고 ...0007이라면 ...0006을 watch할 것이다.
5. 리더가 다운되면 연관된 znode (/app/leader_electionN)이 제거된다.
6. 다음 라인 내 팔로워 노드는 리더가 제거되었다는 것에 대해 watcher를 통해 알림을 받게 될 것이다.
7. 다음 라인 내 팔로워 노드는 가장 작은 수를 가진 다른 znode가 있는지 검사한다. 없다면 해당 노드가 리더의 역할을 맡게될 것이다.
