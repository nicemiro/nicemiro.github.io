---
title: Jekyll 설치
titleEn: Jekyll. install and usage for Mac
author: BabyK
date: 2022-03-11
category: Web
layout: post
# published: true
---

### Jekyll 환경설정 (Environment Setup)
#### Ruby 설치  
맥은 기본적으로 Ruby가 설치되어 있음.

```terminal
babyK@macbook ~ % ruby -v
ruby 3.1.1p18 (2022-02-18 revision 53f5fc4236) [x86_64-darwin21]
babyK@macbook ~ % gem -v
3.3.7
```  

hombrew를 사용하여 최신버젼으로 설치
```terminal
babyK@macbook ~ % brew install ruby
```
<br>
#### bundler, jekyll 설치
[Jekyll][2]은 Ruby언어를 사용하여 만들어진 정적 사이트 생성 툴(Static site builder)이며 [Liquid][1] 문법을 사용하여 웹페이지를 만들 수 있게 해준다.  
Github의 공동 설립자가 만들었으며 Github page에 default로 적용되어 사용되고 있다.    
정적 페이지이기 때문에 사용자가 미리 작성해 놓지 않으면 보여줄 수 없는 부분들도 있고 수정하다보면 차라리 이럴꺼면 HTML과 Javascript를 사용해서 그냥 새로 만드는게 낫지 않을까 하는 생각이 들때도 있지만 다른 유저들이 만들어 놓은 수많은 테마들을 적용하고 거기에서 또 사용자의 입맛에 맞게 조금씩 수정해 가는 재미가 있다.

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
* `GemNotFound` 에러 발생시 'bundle' or 'bundle install' 입력. GemFile 안에 등록된 gem 들이 설치됨.

<br>

**Jekyll 폴더 생성**  
gitRepo 폴더 밑에 testPage로 생성.
```terminal
babyk@Ks-macbook gitRepo % jekyll new testPage
Running bundle install in /Users/babyk/workspace/gitRepo/testPage...
  Bundler: Fetching gem metadata from https://rubygems.org/..........
  Bundler: Resolving dependencies...
  Bundler: Using public_suffix 4.0.6
  Bundler: Using bundler 2.3.8
  Bundler: Using colorator 1.1.0
  Bundler: Using concurrent-ruby 1.1.9
  Bundler: Using eventmachine 1.2.7
  Bundler: Using http_parser.rb 0.8.0
  Bundler: Using ffi 1.15.5
  Bundler: Using forwardable-extended 2.6.0
  Bundler: Using rb-fsevent 0.11.1
  Bundler: Using liquid 4.0.3
  ...
  Bundler: Bundle complete! 7 Gemfile dependencies, 31 gems now installed.
  Bundler: Use `bundle info [gemname]` to see where a bundled gem is installed.
New jekyll site installed in /Users/babyk/workspace/gitRepo/testPage.
babyk@Ks-macbook gitRepo %
```

<br>

**서버 start**  
처음 서버 실행시에는 'bundle exec jekeyll serve' 를 입력하여 실행할 수 있고 이후부터는 'jekyll serve'만 입력해도 실행된다.
```terminal
babyk@Ks-macbook testPage % bundle exec jekyll serve
Configuration file: /Users/babyk/workspace/gitRepo/testPage/_config.yml
            Source: /Users/babyk/workspace/gitRepo/testPage
       Destination: /Users/babyk/workspace/gitRepo/testPage/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 0.343 seconds.
 Auto-regeneration: enabled for '/Users/babyk/workspace/gitRepo/testPage'
                    ------------------------------------------------
      Jekyll 4.2.2   Please append `--trace` to the `serve` command
                     for any additional information or backtrace.
                    ------------------------------------------------
/usr/local/lib/ruby/gems/3.1.0/gems/jekyll-4.2.2/lib/jekyll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)
	from /usr/local/lib/ruby/gems/3.1.0/gems/jekyll-4.2.2/lib/jekyll/commands/serve/servlet.rb:3:in `<top (required)>'

babyk@Ks-macbook testPage %
```
*`cannot load such file -- webrick (LoadError)`  
Gemfile 안에 webrick gem 이 없어서 발생하는 문제로 'bundle add webrick' 입력.

```terminal
babyk@Ks-macbook testPage % bundle add webrick
Fetching gem metadata from https://rubygems.org/.........
Resolving dependencies...
Fetching gem metadata from https://rubygems.org/.........
Resolving dependencies...
Using public_suffix 4.0.6
Using bundler 2.3.8
Using colorator 1.1.0
Using concurrent-ruby 1.1.9
Using eventmachine 1.2.7
Using http_parser.rb 0.8.0
Using ffi 1.15.5
Using forwardable-extended 2.6.0
Using rb-fsevent 0.11.1
Using rexml 3.2.5
Using liquid 4.0.3
Using mercenary 0.4.0
Using rouge 3.28.0
Using webrick 1.7.0
...
babyk@Ks-macbook testPage %
```
  
Gemfile 을 열어보면 아래와 같이 마지막 줄에 webrick gem이 추가된 것을 확인할 수 있다.

```ruby
# frozen_string_literal: true

source "https://rubygems.org"

git_source(:github) { |repo_name| "https://github.com/#{repo_name}" }
gem "jekyll"
gem 'jekyll-feed'
gem 'jemoji'
gem 'webrick'

# gem "rails"
```

다시 서버 스타트.
```terminal
babyk@Ks-macbook testPage % jekyll serve
Configuration file: /Users/babyk/workspace/gitRepo/testPage/_config.yml
            Source: /Users/babyk/workspace/gitRepo/testPage
       Destination: /Users/babyk/workspace/gitRepo/testPage/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 0.33 seconds.
 Auto-regeneration: enabled for '/Users/babyk/workspace/gitRepo/testPage'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

<br>
**웹페이지 확인**  
디폴트 주소는 http://localhost:4000가 사용되며 포트변경시 'jekyll serve --port 0000' 와 같이 변경할 포트번호를 입력한다.

<img src="/img/minimaPic.png" >

<br>
기본테마인 minima 가 적용되어 있다.  
폰트 가독성과 화면구성이 꽤 훌륭한 테마라서 사이드메뉴 하나만 추가하고 사용해도 괜찮을 것 같다.  
Jekyll을 사용해 로컬에서 생성한 웹페이지를 Github 사이트에 올려 GithubPage에서 띄우면 곧바로 훌륭한 홈페이지가 만들어진다.

<br>
#### bundler 에 대한 추가 설명.  
bundle install 커맨드는 Gemfile 안에 리스트업 되어 있는 버전의 Gem들을 설치해준다.
설치하면서 각 버전들의 dependency를 체크하고 Gemfile.lock 파일을 생성해주는데
우리가 bundle exec jekyll serve 를 입력하여 jekyll을 실행하면 이 Gemfile.lock 에 명시된 Gem(라이브러리)의 버젼을 사용해서 실행하게 된다.
이렇게 하면 작성된 페이지를 다른 환경에 (github 서버라던가) 띄워야 할때도 동일한 Gemfile.lock 파일을 사용하여 버전 충돌을 방지할 수 있다.

[1]: https://shopify.github.io/liquid/
[2]: https://jekyllrb.com/

