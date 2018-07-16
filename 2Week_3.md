# 2Week_3

## 06.14(목)

> 09:00~09:50

* Codewars || 자습

> 10:00~12:00

#### Rails Tutorial Projects

* `rails new zero_app` - c9, ubuntu OS

  1. `$ sudo apt-get update`
  2. zsh 설치 : `$ sudo apt-get install zsh`
  3. zsh 변경 : `$ sudo chsh -s /usr/bin/zsh` , `$ reset`, `$ zsh`
  4. zsh customizing : `$ sudo sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
  5. zsh 테마 변경 : `$ vi ~/.zshrc `  수정 후 `$ source ~/.zshrc`

  ```shell
  ZSH_THEME="THEME_NAME" # 테마이름 넣기
  ```
  6. zsh plugins 설치 : `$ vi ~/.zshrc`  수정 후 `$ source ~/.zshrc`

     ``` shell
     plugins=(
       git
       rails
       gem
       rvm
     )
     ```

  7. rails(v 5.1.6) 설치 :  `$ gem install rails -v 5.1.6`

     -----------------------------------------

  8. 새 터미널에서 project 생성 : `$ rails new zero_app`

  9. `rails server -b $IP -p $PORT`

  10. `rs` 명령어 추가 : `$ echo "alias 'rs'='rails s -b 0.0.0.0 -p 8080'"` 수정 후 `$ source ~/.zshrc`

      ( `ps aux | grep puma` )


>13:00~14:50

* Ruby On Rails - What?

* Ruby On Rails - How?

* `zero_app` 

  * 기본 routing

    * zero_app/config/`routes.rb`

      ```ruby
      Rails.application.routes.draw do
        get '/', to: 'application#hello'
      end
      ```

    * zero_app/app/controllers/`application_controller.rb`

      ```ruby
      class ApplicationController < ActionController::Base
        protect_from_forgery with: :exception
        def hello 
          puts request.remote_ip.class
          
          render html:  "#{Geocoder.address(request.remote_ip)}에서 접속한 IP는 #{request.remote_ip}입니다."
        end
      end
      ```

    * zero_app/`Gemfile` - `gem 'geocoder'` 추가후 `$ bundle`

      

  * gem geocoder 사용

  * git - bitbuket 연결

    *  `git config --global user.name 'USERNAME'`
    *  `git config --global user.email 'USER_EMAIL'`
    *  bitbuket에 SSH 추가
       1. c9 터미널에서 `$ cat ~/.ssh/id_rsa.pub` 후 내용 복사
       2. bitbuket -> View Profile -> Settigns -> SSH Keys -> `Add key`버튼 클릭
       *  복사한 내용 붙이기

      *  `$ git remote add origin SSH주소`
      *  git config 확인 : `$ cat .git/config`
      *  `$ git push -u origin master`

> 15:00~15:50

#### `zero_app` Heroku 배포

* Heroku 가입

* `Gemfile`수정 - heroku에서 `sqlite3` gem 사용 불가, production때는 `pg` gem 사용

  ```ruby
  source 'https://rubygems.org'
  
  git_source(:github) do |repo_name|
    repo_name = "#{repo_name}/#{repo_name}" unless repo_name.include?("/")
    "https://github.com/#{repo_name}.git"
  end
  
  
  gem 'rails', '~> 5.1.6'
  gem 'puma', '~> 3.7'
  gem 'sass-rails', '~> 5.0'
  gem 'uglifier', '>= 1.3.0'
  gem 'coffee-rails', '~> 4.2'
  gem 'turbolinks', '~> 5'
  gem 'jbuilder', '~> 2.5'
  gem 'geocoder'
  
  group :development, :test do
    gem 'sqlite3' # development, test로 옮기기
    gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]
    gem 'capybara', '~> 2.13'
    gem 'selenium-webdriver'
  end
  
  group :production do # production에만 쓸 pg gem 추가 
    gem 'pg' 
  end
  
  group :development do
    gem 'web-console', '>= 3.3.0'
    gem 'listen', '>= 3.0.5', '< 3.2'
    gem 'spring'
    gem 'spring-watcher-listen', '~> 2.0.0'
  end
  
  gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]
  ```

* heroku 로그인 : `$ heroku login`

* heroku에 ssh key 등록 : `$ heroku keys:add`

* heroku 생성(이름 자동 생성) : `$ heroku create`

* heroku에 push :`$ git push heroku master`

> 16:00~18:00

#### rails new toy_app

* `rails new toy_app`

  * git ~/.gitconfig -> editor: vim

* git 추가 && 이전 commit 수정`$ git commit --amend`

* bitbuket 연결하기 `$ git remote add origin SSH주소` && `$ git push -u origin master`

* Controllers & Views

  * `$ rails generate controller home` -> HomeController 생성

    * toy_app/`Gemfile` -> `gem 'artii'` 설치

    * toy_app/app/controllers/`home_controller.rb`

    ```ruby
    class HomeController < ApplicationController
      def index 
        fonts = ['3-d', 'speed', 'big', 'acrobatic']
        text = ['Hack Your Life', 'HELLO', 'Happy Hacking', 'Winter is coming']
      
        a = Artii::Base.new(font: fonts.sample)
        @output = a.asciify(text.sample)
      end
        
      def hello
        render html: 'hello'
      end
        
      def lotto 
        @numbers = [*1..45].sample(6).sort
      end
    end
    ```

    * toy_app/config/`routes.rb`

    ```ruby
    Rails.application.routes.draw do
      root 'home#index'
      get '/hello', to: 'home#hello'
      get '/lotto', to: 'home#lotto'
    end
    ```

    * toy_app/app/views/home/`index.html.erb`

    ```erb
    <pre><%=@output%></pre>
    
    <ul>
      <li><a href='/hello'>hello</a></li>
      <li><a href='/lotto'>lotto</a></li>
    </ul>
    ```

    * toy_app/app/views/home/`index.html.erb`

    ```erb
    <h1>로또 번호들</h1>
    <%= @numbers %>
    ```

* `toy_app` heroku 배포

  0. `$ heroku login` `$ hroku keys:add` 생략 

  1. `$ git add . && git commit -m 'Edit 06/14'` - git commit 하기
  2. `$ heroku create`
  3. `$ git push heroku master`