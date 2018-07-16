# 4week_1

## 06.25(월)

### 1. Sample_app Layout setting

#### 1.1 emmet

- [emmet](https://docs.emmet.io/cheat-sheet/) 을 이용해 navbar 세팅
- `application.html.erb`

```erb
<!DOCTYPE html>
<html>
<head>
...
</head>

<body>
  <header class="navbar navbar-fixed-top navbar-inverse">
    <div class="container">
      <%= link_to "Harry's blog", '#', id: 'logo' %>
      <nav>
        <ul class="nav navbar-nav navbar-right">
          <li><%= link_to "Home", '#' %></li>
          <li><%= link_to "Help", '#' %></li>
          <li><%= link_to "Sign in", '#' %></li>
        </ul>
      </nav>
    </div>
  </header>
  <div class="container">
    <%= yield %>
  </div>
</body>
</html>
```

#### 1.2 [Install Bootstrap Ruby Gem 4](https://github.com/twbs/bootstrap-rubygem)

- `gemfile`

```ruby
# add
gem 'bootstrap'
gem 'jquery-rails'
```

- `app/assets/stylesheets/custom.scss` 파일 생성

```scss
@import "bootstrap";
```

- `app/assets/javascripsts/application.js` *(순서 유의)*

```js
//= require rails-ujs
//= require turbolinks
//= require jquery3
//= require popper
//= require bootstrap-sprockets
//= require_tree .
```

#### 1.3 render

- `app/views/layouts/_header.html.erb`

```erb
<header class="navbar navbar-fixed-top navbar-inverse">
  <div class="container">
    <%= link_to "Harry's blog", '#', id: 'logo' %>
    <nav>
      <ul class="nav navbar-nav navbar-right">
        <li><%= link_to "Home", '#' %></li>
        <li><%= link_to "Help", '#' %></li>
        <li><%= link_to "Sign in", '#' %></li>
      </ul>
    </nav>
  </div>
</header>
```

- `app/views/layouts/_footer.html.erb`

```erb
<!-- 보류 -->
```

- `application.html.erb`

```erb
<!DOCTYPE html>
<html>
<head>
...
</head>

<body>
  <%= render 'layouts/header' %>
  <div class="container">
    <%= yield %>
  </div>
   <%= render 'layouts/footer' %>
</body>
</html>
```

#### 1.4 customizing

- `_header.html.erb`

```erb
<header class="navbar navbar-dark bg-dark navbar-expand-lg fixed-top">
  <div class="container">
    <%= link_to "sample app", root_path, id: 'logo', class: 'navbar-brand' %>
    <!--Toggle Button-->
    <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarNav" aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>
    <!--nav bar-->
    <nav class="collapse navbar-collapse" id="navbarNav">
      <ul class="navbar-nav ml-auto">
        <li class="nav-item"><%= link_to "Home",   root_path, class: 'nav-link' %></li>
        <li class="nav-item"><%= link_to "Help",   help_path, class: 'nav-link'%></li>
        <li class="nav-item"><%= link_to "Log in", '#', class: 'nav-link' %></li>
      </ul>
    </nav>
  </div>
</header>

<header class="navbar navbar-dark bg-dark navbar-expand-lg fixed-top">
  <div class="container">
    <%= link_to "Layout App", root_path, id: 'logo', class: 'navbar-brand' %>
    <!--Toggle Button-->
    <button class="navbar-toggler"
            type="button"
            data-toggle="collapse"
            data-target="#navbarNav"
            aria-controls="navbarNav"
            aria-expanded="false"
            aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>

    <!--nav bar-->
    <nav class="collapse navbar-collapse" id="navbarNav">
      <ul class="navbar-nav ml-auto">
        <li class="nav-item"><%= link_to 'Home', root_path, class: 'nav-link' %></li>
        <li class="nav-item"><%= link_to 'Help', help_path, class: 'nav-link' %></li>
        <li class="nav-item"><%= link_to 'Log in', '#', class: 'nav-link' %></li>
      </ul>
    </nav>
  </div>
</header>
```

- `_footer.html.erb`

```erb
<footer class="container">
  <hr class="my-1">
  <small>
      <a href="https://github.com/wnsgh6315">Harry's github</a> &
      <a href="https://github.com/wnsgh6315">This project's repo</a>
    </small>
    <nav>
      <ul>
        <li><%= link_to 'About',   about_path %></li>
        <li><%= link_to 'Contact', contact_path %></li>
      </ul>
    </nav>
</footer>
```

- `routes.rb`

```ruby
Rails.application.routes.draw do
  root 'static_pages#home'

  get '/help', to: 'static_pages#help'

  get '/about', to: 'static_pages#about'
  
  get '/contact', to: 'static_pages#contact'
end
```

- `custom.scss`

```scss
@import "bootstrap";

/* universal */
body {
  padding-top: 80px;
}

section {
  //overflow: auto;
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
body {
  padding-top: 80px;
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

/* 타이포그래피 */
h1, h2, h3, h4, h5, h6 {
  line-height: 1;
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
      colr: #222;
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
      margin-left: 15px;
    }
  }
}
```

- `static_pages_controller_test.rb`

```ruby
require 'test_helper'

class StaticPagesControllerTest < ActionDispatch::IntegrationTest
  def setup
    @base_title = "Harry's blog"
  end
  
  test "should get root" do
    get root_url
    assert_response :success
    assert_select "title", "Home | #{@base_title}"
  end

  test "should get help" do
    get help_url
    assert_response :success
    assert_select "title", "Help | #{@base_title}"
  end
  
  test "should get about" do
    get about_url
    assert_response :success
    assert_select "title", "About | #{@base_title}"
  end
  
  test "should get contact" do
    get contact_url
    assert_response :success
    assert_select "title", "Contact | #{@base_title}"
  end
end
```

#### 1.5 helper 맛보기

- `app/helpers/application_helper.rb`

```ruby
module ApplicationHelper
  def full_title(page_title='')
    base_title = "Harry's blog"
    if page_title.empty?
      base_title
    else
      "#{page_title} | #{base_title}"
    end
  end
end
```

- `application.html.erb`

```erb
<!DOCTYPE html>
<html>
<head>
  <title><%= full_title(yield(:title)) %></title>
...
</head>
</html>
```

- `home.html.erb`

```erb
<!-- 삭제 -->
<% provide(:title, "Home") %>
```

---

### 2. User modeling

- `git checkout -b modeling-users`
- `rails g model User name:string email:string`
- `gemfile`

```ruby
group :development do
	...
  gem 'rails_db'
end
```

- `bundle`
- `rails db:migrate`

> `rails db:rollback ` : 마지막으로 실행한 migrate 를 취소
>
> `rails db:drop` : 모든 모델 데이터를 삭제 (*주의*) 

#### 2.1 validation 유효성검사

>  `rails t:models` : 모델만 테스트

- `user.rb`

```ruby
class User < ApplicationRecord
  validates :name, presence: true, length: { maximum: 50, minimum: 2 }
  validates :email, presence: true, length: { maximum: 255 }
end
```

- `test/models/user_test.rb`

```ruby
require 'test_helper'

class UserTest < ActiveSupport::TestCase
  def setup
    @user = User.new name: "Example User", email: "user@example.com"
  end
  
  test 'should be valid' do
    assert @user.valid? 
  end
  
  test 'name should be present' do
    @user.name = '     '
    assert_not @user.valid?
  end
  
  test 'email should be present' do
    @user.email = '   '
    assert_not @user.valid?
  end
  
  test 'name should not be too long' do
    @user.name = 'a' * 51
    assert_not @user.valid?
  end
  
  test 'email should not be too long' do
    @user.email = 'a' * 244 + '@example.com'
    assert_not @user.valid?
  end
end
```

#### 2.2 정규표현식

```
예시) /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i

# /  : 시작
# \A : 시작하는 글자 검사 시작
# [\w + \- .] : [] 안에 하나는 만족해야 함, \w => word char, +-. => 최소한 +-. 이 있어야함
# + : 최소 한 글자 이상은 있어야 함
# @ : @사인만 된다.
# [a-z \d \- .]+ : a-z 알파벳, \d => ditig(숫자), \-. => hypen, dot
# \. : . 만 된다
# [a-z]+ : 알파벳만 1글자 이상
# \z : 끝나는 단어 검사
# / : 끝
# i : 대소문자 괜찮다(case-insensitive)
```

- `user.rb`

```ruby
class User < ApplicationRecord
  validates :name, presence: true, length: { maximum: 50, minimum: 2 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
  validates :email, presence: true, 
                    length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX }
end
```

- `user_test.rb`

````ruby
class UserTest < ActiveSupport::TestCase

    							...
  
  test 'email validation should accept valid addresses' do
    valid_addresses = %w[user@example.com USER@foo.com A_US-ER@foo.bar.org first.last@foo.kr alice+bob@baz.cn]
    
    valid_addresses.each do |valid_address|
      @user.email = valid_address
      assert(@user.valid?, "#{valid_address.inspect} should be valid")
    end
  end
  
  test 'email validation should reject invalid addresses' do
    invalid_addresses = %w[user@example,com user_foo_asd.org user@user.com. djasid@bar+qwqweqe.com]
    
    invalid_addresses.each do |invalid_address|
      @user.email = invalid_address
      assert_not(@user.valid?, "#{invalid_address} should be invalid")
    end
  end
end
````

#### 2.3 중복 검사

- `user_test.rb`

```ruby
class UserTest < ActiveSupport::TestCase

    							...
        
  test 'email addresses should be unique' do
    copy_user = @user.dup
    @user.save
    assert_not copy_user.valid?
  end
end
```

- `user.rb`

```ruby
class User < ApplicationRecord
  validates :name, presence: true, length: { maximum: 50, minimum: 2 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i
  validates :email, presence: true, 
                    length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: true
end
```

#### 2.4 대소문자

- `user_test.rb`

```ruby
class UserTest < ActiveSupport::TestCase

    							...
        
  test 'email addresses should be unique' do
    copy_user = @user.dup
    copy_user.email = @user.email.upcase # 대문자
    @user.save
    assert_not copy_user.valid?
  end
end
```

- `user.rb`

```ruby
class User < ApplicationRecord
  validates :name, presence: true, length: { maximum: 50, minimum: 2 }
  VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i
  validates :email, presence: true, 
                    length: { maximum: 255 },
                    format: { with: VALID_EMAIL_REGEX },
                    uniqueness: { case_sensitive: false } 
end
```

#### 2.5 데이터베이스에서도 유효성 검증

> 만약 수 십만 명의 사용자가 가입할 경우 데이터베이스에 완전히 같은 시간에 같은 내용이 중복으로 들어갈 수 있다. 이런 경우를 대비해서 데이터베이스 내부적으로도 검증할 수 있는 것이 필요하다.  
>
> 매우 짧은 시간에 발생하는 특수한 경우이기 때문에 우리가 지금까지 해온 테스트의 범위를 넘어서서 직접 확인하기는 어렵다.

- `rails db:rollback`
- `20180625023514_create_users.rb`

```ruby
class CreateUsers < ActiveRecord::Migration[5.1]
  def change
    create_table :users do |t|
      t.string :name
      t.string :email, index: { unique: true }

      t.timestamps
    end
  end
end
```

- `rails db:migrate`
- `test/fixtures/users.yml` 파일 내용 삭제
- `user.rb`

```ruby
class User < ApplicationRecord
  before_save { self.email = self.email.downcase }
                                ...
end
```

