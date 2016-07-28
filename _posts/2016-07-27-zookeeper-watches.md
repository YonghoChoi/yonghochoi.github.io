---
layout: post
title:  "ZooKeeper Watches"
date:   2016-07-27 11:00:00 +0900
tags: [zookeeper]
---

주키퍼에서의 모든 읽기 연산(getData(), getChildren(), exists())들은 watch를 설정할 수 있는 옵션을 가지고 있다. 하나의 watch event는 데이터의 변경이 발생 하였을 때 한번만 트리거 되어 watch로 등록되어 있는 클라이언트에게 전송된다.

* One-time trigger
