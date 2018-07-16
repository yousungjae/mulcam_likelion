# 3week_1

## 06.18(월)

### 모델 복습

-  `rails g model Menu`

- 20180618011228_create_menus.rb

  ```ruby
  class CreateMenus < ActiveRecord::Migration[5.1]
    def change
      create_table :menus do |t|
        
        t.string :name
        t.integer :price
  
        t.timestamps
      end
    end
  end
  ```

- `rails db:migrate`

- 레일즈 콘솔창에서 CRUD 실습 진행

---

```ruby
Menu.create({ :name => "tomato", :price => 700 })
Menu.create({ name: "tomato", price: 700 })
Menu.create name: "tomato", price: 700
```

```ruby
menu = Menu.new name: 'kimbab', price: 2500
menu.save
```

### 실습

- `rails g model Person`

- `rails d model Person`

- `rails g model Persion name:string age:integer gender:boolean contry`

  ```ruby
  class CreatePeople < ActiveRecord::Migration[5.1]
    def change
      create_table :people do |t|
        
        t.string :name
        t.integer :age
        t.boolean :gender
        t.string :country
  
        t.timestamps
      end
    end
  end
  ```

- `rails db:migrate`

- `Person.create! name: 'harry', age: 28, gender: true, country: 'KOR'`

- 10개이상 데이터 추가하기

  ```ruby
  p = Person.find 1
  p.name 			# 원하는 결과 값이 안나온다.
  p = p.first		# 배열로 받았기 때문에 특정 배열을 선택해서 다시 담아줘야한다.
  p.name 			# 이제야 원하는 값을 찾을 수 있다.
  ```

  ```ruby
  older_people = Person.where 'age >= ?', 40
  middle_people = Person.where 'age >= ? AND age < ?', 30, 50 
  
  p = Person.new name: 'hansen', age: 60, gender: false, country: 'JPN'
  p.save
  p.update name: 'marvel', age: 100
  ```

  ---

### CRUD 구현

- `rails g controller Menus` (컨트롤러명은 모델의 복수형으로 쓰는게 기본적이다.)

#### 1. Retrieve 구현

- `menus_controller.rb`

  ```ruby
  class MenusController < ApplicationController
    # Create
    
    # Retrieve
    def index
      # 모든 메뉴를 보여주는 화면
      @menus = Menu.all
    end
    
    def show
      # 1개의 메뉴를 보여주는 화면
      @menu = Menu.find params[:id]
    end
    
    # Update
    
    # Destroy
    
  end
  ```

- `index.html.erb`

  ```erb
  <h1>All Menus</h1>
  <hr />
  <% @menus.each do |menu| %>
    <a href="/menus/show/<%= menu.id %>">
      <p><%= menu.name %></p>
    </a>
  <% end %>
  ```

- `show.html.erb`

  ```erb
  <h1>One Menu</h1>
  <hr />
  <h2><%= @menu.name %></h2>
  <p><%= @menu.price %> 원</p>
  <a href="/menus/index">돌아가기</a>
  ```

- `routes.rb`

  ```ruby
  Rails.application.routes.draw do
    # menus - Create
      
    # menus - Retrieve
    get '/menus/index', to: 'menus#index'
    get '/menus/show/:id', to: 'menus#show'
      
    # menus - Update
      
    # menus - Destroy
  end
  ```

#### 2. Create 구현

- `menus_controller.rb`

  ```ruby
  class MenusController < ApplicationController
    # Create
    def new
      # menu 하나를 입력하는 화면
    end
    
    def create
      # menu를 DB 에 저장하는 액션
      menu = Menu.new
      menu.name = params[:menu][:name]
      menu.price = params[:menu][:price]
      menu.save
      
      # menu = Menu.create name: params[:menu][:name], price: params[:menu][:price]
      redirect_to "/menus/show/#{menu.id}"
    end
    
    # Retrieve
    def index
      # 모든 menu를 보여주는 화면
      @menus = Menu.all
    end
    
    def show
      # 1개의 menu를 보여주는 화면
      @menu = Menu.find params[:id]
    end
    
    # Update
    
    # Destroy
    
  end
  ```

- `new.html.erb`

  ```erb
  <h1>New Menu</h1>
  <hr />
  <form action="/menus/create" method="POST">
    <input type="text" name="menu[name]"></input>
    <input type="number" name="menu[price]"></input>
    <input type="submit"></input>
  </form>
  ```

- `index.html.erb`

  ```erb
  <h1>All Menus</h1>
  <h2><a href="/menus/new">New Menu</a></h2>
  <hr />
  <% @menus.each do |menu| %>
    <a href="/menus/show/<%= menu.id %>">
      <p><%= menu.name %></p>
    </a>
  <% end %>
  ```

- `routes.rb`

  ```ruby
  Rails.application.routes.draw do
    # menus - Create
    get '/menus/new', to: 'menus#new'
    post '/menus/create', to: 'menus#create'
    
    # menus - Retrieve
    get '/menus/index', to: 'menus#index'
    get '/menus/show/:id', to: 'menus#show'
    
    # menus - Update
    
    # menus - Destroy
  end
  ```

