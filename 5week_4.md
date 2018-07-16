# 5week_4

## 07.05(목)

### 0. 쿠키와 세션 - 개념

1) http protocol 단점

- **stateless protocol** (요청을 받으면 서버에서 응답을 보내고 거기서 끝 == **상태**가 없다)
- 즉, 일정한 정보를 지속적으로 들고 있을 수 없다.
- ex) 로그인/장바구니 는 http 로 만들면 문제가 생긴다.

2) 해결책

1. 세션

- 일정 시간(브라우저 종료시까지) 반 영구적으로 상태를 유지(**서버**)
  - 로그인/장바구니

2. 쿠키

- 사용자의 브라우저에 저장되는 텍스트 정보(**클라이언트**)
- 쿠키를 통해 세션을 구현

---

### 1. 로그인 페이지 구현 

```shell
$ rails g controller Sessions new
```

- `routes.rb`

  ```ruby
  get '/login', to:   'sessions#new'
  post '/login', to: 'sessions#create'
  delete '/logout', to: 'sessions#destroy'
  ```

- sessions/`new.html.erb`

  ```erb
  <h1>Log in</h1>
  <%= form_for(:session, url: login_path) do |f| %>
    <div>
      <%= f.label :email %>
      <%= f.email_field :email %>
    </div>
    <div>
      <%= f.label :password %>
      <%= f.password_field :password %>
    </div>
    <div>
      <%= f.submit "Log in", class: 'btn btn-primary' %>
    </div>
  <% end %>
  <p>
    회원이 아니세요?
    <%= link_to '회원 가입', signup_path %>
  </p>
  ```

- `sessions_controller.rb`

  ```ruby
  class SessionsController < ApplicationController
    def new
    end
    def create
      render 'new'
    end
    def destroy
    end
  end
  ```


---

### 2. 로그인 내부동작 구현

- `sessions_controller.rb`

  ```ruby
  class SessionsController < ApplicationController
    def new
    end
    
    def create
      user = User.find_by(email: params[:session][:email])
      if user && user.authenticate(params[:session][:password])
        session[:user_id] = user.id
        redirect_to user_url(user)
        # 로그인한 유저를 어딘가로 redirect 시킨다
      else
        flash[:danger] = 'email / password 가 일치하지 않습니다'
        render 'new'  
      end
     
    end
    
    def destroy
    end
  end
  
  ```

- `application.html.erb` : 로그인하면 로그아웃 버튼, 로그아웃하면 로그인 버튼 출력 하도록

  ```erb
  <body>
    <div class="container">
      <header>
        <% if User.find_by(id: session[:user_id]) %>
          <%= link_to 'Logout', logout_path, method: :DELETE %>
        <% else %>
          <%= link_to 'Login', login_path %>
        <% end %>
      </header>
  	...
  ```

### 3. 헬퍼 만들기

- `appplication_controller.rb` : 모든 컨트롤러에서 sessionshelper 를 사용하기 위해 적용

  ```ruby
  class ApplicationController < ActionController::Base
    protect_from_forgery with: :exception
    include SessionsHelper
  end
  ```

##### 1) 로그인

- `sessions_helper.rb`

  ```ruby
  module SessionsHelper
    def log_in(user)
      session[:user_id] = user.id
    end
  end
  ```

- `sessions_controller.rb`

  ```ruby
  class SessionsController < ApplicationController
    def new
    end
    
    def create
      user = User.find_by(email: params[:session][:email])
      if user && user.authenticate(params[:session][:password])
        log_in(user)
        redirect_to user_url(user)
  	...
  ```

##### 2) 현재 유저의 접속여부 확인

- `sessions_helper.rb`

  ```ruby
  module SessionsHelper
  	...
    def current_user
      @current_user = @current_user || User.find_by(id: session[:user_id])
      # @current_user ||= User.find_by(id: session[:user_id])
    end
    
    def logged_in?
      !current_user.nil?
    end
  end
  ```

- `application.html.erb` 

  ```erb
  <body>
    <div class="container">
      <header>
        <% if logged_in? %>
          <%= link_to 'Logout', logout_path, method: :DELETE %>
        <% else %>
          <%= link_to 'Login', login_path %>
        <% end %>
      </header>
        ...
  ```

-  `users_controller.rb` : 회원가입에 성공하면 바로 로그인 되도록

  ```ruby
    def create
      @user = User.new user_params
      if @user.save
        log_in @user
        flash[:success] = "회원가입을 축하합니다."
        redirect_to user_path(@user)
      else
        render 'new'
      end
    end
  ```

##### 3) 로그아웃

- `sessions_helper.rb`

  ```ruby
  module SessionsHelper
  	...
    def log_out
      session.delete(:user_id)
      @current_user = nil
    end
  	...
  ```

- `sessions_controller.rb`

  ```ruby
    ...
    def destroy
      log_out
      redirect_to login_url
    end
  end
  ```

  



