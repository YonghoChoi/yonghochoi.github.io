---
layout: post
title:  "StartSSL을 사용하여 인증서 생성하기"
date:   2016-10-03 17:30:00 +0900
tags: [security]
---

## SSL

###  대칭키 암호화

암호화와 복호화가 동일한 키를 가지고 수행된다. 로컬에서만 암/복호화를 수행한다면 크게 문제될 것은 없지만 원격지에 있는 대상에게 암호화된 data를 전송하고 원격지에서 복호화를 해야한다면 키를 전달하는데에 있어서 보안 이슈가 발생한다.

{% highlight shell %}
$ openssl enc -e -des3 -salt -in plaintext.txt -out ciphertext.bin
{% endhighlight %}

* openssl을 이용하여 des3 방식으로 암호화
* plaintext.txt 파일을 암호화하여 ciphertext.bin 파일을 생성

  ​



### 공개키 암호화

대칭키 암호화의 취약점을 보완. RSA 방식은 두개의 키를 가지고 암/복호화를 수행하는데 A,B 키가 있다고 가정하면, A의 키로 암호화를 한 경우 B의 키로만 복호화를 할 수가 있고, B의 키로 암호화를 한 경우 A의 키로만 복호화를 할 수 있다. 암호화하는 키와 복호화하는 키가 다르다고 하여 비대칭 키라고 한다.



클라이언트와 서버 간의 통신에서 암/복호화를 수행하기 위해 자신만 알 수 있는 private key와 외부에 공개할 public key를 소유한다.  서버는 클라이언트에게 public key를 전달해 주고, data를 전송할 때 자신의 private key로 암호화하여 전송하면 클라이언트는 public key를 이용하여 복호화를 수행한다. 클라이언트도 같은 방법으로 서버에게 data를 전달한다.



SSL의 경우에도 같은 방식을 사용하고 있는데, 비대칭 키의 암/복호화 과정이 대칭키의 암/복호화보다 부하를 많이 주기 때문에 대칭키와 혼용해서 사용하는 방식을 택하고 있다.



#### private key 생성

{% highlight shell %}
$ openssl genrsa -out private.pem 1024
Generating RSA private key, 1024 bit long modulus
..................................................++++++
......++++++
e is 65537 (0x10001)
{% endhighlight %}

- rsa 방식으로 공개키 암호화를 생성
- 마지막 인자의 1024의 수치가 높아질 수록 암호화의 복잡도가 증가. 그에 비례해서 cpu 사용률도 증가.

#### public key 생성

{% highlight shell %}
$ openssl rsa -in private.pem -out public.pem -outform PEM -pubout
writing RSA key
{% endhighlight %}

- 위에서 만든 private key를 사용하여 public key 생성

#### encrypt

{% highlight shell %}
$ openssl rsautl -encrypt -inkey public.pem -pubin -in plaintext.txt -out encrypttext.ssl
{% endhighlight %}

- rsa를 사용하여 위에서 생성한 public.pem 공개키로 plaintext.text를 암호화하여 encrypttext.ssl 파일 생성

#### decrypt

{% highlight shell %}
$ openssl rsautl -decrypt -inkey private.pem -in encrypttext.ssl -out decrypttext.txt
{% endhighlight %}

- rsa를 사용하여 위에서 생성한 private.pem 개인키로 공개키로 암호화했던 encrypttext.ssl 파일을 복호화 하여 decrypttext.txt 파일 생성



### 브라우저와 서버간 SSL을 이용한 통신



인증기관에 의해 암호화된 인증서를 서버가 소유하고 있고(인증서 발급 과정은 아래 요약 참고), 이 인증서 안에는 서버의 public key가 포함되어 있다. 브라우저는 인증기관을 통해 인증기관의 public key를 받아서 서버로 부터 받은 인증서를 복호화한다. (인증기관에서 암호화한 것이기 때문에 인증기관의 public key로 복호화 가능) 그럼 이제 브라우저는 서버의 public key를 소유하게 되고 서버에서 서버의 private key로 암호화한 data를 복호화 할 수 있게 된다. 클라이언트는 대칭크를 만들어서 서버의 public key로 암호화를 한 후 서버에 전달하고, 서버는 자신의 private key로 이를 복호화하여 이 후 통신에서는 대칭키로 암호화한 데이터를 전달한다.