- **레일즈 보안오류가 난다. 일단 수정해야 할 1가지가 있다.**

- `application_controller.rb`

  ```ruby
  class ApplicationController < ActionController::Base
    # protect_from_forgery with: :exception
  end
  ```

> brute force attack (무차별 대입 공격)
>
> CSRF (사이트 간 요청 위조)

- `menus_controller.rb`

  ```ruby
  def new
      # menu 하나를 입력하는 화면
      @token = form_authenticity_token
  end
  ```

- `new.html.erb`

  ```erb
  <h1>New Menu</h1>
  <hr />
  <form action="/menus/create" method="POST">
    <input type="hidden" name="authenticity_token" value=<%= @token %>></input>
    <input type="text" name="menu[name]"></input>
    <input type="number" name="menu[price]"></input>
    <input type="submit"></input>
  </form>
  ```

#### 3. Destroy 구현

- `menus_controller.rb`

  ```ruby
  class MenusController < ApplicationController
    # Create
    def new
      # menu 하나를 입력하는 화면
      @token = form_authenticity_token
    end
    
    def create
      # menu를 DB 에 저장하는 액션
      menu = Menu.new
      menu.name = params[:menu][:name]
      menu.price = params[:menu][:price]
      menu.save
      
      # menu = Menu.create name: params[:menu][:name], price: params[:menu][:price]
      redirect_to "/menus/show/#{menu.id}"
    end
    
    # Retrieve
    def index
      # 모든 menu를 보여주는 화면
      @menus = Menu.all
    end
    
    def show
      # 1개의 menu를 보여주는 화면
      @menu = Menu.find params[:id]
    end
    
    # Update
    
    # Destroy
    def destroy
      # 특정 menu 를 삭제하는 액션
      @menu = Menu.find params[:id]
      @menu.destroy
      redirect_to '/menus/index'
    end
  end
  ```

- `show.html.erb`

  - `a 태그의 data-xxx 속성은 html 문법이 아닌 jquery 기반 문법이다.`

  ```erb
  <h1>One Menu</h1>
  <hr />
  <h2><%= @menu.name %></h2>
  <p><%= @menu.price %> 원</p>
  <a href="/menus/index">돌아가기</a>
  
  <a href="/menus/destroy/<%= @menu.id %>" data-method="DELETE" data-confirm="진짜 지우실건가요?">
    <p>삭제</p>
  </a>
  ```

- `routes.rb`

  ```ruby
  Rails.application.routes.draw do
    # menus - Create == POST
    get '/menus/new', to: 'menus#new'
    post '/menus/create', to: 'menus#create'
    
    # menus - Retrieve == GET
    get '/menus/index', to: 'menus#index'
    get '/menus/show/:id', to: 'menus#show'
    
    # menus - Update == PATCH
    
    # menus - Destroy == DELETE
    delete '/menus/destroy/:id', to: 'menus#destroy'
  end
  ```

#### 4. Update 구현

- `menus_controller.rb`

  ```ruby
  class MenusController < ApplicationController
  ...
    
    # Update
    def edit
      # 기존 menu를 수정하는 화면
      @menu = Menu.find params[:id]
      @token = form_authenticity_token
    end
    
    def update
      # 수정한 menu 를 DB에 저장하는 액션
      menu = Menu.find params[:menu][:id]
      menu.name = params[:menu][:name]
      menu.price = params[:menu][:price]
      menu.save
      redirect_to "/menus/show/#{menu.id}"
    end
    
  ...
  end
  ```

- `show.html.erb`

  ```erb
  <h1>One Menu</h1>
  <hr />
  <h2><%= @menu.name %></h2>
  <p><%= @menu.price %> 원</p>
  <a href="/menus/index">돌아가기</a>
  
  <a href="/menus/edit/<%= @menu.id %>">
    <p>수정</p>
  </a>
  
  <a href="/menus/destroy/<%= @menu.id %>" data-method="DELETE" data-confirm="진짜 지우실건가요?">
    <p>삭제</p>
  </a>
  ```

- `edit.html.erb`

  ```erb
  <h1>Edit Menu</h1>
  <hr />
  <form action="/menus/update" method="POST">
    <input type="hidden" name="_method" value="PATCH"></input>
    <input type="hidden" name="authenticity_token" value=<%= @token %>></input>
    <input type="hidden" name="menu[id]" value=<%= @menu.id %>></input>
    <input type="text" name="menu[name]" value=<%= @menu.name %>></input>
    <input type="number" name="menu[price]" value=<%= @menu.price %>></input>
    <input type="submit"></input>
  </form>
  ```

- `routes.rb`

  ```ruby
  Rails.application.routes.draw do
  ...
    
    # menus - Update == PATCH
    get '/menus/edit/:id', to: 'menus#edit'
    patch '/menus/update', to: 'menus#update'
  
   ...
  end
  ```

> TODO: 주석
>
> `# TODO: 주석내용`
>
> `rails notes` 를 사용하여 주석들의 위치와 내용을 손쉽게 확인할 수 있다.