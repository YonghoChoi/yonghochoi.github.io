---
layout: post
title:  "Jekyll을 사용하여 Github에 블로그 만들기"
date:   2015-12-23 16:44:00 +0900
---
> Github에서 page automatic generator 기능을 통해 간편하게 블로그 페이지를 작성할 수 있도록 제공한다. 이를 이용해서 깔끔하고 본인 취향에 맞는 블로그를 개설할 수가 있다. <br /> 관련 내용은 [Github의 설명 페이지]를 참고해서 제작했다. 설명에 따르면 이 블로그 페이지는 공개적인 페이지라서 패스워드나 개인정보를 Repository에 저장하면 안된다고 경고한다. 심지어 repository가 private라도 이 블로그를 통해 공개될 수 있으므로 주의해야 한다.

## Github에 Repository 생성

시작은 간단하게 “블로그명.github.io” 형태로 repository를 생성한다. 그 다음 이 repository의 settings로 들어간 후 Launch automatic generator를 선택하여 블로그 템플릿을 지정하면 해당 템플릿이 적용된 index.html 파일이 생성된다.

그 이후부터는 repository를 clone 해서 작업한 후 github에 변경 내용을 반영하는 식으로 발전시켜 나가면 된다.

Github page는 Jekyll을 지원하는데 모든 페이지에 복사할 필요 없이 쉽게 헤더와 푸터를 만들어준다.  repository 내의 특별한 이름을 가진 branch에 push를 할 때 Jekyll이 동작한다. User 페이지를 제작하는 경우에는 master를 사용하면 되고, ProjectPage일 경우에는 gh-pages라는 branch를 사용하면 된다.


## 로컬에 [Jekyll] 설치

Github에서는 문제를 사전에 발견하는데 도움을 줄 수 있도록 로컬에 Jekyll을 설치하길 강력히 권장하고 있다.

Jekyll은 윈도우에서 공식적으로 지원하지는 않는다. 윈도우에 설치 하려면 [Jekyll Windows 설치 가이드]를 참고한다. 가이드 순서대로 진행하면 Ruby와 Jekyll을 설치하게 되는데 Ruby는 최소한 2.0.0 이상의 버젼이어야 한다.

설치 완료 후 추가로 필요한 package들을 설치 한다.
  <pre>
  gem install github-pages
  gem install bundler

  github-pages versions
  </pre>

bundler는 루비 프로젝트를 위한 일관된 환경을 제공(gem들과 dependency 관리)해준다. 이제 Jekyll을 사용하여 간단하게 웹 페이지를 만들어볼 수 있다.
  <pre>
  jekyll new myblog
  cd myblog
  jekyll serve
  </pre>
  * jekyll new myblog : myblog라는 이름으로 파일 생성
  * jekyll serve : 페이지가 있는 디렉토리에서 명령을 수행하면 서버가 작동한다.

Bundle을 사용하여 dependency를 관리하려면 Gemfile이 필요하므로 repository root에 Gemfile을 작성해준다.
  <pre>
  source 'https://rubygems.org'
  gem 'github-pages'
  </pre>

bundle install 명령을 수행하면 dependency들이 설치된다.
  <pre>
  bundle install
  </pre>

Github 페이지에서 Gemfile이 동작하도록 하려면 GIthub 에 push를 해준다. (bundle install 후 생성되는 Github.lock 파일과 함께 push) 이제 bundle exec jekyll serve 명령으로 서버를 구동시킬 수가 있는데 python이 설치되어 있지 않으면 liquid exception 이 발생한다. 파이썬 3.x 버젼으로 설치 후 수행하면 4000번 port로 서버가 구동된다.
  <pre>
  bundle exec jekyll serve
  </pre>
  * 위에서 처럼 bundle exec 를 사용하지 않고도 jekyll serve 명령만으로도 수행이 가능한데 gem을 실행 시킬 때 bundle을 사용하지 않으면 잘 작동하는 것 처럼 보이지만 그렇지 않을 수도 있다고 경고한다. dependency 때문에 프로그램이 괴로워할 수도 있다는 것이다.

dependency가 bundler를 통해 관리되고 있으므로 연관된 모듈들이 업데이트 되더라도 bundle update 명령을 통해 쉽게 최신 버젼으로 유지할 수가 있다.
  <pre>
  bundle update
  gem update github-pages (opt)
  </pre>

Jekyll은 대부분의 설정을 \_config.yml 파일을 통해 설정한다.

## \_posts 폴더

블로그 포스트들이 존재하는 폴더이고, Markdown 이나 HTML 문서가 될 수 있다. 모든 포스트들은 YAML Front Matter(페이지 최상단에 “---”으로 시작하는 코드)를 포함해야 정상적으로 동작한다.

\_posts 폴더에 파일을 추가 할 때는 파일 이름이 중요한데 Jekyll는 아래의 형식을 따를 것을 요구한다.
  <pre>
  YEAR-MONTH-DAY-title.MARKUP
  ex) 2015-12-23-first-page.md/html
  </pre>

[Github의 설명 페이지]: https://help.github.com/categories/github-pages-basics/
[Jekyll]: http://jekyllrb.com/docs/posts/
[Jekyll Windows 설치 가이드]: http://jekyll-windows.juthilo.com
