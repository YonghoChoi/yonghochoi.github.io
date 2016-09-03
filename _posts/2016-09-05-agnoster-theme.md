---
layout: post
title:  "Agnoster 폰트 적용"
date:   2016-09-05 16:30:00 +0900
tags: [setting]
---

Agnoster 테마를 적용하면 터미널에 글자가 깨져서 출력이 되는데 이를 방지하기 위해 [폰트]를 다운받아 설치한다.

1. zip 파일로 압축이 되어 있는데 간단하게 더블클릭하여 압축을 해제한다.

2. 압축을 해제한 디렉토리 안의 UbuntuMono 디렉토리로 이동한다.

3. *.ttf 파일들을 설치한다. 이것도 간단하게 더블클릭 하면 설치할 수 있다.

4. 터미널의 Preferences 창으로 들어가서 Font를 변경한다.

   * iTerm을 사용하는 경우 Preferences > Profile 창의 Font들을 변경해주면 된다.

5. Oh-My-ZSH가 설치되어있지 않은 경우 설치를 진행한다. ([oh-my-zsh 링크] 참고)

6. 홈디렉토리에서 .zshrc 파일을 열어서 테마를 변경한다.

   ```shell
   ZSH_THEME="agnoster"
   ```

7. 터미널을 재시작하면 설치 완료!

   ​

[폰트]: https://github.com/powerline/fonts/archive/master.zip
[oh-my-zsh 링크]: https://github.com/robbyrussell/oh-my-zsh
