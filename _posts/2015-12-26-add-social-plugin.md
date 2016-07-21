---
layout: post
title:  "웹 페이지에 소셜 플러그인 추가하기"
date:   2015-12-26 00:32:00 +0900
---
## 웹페이지에 소셜 공유 링크 추가하기

 [구글]이나 [트위터], [페이스북]에서는 트윗하기, 좋아요 버튼에 대한 API를 제공한다. 코드까지 제공해주기 때문에 API 제공 페이지에서 복사해서 간단하게 사용할 수가 있다.

 구글과 페이스북은 현재 페이지의 URL을 알아서 링크해주기 때문에 문제가 되지 않았는데 페이스북 API의 경우에는 현재 페이지의 URL을 직접 넣어주어야 했다. 그래서 이 부분을 동적으로 생성해주어야 하는데 Jekyll을 이용해서 간단하게 해결할 수 있었다.

## Jekyll로 현재 페이지 URL 만들기

 Jekyll 홈페이지를 참고해서 현재 페이지의 URL을 얻어 올 수 있는 Variable이 page와 site라는 것을 알 수 있었다. page는 단어 그대로 현재 페이지에 대한 정보를 담고 있고 site에는 이 홈페이지에 대한 정보가 담겨져 있다. 그래서 page를 통해 URI를 얻어 올 수 있고, site를 통해 이 사이트의 주소를 알수가 있어서 조합하면 현재 페이지의 주소를 알아낼 수가 있다.
 <pre>
 href="{{ page.url | prepend: site.url }}"
 </pre>

[구글]: https://developers.google.com/+/web/share/
[트위터]: https://about.twitter.com/ko/resources/buttons
[페이스북]: https://developers.facebook.com/docs/plugins?locale=ko_KR#like-share-send
