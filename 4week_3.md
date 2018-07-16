# 4week_3

## 06.27(수)

### 1. Gravatar

#### 1.1 create, show controller 작성

- `user_controller.rb`

  ```ruby
  class UsersController < ApplicationController
    def new
      @user = User.new
    end
    
    def create
      @user = User.new user_params
      @user.save
      redirect_to signup_url # 아직 미완성
    end
    
    def show
      @user = User.find params[:id]
    end
    
    def index
    end
    
    
    private
    def user_params
      params.require(:user).permit(:name, :email, :password, :password_confirmation)
    end
  end
  ```

#### 1.2 show view 작성

- `show.html.erb`

  ```erb
  <% provide(:title, @user.name) %>
  <%= @user.name %>
  ```

#### 1.3 Gravatar 사용해보기 (with helper 를 통한 코드의 재사용성)

[GravatarGuide](https://ko.gravatar.com/site/implement/images/ruby/)

- `app/helpers/user_helper.rb`

  ```ruby
  module UsersHelper
    # 특정 User 에 대해 Gravatar 사진을 return 한다
    def gravatar_for(user)
      gravatar_id = Digest::MD5.hexdigest(user.email.downcase)
      gravatar_url = "https://secure.gravatar.com/avatar/#{gravatar_id}"
      image_tag(gravatar_url, alt: user.name, class: "gravatar")
    end
  end
  ```

- **Gravatar 가입 후 프로필 사진 넣기**

- Gravatar 에 *가입한 메일과 동일한 메일* 로 현재 c9 프로젝트 페이지에 가입 후 프로필 사진 확인해보기

- `custom.scss` : 해당 코드 추가

  ```scss
  .gravatar {
    float: left;
    margin-right: 10px;
  }
  
  .gravatar_edit {
    margin-top: 15px;
  }
  ```

- `user_controller.rb`

  ```ruby
  								...
  def create
    @user = User.new user_params
    @user.save
    redirect_to user_url(@user)
  end
  								...
  ```

- `show.html.erb`

  ```erb
  <% provide(:title, @user.name) %>
  <div class="row">
    <aside class="col-md-4">
      <section class="user_info">
        <h1>
          <%= gravatar_for @user %>
          <%= @user.name %>
        </h1>
      </section>
    </aside>
  </div>
  ```

### 2. Error messages

#### 2.1 render

- 빈값으로 가입버튼을 누르면 에러페이지가 뜨는데 직관성이 부족하므로 
  if 문 으로 조건 걸기 (render 개념)

- `user_controller.rb`

  ```ruby
  def create
      @user = User.new user_params
      if @user.save
          redirect_to user_url(@user)
      else
          render 'new' 
      end
  end
  ```

- `render 'new'`

  - missing template 에러 방지 ( 변화한 url 확인해보기 )
  - view 가 없는 create 액션을 유지하면서 다른 화면을 보여주기 위해 사용
  - 'new' 인 이유, new.html.erb 를 지칭

#### 2.2 Error message module

- 에러메세지 모듈화
- 여러 모듈을 관리하기 위해 새로운 폴더 생성

- `view/shared/_error_messages.html.erb`

  ```erb
  <% if @user.errors.any? %>
    <div id="error_explanation">
      <div class="alert alert-danger">
        제출하신 양식에는 <%= @user.errors.count %>개의 에러가 있습니다.
      </div>
      <ul>
        <% @user.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
  ```

- `new.html.erb`

  ```erb
  <% provide(:title, 'Sign Up') %>
  <h1>Sign Up</h1>
  <div class="row">
    <div class="col-md-6 offset-md-3">
      <%= form_for @user do |f| %>
      
        <%= render 'shared/error_messages' %>
  					
        							...
    </div>
  </div>
  ```

>  password 유효성 검사가 2번 수행되고 때문에 `user.rb` 패스워드 유효성 검사 수정

- `user.rb`

  ```ruby
  class User < ApplicationRecord
  							...
    
    has_secure_password 
    
    validates :password, length: { minimum: 6 }
  end
  ```

### 3. 통합 테스트

#### 3.1 통합 테스트 실습

```shell 
$ rails g integration_test users_signup 
```

- `test/integration/users_signup_test.rb`

  ```ruby
  require 'test_helper'
  
  class UsersSignupTest < ActionDispatch::IntegrationTest
    
    test 'invalid signup info' do
      # 1: 회원가입 페이지 접속
      get signup_path
      assert_no_difference 'User.count' do
        # 2: 회원가입 form 을 작성하여 제출한다.
        post users_path, params: { 
                                  user: 
                                    { 
                                      name: "",
                                      email: "user@invalid",
                                      password: "foo",
                                      password_confirmation: "asd"
                                    }
                                  }
        end
        # 3: 새로 작성하는 페이지로 돌아옴
        assert_template 'users/new'    
        # 4: Error 관련 메세지가 나오는지 확인.
        assert_select 'div#error_explanation'
      end
  end
  ```

#### 3.2 http error

- 회원가입 시 아무것도 기입하지 않고 진행하면 flash 에러 메시지가 뜬다. 여기서 새로 고침을 하면 원래 에러가 나타나는데 이것을 해결하기 위해 아래와 같이 시행한다.

- ***route 와 http 의 개념과 관련된 사항으로 추후에 다시 수업을 진행할 예정.***

- `new.html.erb`

  ```erb
  <% provide(:title, 'Sign Up') %>
  <h1>Sign Up</h1>
  <div class="row">
    <div class="col-md-6 offset-md-3">
      <%= form_for(@user, url: signup_path) do |f| %>
  							...
  </div>
  ```

- `routes.rb`

  ```ruby
  Rails.application.routes.draw do
      			...
    post '/signup', to: 'users#create'
  end
  ```

#### 3.3 Flash message customizing

- `users_controller.rb`

  ```ruby
  class UsersController < ApplicationController
  
      				...
          
    def create
      @user = User.new(user_params)
      if @user.save
        flash[:success] = "Welcome to my blog!"
        redirect_to user_url(@user)
      else
        render 'new'
      end
    end
  
      		 		...
  end
  ```

- `application.html.erb` : flash 메세지 삽입 및 커스터마이징

  ```erb
  <!DOCTYPE html>
  <html>
  
  			...
        
    <body>
      <!--Header-->
      <%= render 'layouts/header' %>
      <div class="container">
        <!--Flash Message를 보여주는 영역--> 
        <% flash.each do |message_type, message| %>
          <div class="alert alert-<%= message_type %>">
            <%= message %>
          </div>
        <% end %>
        <!--View File 의 내용이 들어오는 영역-->
        <%= yield %>
      </div>
      <!--Footer-->
      <%= render 'layouts/footer' %>
      <!--개발환경에서만 보이는 params내용-->
      <%= debug(params) if Rails.env.development? %>
    </body>
  </html>
  ```

---

### 4. 코드 점검

#### 4.1 **테스트 오류 수정 및 CSS 파일 통일을 위한 full code**

- `user.rb`

  ```ruby
  class User < ApplicationRecord
    before_save { self.email = self.email.downcase }
    
    validates :name, presence: true, length: { maximum: 50, minimum: 2 }
    
    VALID_EMAIL_REGEX = /\A[\w+\-.]+@[a-z\d\-]+(\.[a-z\d\-]+)*\.[a-z]+\z/i
    validates :email, presence: true, 
                      length: { maximum: 255 },
                      format: { with: VALID_EMAIL_REGEX },
                      uniqueness: { case_sensitive: false } # 대소문자 구분없이 체크 하겠다.
   
    has_secure_password 
    
    validates :password,
              length: { minimum: 6 },
              format: { with:  /\A[\S]*\z/ }
  end
  ```

- `custom.scss`

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
  
  #error_explanation {
    color: red;
    ul {
      color: red;
      margin: 0 0 30px 0;
    }
  }
  
  
  /* sidebar */
  
  aside {
    section.user_info {
      margin-top: 20px;
    }
    section {
      padding: 10px 0;
      margin-top: 20px;
      &:first-child {
        border: 0;
        padding-top: 0;
      }
      span {
        display: block;
        margin-bottom: 3px;
        line-height: 1;
      }
      h1 {
        font-size: 1.4em;
        text-align: left;
        letter-spacing: -1px;
        margin-bottom: 3px;
        margin-top: 0px;
      }
    }
  }
  
  .gravatar {
    float: left;
    margin-right: 10px;
  }
  
  .gravatar_edit {
    margin-top: 15px;
  }
  ```


---

---

### 5. 혼자서 해보기

**review app**

1. rails project 생성하기 (review_app)
2. Model 생성 (Post)

- string => title
- text => content

3. Routing

   ```ruby
   root 'post#index'
   resources :posts
   ```

4. Controller 생성

5. View 생성 (new index show edit)

6. Form 을 통한 CRUD 구현

