---
layout: post
title:  "ZooKeeper API"
date:   2016-07-26 17:00:00 +0900
tags: [zookeeper]
---

주키퍼는 Java와 C를 위한 공식적인 API를 제공하고, 주키퍼 커뮤니티에서 .NET, python 등 여러 언어의 비공식적인 API를 제공하고 있다.

## 주키퍼 API의 기본

Znode는 주키퍼 앙상블의 핵심 컴포넌트이고 주키퍼 API는 주키퍼 앙상블과 znode의 세세히 조작할 수 있는 간단한 메소드를 제공한다.

* 주키퍼 앙상블에 접속하면 주키퍼 앙상블은 클라이언트에게 하나의 Session ID를 발급한다.
* 주기적으로 서버에 heartbeats를 전송한다. 그렇지 않으면 주키퍼 앙상블은 Session ID를 만료하고 클라이언트는 재연결해야한다.
* znode Get/Set은 session ID가 존재하는 한 활성화된다.
* 모든 작업이 완료되었다면 주키퍼 앙상블과의 연결을 종료한다. 클라이언트가 오랫동안 비활성화 되면, 주키퍼 앙상블은 자동적으로 클라이언트와의 연결을 종료한다.

## Java 바인딩

주키퍼 API의 중점적인 부분은 ZooKeeper class 이다. 이는 주키퍼 앙상블과의 연결을 위해 클래스의 생성자와 메소드를 통해 옵션들을 제공한다.

* connect
* create
* exist
* getData
* setData
* getChildren
* delete
* close

## 주키퍼 앙상블로 연결

ZooKeeper 클래스는 생성자를 통해 접속할 수 있는 기능을 제공한다.

{% highlight java %}
ZooKeeper(String connectionString, int sessionTimeout, Watcher watcher)
{% endhighlight %}

* connectionString : 주키퍼 앙상블 host
* sessionTimeout : milliseconds 형식의 세션 타임아웃
* watcher : Watcher 인터페이스를 구현하고 있는 하나의 오브젝트. 주키퍼 앙상블은 watcher 오브젝트를 통해 접속 상태를 반환한다.

새로운 헬퍼 클래스인 ZooKeeperConnection을 생성하고 connect 메서드를 추가해보자. connect 메서드는 하나의 ZooKeeper 오브젝트를 생성하고 주키퍼 앙상블로 연결하고 나서 오브젝트를 반환한다.

이 예제의 CountDownLatch는 클라이언트가 연결될 때 까지 메인 프로세스를 멈추는데 wait을 사용한다.

주키퍼 앙상블은 Watcher callback을 통해 커넥션 상태를 받는다. Watcher callback은 주키퍼 앙상블로 클라이언트가 연결 될 때 한번 호출될 것이고 Watcher callback은 메인 프로세스의 await lock을 해제하기 위해 CountDownLatch의 countDown 메서드를 호출한다.

{% highlight java %}
public class ZooKeeperConnection {
    private ZooKeeper zoo;
    final CountDownLatch connectedSignal = new CountDownLatch(1);

    public ZooKeeper connect(String host) throws IOException, InterruptedException {
        zoo = new ZooKeeper(host, 5000, new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                if(event.getState() == Event.KeeperState.SyncConnected) {
                    connectedSignal.countDown();
                }
            }
        });

        connectedSignal.await();
        return zoo;
    }

    public void close() throws InterruptedException {
        zoo.close();
    }
}
{% endhighlight %}

## Znode 생성

ZooKeeper 클래스는 주키퍼 앙상블에 새로운 znode를 생성하기 위한 create 메서드를 제공한다.

{% highlight java%}
create(String path, byte[] data, List<ACL> acl, CreateMode createMode)
{% endhighlight %}

* path : Znode의 경로. ex) /myapp1, /myapp2, /myapp1/mydata1, myapp2/mydata1/myanothersubdata
* data : 특정 znode 경로에 저장된 데이터
* acl : 생성된 노드의 접근 제어 리스트. ZooKeeper API는 기본 acl 리스트를 얻기 위한 ZooDefs.Ids static 인터페이스를 제공한다. ex) ZooDefs.Ids.OPEN_ACL_UNSAFE는 열려있는 znode를 위한 acl의 리스트를 반환한다.
* createMode : node의 타입이다. ephemeral이 될 수도 있고 sequential이 될 수도 있고 둘 다가 될 수도 있다. (하나의 enum 값)