요약을 해보면,



1. 서버에서 인증기관으로 인증서 요청.
2. 인증 기관에서는 인증 정보와 함께 서버의 public key를 인증기관의 private key로 암호화하여 인증서 제작
3. 만들어진 인증서를 서버에 전달.
4. 클라이언트에서 사용하는 브라우저에는 각 인증기관의 public key가 내장되어 있다.
5. 브라우저에서 서버로 접속 요청
6. 서버에서는 인증기관으로 부터 발급받은 인증서를 브라우저에 전달
7. 브라우저에서 내장되어 있는 해당 인증기관의 public key로 인증서를 복호화.
8. 클라이언트는 해당 도메인의 인증 정보와 서버의 public key를 획득한다.
9. 대칭키를 생성한 후 획득한 서버의 public key로 암호화하여 서버에 전달한다.
10. 서버의 public key로 암호화된 data 이므로 private key를 사용하여 복호화하여 대칭키를 획득한다.
11. 브라우저와 서버가 동일한 대칭키를 획득하게 되었으므로 이 후 통신은 대칭키 암호화 방식으로 수행한다.






## StartSSL을 사용하여 인증서 만들기

### 가입

* https://startssl.com/ 로 접속

* 가입이 되어 있지 않다면 Sign up

  ![](http://yonghochoi.github.io/images/security/startssl_1.png)

* 가입 절차를 수행하고 나면 .p12 인증서 파일을 다운 받을 수 있게 된다.

  * 이 인증서 파일은 인증서를 관리하기 위해 startssl에 접속할 때 사용되는 것으로 이 인증서가 없으면 로그인이 불가능하다. 그러므로 잘 관리해야함.

* 다운 받은 인증서를 브라우저에 등록한다.

  * 크롬의 경우 설정에 들어가서 인증서 관리에 추가해주면 된다.

* 도메인에 대한 인증 절차 수행

  ![](http://yonghochoi.github.io/images/security/startssl_2.png)

  ![](http://yonghochoi.github.io/images/security/startssl_3.png)

  ​

* https를 사용할 도메인에 대한 검증을 수행한다.

  ![](http://yonghochoi.github.io/images/security/startssl_4.png)

* 인증 코드를 받을 이메일을 선택한다.

  ![](http://yonghochoi.github.io/images/security/startssl_5.png)

  ​

* To "Order SSL Certificate"를 선택하거나 메인 화면의 Control pannel 메뉴를 선택하여 private 생성 절차 수행 한다.

  ![](http://yonghochoi.github.io/images/security/startssl_6.png)

* 호스트네임 입력.

  ![](http://yonghochoi.github.io/images/security/startssl_7.png)

  * 상단에 위치한 도메인이 공통 이름이 됨.

* openssl을 사용하여 private 키 생성

  {% highlight shell %}
  $ openssl req -newkey rsa:2048 -keyout username.key -out username.csr
  {% endhighlight %}

* .crt 파일과 .key 파일이 생성되는데 crt 파일의 내용을 등록한다.

  ![](http://yonghochoi.github.io/images/security/startssl_8.png)

* 인증서 등록 완료

  ![](http://yonghochoi.github.io/images/security/startssl_9.png)

* 최종 생성된 인증관련 파일들

  * username.key : 서버쪽 비공개 키
  * username.crt : SSL 디지털 인증서
    * 인증서와 username.key(private key)로 생성한 public key가 포함되어 있음.
  * username.pem : ROOT CA 인증서
  * intermediate.pem : 중계자 인증서
