# 용호의 블로그

## 설치

 jekyll 기반으로 되어 있기 때문에 jekyll 설치가 선행되어야 합니다.

### jekyll 설치

 jekyll은 구동을 하기 위해 Ruby와 python 등의 Requirements가 있습니다. [jekyll Installation] 페이지를 참고하여 설치하기 바랍니다. 아래는 Requirements 들이 준비된 후 gem을 이용하여 jekyll을 설치하고, 기본 템플릿 페이지를 구성하는 방법입니다.
```
$ gem install jekyll
$ jekyll new myblog
$ cd myblog
```

### bundler 설치

bundle 명령을 사용하여 서버를 구동시키기 위해서는 bundler의 설치가 필요합니다.
```
$ gem install bundler
```

### jekyll 구동

jekyll을 구동하면 기본 4000 port로 서버가 오픈됩니다.

```
$ bundle exec jekyll serve
```

구동이 안되는 경우 경로에 한글이 있지는 않은지 확인해주세요. (한글 포함시 error 발생)

[jekyll Installation]: https://jekyllrb.com/docs/installation/
