---
title: 윈도우10 로컬에서 Jekyll Github Page 구동하기.
categories:
- Github

tags:
- Jekyll
- Window10
- Github Page

photos:
- https://jekyllrb-ko.github.io/img/logo-2x.png
---

<!-- 하고 싶은말 -->

## 노트북에서도 포스팅을 할 수 있게 세팅하며 내용을 정리 했습니다.

---

`이미 Gemfile이 갖춰졌다는 가정하에 진행 합니다.`
#### 만약 Gemfile이 갖춰지지 않거나, 플러그인 오류가 날시 조치가 따로 필요합니다.
##### 저의 경우 Gemfile을 아래와 같이 세팅 했습니다.

```
#gem 'github-pages', group: :jekyll_plugins
#gem 'jekyll-admin', group: :jekyll_plugins

gem 'jekyll'
gem 'jekyll-feed'
gem 'jekyll-sitemap'
gem 'jekyll-seo-tag'
gem 'jemoji'
gem 'jekyll-readme-index'
gem 'jekyll-paginate', group: :jekyll_plugins
gem 'wdm', '>= 0.1.0'
```

* 사용하시는 지킬 테마에 따라서 필요한 플러그인이 다를것 입니다.
* 그 부분은 오류메시지를 보시면서 추가 하시기 바랍니다.

---

#### 제일먼저 루비를 설치합니다.

* [루비 설치 공식홈페이지](https://rubyinstaller.org/downloads/)
* With DEVKIT을 받아 설치 합시다.
* 저의 경우 높은 버전을 받으면 에러가 발생 했습니다.
* `Ruby+Devkit 2.5.7-1 (x64)` 을 설치 했습니다.
* 저는 설치 할때 설정에서 `UTF-8` 인코딩 설정을 했습니다.
    * 나중에 인코딩 에러가 날경우, chcp 65001 명령어를 입력해주면 해결된다고 합니다.


####  Start Command Prompt with Ruby를 실행 후 Bundler 를 설치합니다.
> gem install bundler


###### 만약 Gemfile이 없거나, 뭔지도 몰라요 라면
> gem install jekyll  
gem install minima  
gem install bundler  
gem install jekyll-feed  
gem install tzinfo-data  

`기본적인 패키지를 다 미리 다운 받아 둡시다`


#### 파워쉘을 실행 시킨뒤 깃허브 페이지를 Clone한 폴더로 갑니다.
> C:\Users\ybook\source\Github\Github_Blog\UnluckyJung.github.io


* `bundle install` 명령어를 입력해 필요한 gem들을 설치합시다.
    * Gemfile이 있으시다면, 정상적으로 문제 없이 설치가 될것입니다.
    * 만약 Gemfile이 없으시다면 바로 `jekyll serve` 을 입력해 실행 시켜 봅니다.

* `bundle install` 이후 `bundle exec jekyll serve` 을 입력해 실행 시킵니다.

#### 페이지를 확인해봅니다.
* [http://localhost:4000/](https://jekyllrb-ko.github.io/docs/quickstart/)

---


#### 복잡해 보이지만, Gemfile이 있으시다면 단 3번의 명령어로 모든 작업을 마칠 수 있습니다.
> gem install bundler  
bundle install  
bundle exec jekyll serve

[레퍼런스](https://jekyllrb-ko.github.io/docs/quickstart/)