create 메서드의 기능을 확인해보기 위해 새로운 자바 어플리케이션인 ZKCreate.java를 생성해보자. 메인 메서드에서는 ZooKeeperConnection 타입의 오브젝트 하나를 생성하고 주키퍼 앙상블로 connect 메서드를 통해 접속한다.

connect 메서드는 ZooKeeper 오브젝트인 zk를 반환할 것이다. custom path와 데이터와 함께 zk 오브젝트의 create 메서드를 호출한다.

{% highlight java %}
public class ZKCreate {
    private static ZooKeeper zk;
    private static ZooKeeperConnection conn;

    public static void create(String path, byte[] data) throws KeeperException, InterruptedException {
        zk.create(path, data, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
    }

    public static void main(String[] args) {
        String path = "/MyFirstZnode";

        byte[] data = "My first zookeeper app".getBytes();

        try {
            conn = new ZooKeeperConnection();
            zk = conn.connect("localhost");
            create(path, data);
            conn.close();
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
}
{% endhighlight %}

## Exist

ZooKeeper 클래스는 Znode의 존재 여부를 체크하는 exists 메서드를 제공한다. 찾으려는 znode가 존재하면 해당 znode의 메타데이터를 반환한다.

{% highlight java %}
exists(String path, boolean watcher)
{% endhighlight %}

{% highlight java %}
public class ZKExist {
    private static ZooKeeper zk;
    private static ZooKeeperConnection conn;

    public static Stat znode_exists(String path) throws KeeperException, InterruptedException {
        return zk.exists(path, true);
    }

    public static void main(String[] args) {
        String path = "/MyFirstZnode";

        try {
            conn = new ZooKeeperConnection();
            zk = conn.connect("localhost");
            Stat stat = znode_exists(path);

            if(stat != null) {
                System.out.println("Node exists and the node version is " + stat.getVersion());
            } else {
                System.out.println("Node does not exists.");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

## getData 메서드

ZooKeeper 클래스는 지정된 znode 내에 저장된 데이터와 status 정보를 얻기위한 getData 메서드를 제공한다.

{% highlight java%}
getData(String path, Watcher watcher, Stat stat)
{% endhighlight %}

* path : znode의 경로.
* watcher : Watcher 타입의 콜백 함수. 주키퍼 앙상블은 지정된 znode의 데이터가 변경될 때 이 Watcher 콜백을 통해 알리게 된다.
* stat : znode의 메타데이터를 반환한다.

{% highlight java%}
public class ZKGetData {
    private static ZooKeeper zk;
    private static ZooKeeperConnection conn;

    public static Stat znode_exists(String path) throws KeeperException, InterruptedException {
        return zk.exists(path, true);
    }

    public static void main(String[] args) {
        String path = "/MyFirstZnode";
        final CountDownLatch connectedSignal = new CountDownLatch(1);

        try {
            conn = new ZooKeeperConnection();
            zk = conn.connect("localhost");
            Stat stat = znode_exists(path);

            if (stat != null) {
                byte[] b = zk.getData(path, event -> {
                    if(event.getType() == Watcher.Event.EventType.None) {
                        switch(event.getState()) {
                            case Expired:
                                connectedSignal.countDown();
                                break;
                        }
                    } else {
                        String path1 = "/MyFirstZnode";

                        try{
                            byte[] bn = zk.getData(path1, false, null);
                            String data = new String(bn, "UTF-8");
                            System.out.println(data);
                            connectedSignal.countDown();
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                }, null);

                String data = new String(b, "UTF-8");
                System.out.println(data);

                connectedSignal.await();
            } else {
                System.out.println("Node does not exists.");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}
{% endhighlight %}

어플리케이션을 실행하면 현재 셋팅된 data가 콘솔창에 출력되고 await 상태로 대기하고 있다. Watcher에 대한 콜백 함수도 지정해놓았으므로 zkCli를 사용하여 데이터를 set해보면 Watcher 콜백함수가 실행될 것이다.

{% highlight shell %}
bin/zkCli.sh
>>> set /MyFirstZnode Hello
{% endhighlight %}

그러면 콘솔창에 콜백함수가 호출되어 Hello가 출력되고 countDown 메서드를 통해 await 상태의 lock을 해제하므로 어플리케이션이 종료된다.

{% highlight shell %}
Hello
{% endhighlight %}

## setData 메서드

ZooKeeper 클래스는 지정한 znode에 data를 변경하여 추가하기 위한 setData 메서드를 제공한다.

{% highlight java %}
setData(String path, byte[] data, int version)
{% endhighlight %}

* path : znode의 경로.
* data : 지정된 znode 경로에 저장된 데이터.
* version : znode의 현재 버전. ZooKeeper는 data에 변화가 있으면 znode의 버전 번호를 갱신한다.

{% highlight java %}
public class ZKSetData {
    private static ZooKeeper zk;
    private static ZooKeeperConnection conn;

    public static void update(String path, byte[] data) throws KeeperException, InterruptedException {
        zk.setData(path, data, zk.exists(path, true).getVersion());
    }

    public static void main(String[] args) {
        String path = "/MyFirstZnode";
        byte[] data = "Success".getBytes();

        try {
            conn = new ZooKeeperConnection();
            zk = conn.connect("localhost");
            update(path, data);
            System.out.println("Done.");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

data가 변경된 것을 확인하기 위해 zkCli를 사용하여 get 명령을 수행해본다.

{% highlight shell %}
bin/zkCli.sh
>>> get /MyFirstZnode
Success
cZxid = 0xf
ctime = Wed Jul 27 18:12:21 KST 2016
mZxid = 0x24
mtime = Wed Jul 27 20:57:11 KST 2016
pZxid = 0xf
cversion = 0
dataVersion = 2
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 0
{% endhighlight %}

## getChildren 메서드

ZooKeeper 클래스는 특정한 znode의 모든 sub-node들을 얻어내기 위한 getChildren 메서드를 제공한다.

{% highlight java %}
getChildren(String path, Watcher watcher)
{% endhighlight %}

* path : znode의 경로.
* watcher : Watcher 타입의 콜백 함수. 주키퍼 앙상블은 지정된 znode의 데이터가 변경될 때 이 Watcher 콜백을 통해 알리게 된다.

{% highlight java %}
public class ZKGetChildren {
    private static ZooKeeper zk;
    private static ZooKeeperConnection conn;

    public static Stat znode_exists(String path) throws KeeperException, InterruptedException {
        return zk.exists(path, true);
    }

    public static void main(String[] args) {
        String path = "/MyFirstZnode";

        try {
            conn = new ZooKeeperConnection();
            zk = conn.connect("localhost");
            Stat stat = znode_exists(path);

            if(stat != null) {
                List<String> children = zk.getChildren(path, false);
                children.forEach(System.out::println);
            } else {
                System.out.println("Node does not exists.");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

sub-node가 존재하지 않는다면 zkCli를 사용하여 추가한다.

{% highlight shell %}
bin/zkCli.sh
>>> create /MyFirstZnode/myfirstsubnode Hi
>>> create /MyFirstZnode/mysecondsubnode Hi
{% endhighlight %}


## Delete Znode

ZooKeeper 클래스는 지정된 znode를 제거하기 위한 delete 메서드를 제공한다.

{% highlight java %}
delete(String path, int version)
{% endhighlight %}

* path : znode의 경로.
* version : znode의 현재 버전.

{% highlight java %}
public class ZKDelete {
    private static ZooKeeper zk;
    private static ZooKeeperConnection conn;

    public static void delete(String path) throws KeeperException, InterruptedException {
        zk.delete(path, zk.exists(path, true).getVersion());
    }

    public static void main(String[] args) {
        String path = "/MyFirstZnode";

        try {
            conn = new ZooKeeperConnection();
            zk = conn.connect("localhost");
            delete(path);
            System.out.println("Done.");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

이 때, 지정된 노드 내에 sub-node가 존재하면 Exception이 발생한다. 그러므로 하위 노드들을 전부 제거 후 delete를 수행해야 한다. sub-node도 마찬가지로 path 설정 후 delete 메서드로 제거.

{% highlight shell %}
org.apache.zookeeper.KeeperException$NotEmptyException: KeeperErrorCode = Directory not empty for /MyFirstZnode
	at org.apache.zookeeper.KeeperException.create(KeeperException.java:125)
	at org.apache.zookeeper.KeeperException.create(KeeperException.java:51)
	at org.apache.zookeeper.ZooKeeper.delete(ZooKeeper.java:873)
	at ZKDelete.delete(ZKDelete.java:11)
	at ZKDelete.main(ZKDelete.java:20)
{% endhighlight %}
