# 4week_2

## 06.26(화)

### 1. Password

[SHA256 해시 생성기](http://www.convertstring.com/ko/Hash/SHA256)

#### 1.1 Password setting

**has_secure_password**[[doc]](http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html)

- `has_secure_password` 메서드 사용 시 자동으로 추가되는 유효화 검증들
  - 생성 시 암호가 있어야 함
  - 암호 길이는 72 bytes 보다 작거나 같아야 함
  - 비밀번호 확인 ( `password_confirmation` 속성 사용 )

- `user.rb`

  ```ruby
  class User < ApplicationRecord
  				...
    has_secure_password
  end
  ```

**기존 table 에 column 추가**

```shell
$ rails g migration add_password_digest_to_users password_digest:string
$ rails db:migrate
```

> `password_digest` 명칭은 규약

#### 1.2 암호화 gem

**gem 'bcrypt'**

- gem 추가 후 `bundle`

- `user_test.rb`

  ```ruby
  class UserTest < ActiveSupport::TestCase
    def setup
      @user = User.new name: "Example User", email: "user@example.com", 
        				 password: "likelion", password_confirmation: "likelion"
    end
      
      
      					...
          
      
          
    test 'password should be present (nonblank)' do
      @user.password = @user.password_confirmation = ' ' * 6
      assert_not @user.valid?
    end
    
    test 'password should have a minimum length' do
      @user.password = @user.password_confirmation = 'a' * 5
      assert_not @user.valid?
    end
  end
  ```

- `user.rb`

  ```ruby
  class User < ApplicationRecord
  								...
    has_secure_password
    
    validates :password, presence: true, length: { minimum: 6 }
  end
  ```

**Authenticate**

```shell
$ u.authenticate 'password'
=> 비밀번호가 맞으면 사용자 정보 모두 출력(틀리면 false 출력)
$ !!u.authenticate('password')
=> true
```

- `developement.rb`

  ```ruby
  Rails.application.configure do
    config.web_console.whiny_request = false
      								...
  end
  ```

**Web_console setting**

- `application.html.erb`  : 아래 debug 코드 추가

```erb
...
<body>
  <%= render 'layouts/header' %>
  <div class="container">
    <%= yield %>
  </div>
  <%= render 'layouts/footer' %>
    
  <%= debug(params) if Rails.env.development? %>
</body>
...
```

```shell
$ git add .
$ git commit -m "finish setting"
$ git push
$ git checkout master
$ git merge modeling-users

$ heroku login
$ git push heroku master
```

### 2. Sign-up

#### 2.1 사전 준비

```shell
$ git checkout -b sign-up
$ rails g controller Users new
```

- `routes.rb`

  ```ruby
  Rails.application.routes.draw do
    get '/signup', to: 'users#new'
    resources :users
  					...
  end
  ```

```shell
$ rails db:drop
$ rails db:migrate
$ rails c

$ u = User.new
$ u.name = 'harry'
$ u.email = 'harry@likelion.org'
$ u.password = '123456'
$ u.password_confirmation = '123456'
$ u.save
$ u

이런식으로 몇개 더 생성해 본다.
```

#### 2.2 회원가입 양식 만들기

- `new.html.erb`

  ```erb
  <% provide(:title, 'Sign Up') %>
  <h1>Sign Up</h1>
  <div class="row">
    <div class="col-md-6 offset-md-3">
      <%= form_for @user do |f| %>
        <%= f.label :name %>
        <%= f.text_field :name %>
        
        <%= f.label :email %>
        <%= f.email_field :email %>
        
        <%= f.label :password %>
        <%= f.password_field :password %>
        
        <%= f.label :password_confirmation %>
        <%= f.password_field :password_confirmation %>
        
        <%= f.submit "Create Account", class: "btn btn-primary" %>
      <% end %>
    </div>
  </div>
  ```

- `users_controller.rb`

  ```ruby
  class UsersController < ApplicationController
    def new
      @user = User.new
    end
  end
  ```

- `custom.scss` : 아래 코드로 전체 교체

  ```scss
  @import "bootstrap";
  
  /* mixins, variables, etc. */
  
  $gray-medium-light: #eaeaea;
  
  @mixin box_sizing {
    -moz-box-sizing:    border-box;
    -webkit-box-sizing: border-box;
    box-sizing:         border-box;
  }
  
  /* miscellaneous */
  
  .debug_dump {
    clear: both;
    float: left;
    width: 100%;
    margin-top: 45px;
    @include box_sizing;
    -moz-box-sizing:    border-box;
    -webkit-box-sizing: border-box;
    box-sizing:         border-box;
  }
  /* mixins, variables, etc. */
  
  $gray-medium-light: #eaeaea;
  
  @mixin box_sizing {
    -moz-box-sizing:    border-box;
    -webkit-box-sizing: border-box;
    box-sizing:         border-box;
  }
  
  /* miscellaneous */
  
  .debug_dump {
    clear: both;
    float: left;
    width: 100%;
    margin-top: 45px;
    @include box_sizing;
    -moz-box-sizing:    border-box;
    -webkit-box-sizing: border-box;
    box-sizing:         border-box;
  }
  
  /* universal */
  
  body {
    padding-top: 100px;
  }
  
  section {
    overflow: auto;
  }
  
  textarea {
    resize: vertical;
  }
  
  .center {
    text-align: center;
    h1 {
      margin-bottom: 10px;
    }
  }
  
  /* typography */
  h1, h2, h3, h4, h5, h6 {
    line-height: 1;
  }
  
  .jumbotron {
    h1, h2{
      text-align: left;
    }
  }
  
  h1 {
    font-size: 3em;
    letter-spacing: -2px;
    margin-bottom: 30px;
    text-align: center;
  }
  
  h2 {
    font-size: 1.2em;
    letter-spacing: -1px;
    margin-bottom: 30px;
    text-align: center;
    font-weight: normal;
    color: #777;
  }
  
  p {
    font-size: 1.1em;
    line-height: 1.7em;
  }
  
  
  /* header */
  #logo {
    text-transform: uppercase;
    letter-spacing: -1px;
    font-size: 1.7em;
    font-weight: bold;
    &:hover {
      color: #FFFFFF;
      text-decoration: none;
    }
  }
  
  /* footer */
  footer {
    margin-top: 45px;
    a {
      color: #555;
      &:hover {
        color: #222;
      }
    }
    small {
      float: left;
    }
    ul {
      float: right;
      list-style: none;
      li {
        float: left;
        margin-left: 15px
      }
    }
  }
  
  /* 전체 */
  
  textarea {
    resize: vertical;
  }
  
  .center {
    text-align: center;
    h1 {
      margin-bottom: 10px;
    }
  }
  
  /* forms */
  
  input, textarea, select, .uneditable-input {
    border: 1px solid #bbb;
    width: 100%;
    margin-bottom: 15px;
    @include box_sizing;
  }
  
  input {
    height: auto !important;
  }
  ```

  