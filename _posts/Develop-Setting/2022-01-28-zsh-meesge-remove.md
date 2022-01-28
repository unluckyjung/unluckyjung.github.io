---
title: zsh 불필요한 메시지 제거하기 If the above didn't help or you want to skip the verification of
date: 2022-01-28-20:00
categories:
- Develop-Setting

tags:
- zsh
- Tool

---

## zsh 실행시 나오는 불필요한 메시지를 제거해봅니다.
> 에러메시지: If the above didn't help or you want to skip the verification of...


---

## Goal
- zsh 실행시 나오는 불필요한 메시지를 제거해봅니다.

---

## 개요
- zsh 설정을 바꾸다가, 무언가를 손댄 이후로 아래와 같은 메시지가 **iterm** 을 실행시킬때마다 나오게 되었습니다.

```console
[oh-my-zsh] If the above didn't help or you want to skip the verification of
[oh-my-zsh] insecure directories you can set the variable ZSH_DISABLE_COMPFIX to
[oh-my-zsh] "true" before oh-my-zsh is sourced in your zshrc file.
```
초기실행시 실수로나마 `y` 가 아닌 다른것을 누르면 zsh가 작동이 안되어서, 불편함을 느끼고 해당 메시지를 제거하기로 했습니다.

---

## 해결방법

메시지를 잘 읽어봅시다.  

> sourced 전에 ZSH_DISABLE_COMPFIX 를 true로 바꾸어 주세요!  


```console
vi ~/.zshrc
```

zsh 설정을 들어가 줍니다.  



```console
plugins=(git)
ZSH_DISABLE_COMPFIX="true"
source $ZSH/oh-my-zsh.sh
```

대충 적절한 위치에 `ZSH_DISABLE_COMPFIX="true"` 를 넣어줍니다.

```console
source ~/.zshrc
```

재구동시, 메시지가 다시 표시 안되는것을 확인할 수 있습니다.

---

## Reference
- [ohmyzsh issue](https://github.com/ohmyzsh/ohmyzsh/issues/6835#issuecomment-390216875)