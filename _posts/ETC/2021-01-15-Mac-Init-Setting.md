---
title: 맥린이의 맥 개발환경 세팅
date: 2021-01-15-22:35
categories: 
- ETC

tags:
- MAC

photos:
- /post_images/MacBook-Air.jpg

---

## 맥린이의 M1 맥북에어 개발 환경 세팅
> 맥북을 처음 사본 맥린이의 개발환경 세팅 과정을 기록해 봅니다.

---

## 전체적인 흐름
> https://subicura.com/2017/11/22/mac-os-development-environment-setup.html

---

## MAC 환경설정

### 한글에서도 백틱이 나오게하기
* https://ani2life.com/wp/?p=1753


### 스크린샷 저장 위치 변경

```console
defaults write com.apple.screencapture location ~/{경로}/ && killall SystemUIServer
```

* default**s** 이다. `이거 틀려서 왜 안되지 하고 있었음`
* 맨뒤에 `/` 도 넣어주어야한다.

---

## homebrew 설치
> [참고링크](https://ninanung0503.medium.com/apple-silicon-m1-mac%EC%97%90-homebrew-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-7b6c0d3aba08)

* **M1** 설치 방법 
    * 터미널 우측 클릭, 정보 가져오기, `Rosetta를 사용하여 열기` 체크

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"

brew help 
```

### brew 요약

* brew ~ : 커맨드 라인 프로그램 (c, java, python 같은..)
* brew cask ~ : GUI 프로그램 (Safari, Chrome, Word 같은..)

```
brew update : 홈브류 최신버전으로 업데이트
brew upgrade [프로그램명]: 홈브류에 설치된 프로그램 최선버전으로 업데이트
brew search [프로그램명] : 홈브류를 통해 설치 가능한 프로그램 찾기
```

* brew cask list : 홈브류에 설치된 그래픽을 통해 작업하는 프로그램 목록 (Safari, Chrom, Word와 같은 일반적인 앱)


```
brew install [프로그램명] --cask: 프로그램 설치
brew remove [프로그램명] --cask: 홈브류에 설치된 프로그램 삭제
brew cleanup : 업데이트 후 필요없는 이전 버전의 패키지 삭제 // 맥은 업데이트시 이전 버전이 계속 남아있다!
```

### cask 다운 받기
> Safari Chrome Word 와 같이 그래픽을 통해 작업하는 프로그램을 설치하게 해주는 패키지

```
brew install cask
```

### brew 업데이트하기
> brew 작업전 하는것을 추천

```
brew update
```

### brew cask install 문제 발생시
> `Calling brew cask install is disabled! Use brew install [--cask] instead.`

* [개발자 링크](https://github.com/ansible-collections/community.general/issues/1524#issuecomment-749945392) 를 참고한다.
    * **brew cask** 는 2.7.0 업데이트부터 명령이 비활성화 되었다. `2020-12-01` 부터 사용 불가.
    * `brew install --cask` 로 설치한다.
    * 그냥 `brew intall` 해도 되긴되더라.. 

---

## git 설치
* https://ifuwanna.tistory.com/189
* https://nillk.tistory.com/1

```
brew install -s git
git --version
```

* 기본적으로 git이 설치되어 있지만, **구버전**이다.
* 새로운 버전으로 받아준다.
    * 애플 터미널에서 버전 확인시 구버전으로 표시되지만, `iterm2` 에서는 최신버전으로 표시되고 있다.
    * 애플 터미널에서도 최신버전을 적용하고 싶다면, [두번째링크](https://nillk.tistory.com/1) 참고.

### GitKraken 설치
```
brew search GitKraken
brew install Gitkraken --cask
```

---


## iterm2, oh my zsh 설치
> [참고링크](https://tutorialpost.apptilus.com/code/posts/tools/mac-cli-with-iterm2-zsh/)

### iterm2 설치

* `m1`에는 기본적으로 **zsh** 환경으로 되어있다.
    * 따라서 **zsh**는 굳이 따로 설치 해주지 않아도 된다.

```
brew install iterm2
```


### iterm 한글 깨짐 방지
* `profile > text > unicode > from`을 NFC로 변경

### iterm 꾸미기
* https://jojoldu.tistory.com/428

### iterm 접근 권한 주기
> https://gitlab.com/gnachman/iterm2/-/wikis/fulldiskaccess

* `시스템 설정 > 보안 및 개인 정보 보호 > 개인 정보 보호 > 전체 디스크 접근권한`


---

## oh my zsh 설치, 터미널 꾸미기

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### oh my zsh 테마 바꾸기
* [참고링크1](https://medium.com/harrythegreat/oh-my-zsh-iterm2%EB%A1%9C-%ED%84%B0%EB%AF%B8%EB%84%90%EC%9D%84-%EB%8D%94-%EA%B0%95%EB%A0%A5%ED%95%98%EA%B2%8C-a105f2c01bec)

```
vi ~/.zshrc
```
* 설정파일로 접근한다.
* 상단에서 `ZSH_THEME="robyrussell"` 를 찾는다.
* `ZSH_THEME="agnoster"` 로 바꾸어준다.

```
source ~/.zshrc
```

* 해당 명령을 **꼭 실행** 해주어야 적용이된다.
* 이제 보면 기본폰트가 꺠져서 나올것이다. 새로운 폰트를 받아보자
  * 나의 경우 `D2 font` 를 선택했다.

---

### D2 font 다운받기
* https://github.com/naver/d2codingfont/releases/tag/VER1.3.2 로 간다.
* zip 파일을 다운받는다.
* 압축 해제 후 `D2Coding/D2Coding-Ver..........ttf` 파일을 실행시켜서 설치한다.

### D2 front 적용하기
* **iterm**를 킨다.
* `cmd + ,` 를 눌러서 환경설정에 들어간다
* `profile > text > Font` 에서 D2Coding로 바꿔준다.
* iterm을 껏다 킨다.

---

### Syntax Hightlihgt 적용하기

```
brew install zsh-syntax-highlighting
```
* brew 를 이용해서 설치한다.

```
source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```
* 플러그인을 적용한다.

> 이제 사용할 수 있는 명령은 초록색, 없는 명령은 분홍색으로 표시된다.


### zsh-completions, zsh-syntax-highlighting 설치하기
> 자동완성, 신텍스 하이라이팅을 지원해주는 플러그인 입니다.

- 공식문서 가이드를 따라갑니다.
- [zsh-completions](https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md)
- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md)

> 제가 사용하고 있는 Oh My Zsh 기준입니다.

```console
// 설치1
$ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

// 설치2
$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

```console

// 설정 열기
$ vi ~/.zshrc

// 변경
plugins=( 
    # other plugins...
    zsh-autosuggestions
    zsh-syntax-highlighting
)

// 적용
source ~/.zshrc
```

![image](https://user-images.githubusercontent.com/43930419/161419289-01467941-dbfb-4ade-92d8-c1272a9cfad4.png)

![image](https://user-images.githubusercontent.com/43930419/161419295-01c3e99b-4fe1-4a8f-a3b3-aebbcb89fa31.png)

- 적용하고 나면 사용하지 못하는 명령어는 **붉은색**, 추천 자동완성은 **회색**, 사용할수 있는 명령어는 **초록색**으로 표시됩니다.

---

## vim 테마 변경
> 필자의 경우 **dracula** 테마를 선택했다.


```console
// 테마 폴더를 만든다. (경로 같아야합니다.)
mkdir -p ~/.vim/pack/themes/start

// 테마 폴더로 이동
cd ~/.vim/pack/themes/start

// 테마 다운
git clone https://github.com/dracula/vim.git dracula
```

- 테마를 다운 받는다.

```console
vi ~/.vimrc
```

- 내려받은 테마를 적용하기 위해서, `vimrc`를 편집해준다.

```console
packadd! dracula
syntax enable
colorscheme dracula

set laststatus=2
set statusline=\ %<%l:%v\ [%P]%=%a\ %h%m%r\ %F\
set nu
```

- 위를 복사 붙여넣기 하고 wq하고 저장하고 나온다.
  - set 3개는 생략해도 된다.
- 이제 vi, vim을 킬때마다 테마가 적용된 것을 볼 수 있다.

### 참고링크
- https://draculatheme.com/vim
- https://minimin2.tistory.com/145

---

## 앱 설치

### chrome 설치
* itrems 실행.
* brew 를 이용해 설치한다.

```console
brew search chrome 
```

* 구글 크롬이 설치 가능한지 찾아본다.

```console
brew install google-chrome
```

* Cask 목록에 나온 `google-chrome`을 brew를 이용해 설치한다.
    * 앞에 cask를 붙여주지 않아도 된다. 그 이유는 위에서 설명했다.
    * 대신 뒤에 `--cask` 를 붙여줘도 된다.

```console
brew list --cask
```

---

### Visual Studio Code 설치

```
brew search visual studio code
brew install visual-studio-code --cask
```

---

## openJDK 설치하기
> 전체적인 내용을 담은 [링크](https://github.com/AdoptOpenJDK/homebrew-openjdk)

* JAVA8 버전을 설치하는것을 목표로합니다.

```
brew tap AdoptOpenJDK/openjdk
```

* `openjdk` 저장소를 다운받는다.
    * 여기서 tap이란? [링크](https://velog.io/@iamchanii/Brewfile%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%B4%EC%84%9C-%ED%8C%80-%EA%B0%9C%EB%B0%9C-%ED%99%98%EA%B2%BD-%EB%A7%8C%EB%93%A4%EA%B8%B0#tap-brew-cask)
    * 서드파티 저장소라고 생각하면 된다.
    * 해당 저장소를 tap에 추가할 수 있고, **install** 시 해당 tap을 이용할 수 있게 된다.
    * `brew tap` 명령을 통해서 추가된 탭목록을 확인할 수 있다.

```
$ brew install --cask <version>
brew install --cask adoptopenjdk8
```

* 버전을 openjdk8 으로 하여 다운받는다.

```
java -version
```

* 설치된 자바 버전을 확인한다.

---

## 인텔리제이 설치하기

### 홈페이지 가서 dmg 파일 다운받기
> https://www.jetbrains.com/ko-kr/idea/download/#section=mac

* `.dmg(Apple Silicon)` 버전으로 다운받는다.


### 플러그인 설치
> https://goddaehee.tistory.com/198

* `lombok`은 기본으로 설치 되어있다.
* `Rainbow Braket` 설치
* `Key Promoter X` 설치
    * 다시 맥북에서의 단축키를 익혀야 하니..
* `.ignore` 설치
    * https://steady-hello.tistory.com/m/44
* `codegGlance` 설치

### 인텔리제이 세팅
* 폰트 D2로 변경
* 대소문자 구분없이 자동완성 하게 설정
* [한글 설정](https://unluckyjung.github.io/tool/2020/12/02/Intellij-UTF8-setting/)
* [파일크기, 메모리 설정](https://unluckyjung.github.io/tool/2021/01/14/Intellij-Edit-FileSize/)


---

## 도커, Mysql 워크벤치 설치
- [M1 Docker, Workbench 설치](https://unluckyjung.github.io/develop-setting/2021/03/27/M1-Docker-Workbench-Install/)

---

## 참고링크
* 전체 흐름
    * https://subicura.com/2017/11/22/mac-os-development-environment-setup.html

* 툴 다운
    * https://blog.tadadakcode.com/6
    * https://treasurebear.tistory.com/63

* 홈브루
    * https://whitepaek.tistory.com/3
