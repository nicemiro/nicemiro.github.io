---
title: Github Page 만들기
titleEn: Creating a Github Pages for mac
author: BabyK
date: 2022-03-11
category: Jekyll
layout: post
---

### 환경설정 (Environment Setup)
#### Ruby 설치  
맥은 기본적으로 Ruby가 설치되어 있음.

```terminal
babyK@macbook ~ % ruby -v
ruby 3.1.1p18 (2022-02-18 revision 53f5fc4236) [x86_64-darwin21]
babyK@macbook ~ % gem -v
3.3.7
```
<br>
hombrew를 사용하여 최신버젼으로 설치
```terminal
babyK@macbook ~ % brew install ruby
```
<br>
#### bundler, jekyll 설치
Jekyll은 정적 사이트 빌더 (Static site builder)로 [Liquid][1] language를 사용하여 만들어졌다.  
[Liquid][1] 태그와 문법이 기존의 HTML마크업 언어와는 조금 다르지만 사용하기 편리하고 가독성이 꽤 괜찮은 듯.  
Github의 공동 설립자가 만들었으므로 당연하게도 Github page에 default로 적용되어 사용되고 있다.  

<br>
Bundler는 node js의 npm 같은 패키지 매니져인듯.
> 'Bundler provides a consistent environment for Ruby projects by tracking and installing the exact gems and versions that are needed'  
<br>

**Bundler install**
```terminal
babyK@macbook ~ % gem install bundler
```

**Jekyll install**
```terminal
babyK@macbook ~ % gem install jekyll bundle
```

**Jekyll version check**
```terminal
babyK@macbook ~ % jekyll -v
```
* 'GemNotFound' 에러 발생시 'bundle' or 'bundle install'을 입력. GemFile 안에 등록된 gem 들이 설치됨.

<br>

**Jekyll 폴더 생성**
```terminal
babyK@macbook gitRepo % jekyll new newFolder
```


[1]: https://shopify.github.io/liquid/