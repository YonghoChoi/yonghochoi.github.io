---
layout: post
title:  "ZooKeeper 설치"
date:   2016-07-26 15:30:00 +0900
tags: [zookeeper]
---

## 설치

ZooKeeper를 다운 받고 압축을 해제한다. ZooKeeper를 시작하기 위해서는 설정 파일이 필요하므로 conf/zoo.cfg 파일을 생성한다.

{% highlight shell %}
  tickTime=2000
  dataDir=/var/lib/zookeeper
  clientPort=2181
{% endhighlight %}

* tickTime : milliseconds 단위의 heartbeat 시간을 의미한다.
* dataDir : in-memory 데이터베이스 스냅샷을 저장하기 위한 경로이고, 한편으로는 데이터베이스 갱신 시 작성되는 로그가 저장되는 경로이다.
* clientPort : 클라이언트의 커넥션을 listen 하는 port.

이제 ZooKeeper를 실행할 수 있다.

{% highlight shell %}
  bin/zkServer.sh start
{% endhighlight %}

* ZooKeeper의 로그 메세지는 log4j를 사용한다.
* 여기서는 ZooKeeper의 standalone mode로 수행하므로 replication이 없다. 그러므로 ZooKeeper 프로세스가 죽으면 서비스가 죽게 될 것이다.
* Replication을 적용하려면 [Running Replicated ZooKeeper] 참고

## ZooKeeper Storage 관리

* 장기간 동작하는 프로덕션 시스템에서 Zookeeper storage(dataDir과 logs)는 반드시 외부에서 관리되어야한다.
	* [maintenance] 참고.

## ZooKeeper로 연결

bin 디렉토리의 쉘을 이용하여 연결하는 방법과 ZooKeeper 패키지에 포함된 src 디렉토리의 c 컴파일을 이용한 방법이 있는데, 간단하게 bin 디렉토리의 쉘을 사용하여 서버로 접속한다.

{% highlight shell %}
  bin/zkCli.sh -server 127.0.0.1:2181
{% endhighlight %}

* 접속을 하고 나면 쉘 커맨드 입력이 가능해진다.
	* help 명령을 사용하여 실행 가능한 커맨드들의 리스트를 확인.

ls 명령을 통해 znode를 확인할 수 있다.

{% highlight shell %}
ls /
[zookeeper]
{% endhighlight %}

create 명령을 통해 znode 디렉토리를 생성한다. 인자로 znode와 연결짓는 string(이름)도 함께 전달한다.

{% highlight shell %}
  create /zk_test my_data
  Created /zk_test
{% endhighlight %}

ls / 명령을 해보면 추가된 znode 디렉토리를 확인할 수 있다.

{% highlight shell %}
  ls /
  [zookeeper, zk_test]
{% endhighlight %}

get 명령을 수행하여 znode가 잘 연결되었는지 확인한다.

{% highlight shell %}
  get /zk_test
{% endhighlight %}

set 명령으로 zk_test와 연결된 데이터를 변경할 수 있다.

{% highlight shell %}
  set /zk_test junk
{% endhighlight %}

delete 명령으로 node를 제거할 수 있다.

{% highlight shell %}
  delete /zk_test
{% endhighlight %}


[Running Replicated ZooKeeper]: https://zookeeper.apache.org/doc/trunk/zookeeperStarted.html#sc_RunningReplicatedZooKeeper
[maintenance]: https://zookeeper.apache.org/doc/trunk/zookeeperAdmin.html#sc_maintenance
