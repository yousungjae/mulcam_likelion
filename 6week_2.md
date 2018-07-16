# 6week_2

## 07.10(화)

### 1. Model

#### 1.1 모델관계 사전 작업

> Model 5개 : User, Profile, Product, Article, Category

| Model 1  | Model 2 | Relation |
| :------: | :-----: | :------: |
|   User   | Profile |  1 : 1   |
|   User   | Product |  1 : N   |
|   User   | Article |  1 : N   |
| Category | Product |  1 : N   |

```shell
$ rails new relation_app
```

- `gemfile`

  - [gem 'age_calculator'](https://github.com/kenn/age_calculator)

  ```ruby
  gem 'pry-rails'
  gem 'devise'
  gem 'rails_db'
  gem 'age_calculator'
  ```

```shell
$ rails g devise:install
$ rails g devise User

$ rails g model Profile name mobile address birthday:date user:belongs_to
$ rails g model Category name


$ rails g scaffold Article title content:text user:belongs_to
$ rails g scaffold Product name price:integer category:belongs_to user:belongs_to


$ rails db:migrate
```

- `user.rb`

  ```ruby
  class User < ApplicationRecord
    has_many :articles
    has_many :products
    has_one :profile, dependent: :destroy
  	...
  end
  ```

- `profile.rb`

  ```ruby
  class Profile < ApplicationRecord
    belongs_to :user
    VALID_PHONE_REGEX = /\A(010)([1-9]{1}[0-9]{2,3})([0-9]{4})\z/
    validates :mobile, format: { with: VALID_PHONE_REGEX }
  end
  ```

- `product.rb`

  ```ruby
  class Product < ApplicationRecord
    belongs_to :category
    belongs_to :user
  end
  ```

- `category.rb`

  ```ruby
  class Category < ApplicationRecord
    has_many :products
  end
  ```

#### 1.2 profile 컨트롤러 및 라우트 설정

```shell
$ rails g controller profiles new create show edit update
```

- `routes.rb`

  ```ruby
  Rails.application.routes.draw do
    resources :profiles, only: [:create, :update]
    get '/my_profile' => 'profiles#show'
    get '/new_profile' => 'profiles#new'
    post '/new_profile' => 'profiles#create'
    get '/edit_profile' => 'profiles#edit'
    patch '/edit_profile' => 'profiles#update'
    
    resources :products
    resources :articles
    devise_for :users
  end
  ```

- `profiles_controller.rb`

  ```ruby
  class ProfilesController < ApplicationController
    before_action :authenticate_user!
    before_action :set_profile, only: [:show, :edit, :update]
    
    def new
      redirect_to edit_profile_url unless current_user.profile.nil?
      @profile = Profile.new
    end
  
    def create
      @profile = Profile.new profile_params
      @profile.user = current_user
      if @profile.save
        redirect_to my_profile_url
      else
        render 'new'
      end
    end
  
    def show
      redirect_to new_profile_url if @profile.nil?
    end
  
    def edit
    end
  
    def update
    end
    
    private
      def profile_params
        params.require(:profile).permit(:name, :mobile, :address, :birthday)
      end
      def set_profile
        @profile = current_user.profile
      end
  end
  ```

#### 1.3 profile 작성/수정 설정하기

- profiles/`_form.html.erb`

  ```erb
  <h1>New Profile</h1>
  <%= form_for profile, url: new_profile_path do |f| %>
    <div>
      <%= f.label :name %>
      <%= f.text_field :name %>
    </div>
  
    <div>
      <%= f.label :address %>
      <%= f.text_field :address %>
    </div>
  
    <div>
      <%= f.label :mobile %>
      <%= f.telephone_field :mobile %>
    </div>
    
    <div>
      <%= f.label :birthday %>
      <%= f.date_field :birthday %>
    </div>
  
    <div>
      <%= f.submit %>
    </div>
  <% end %>
  ```

- `show.html.erb`

  ```erb
  <h1><%= @profile.name %>님의 프로필입니다.</h1>
  <%= link_to '프로필 수정', edit_profile_path %>
  <p><%= current_user.email %></p>
  <p><%= @profile.address %></p>
  <p><%= @profile.mobile %></p>
  <p><%= @profile.birthday %></p>
  ```

- helpers/`profiles_helper.rb`

  - **하나의 form 으로 서로 다른 form url 을 가진 new 와 edit 에서 사용하기 위해 helper 작성**

  ```ruby
  module ProfilesHelper
    def form_auto_route
      if %w[new create].include?(params[:action])
        new_profile_path
      elsif %w[edit update].include?(params[:action])
        edit_profile_path
      end
    end
  end
  ```

- profiles/`_form.html.erb`

  ```erb
  <%= form_for profile, url: form_auto_route do |f| %>
  	...
  ```

- `new.html.erb`

  ```erb
  <h1>New Profile</h1>
  
  <%= render 'form', profile: @profile %>
  ```

- `edit.html.erb`

  ```erb
  <h1>Edit Profile</h1>
  
  <%= render 'form', profile: @profile %>
  ```

- `profiles_controller.rb`

  ```ruby
  class ProfilesController < ApplicationController  
  ...
  
    def create
      @profile = Profile.new profile_params
      @profile.user = current_user
      if @profile.save
        redirect_to my_profile_url
      else
        render 'new'
      end
    end
  
  ...
  
    def update
      if @profile.update profile_params
        redirect_to my_profile_url
      else
        render 'edit'
      end
    end
    
  ...
  end
  ```

### 2. 좋아요 구현 (M : N)

| Model 1 | Model 2 | Relation |
| :-----: | :-----: | :------: |
|  User   |  Like   |  1 : N   |
| Article |  Like   |  1 : N   |

#### 2.1 모델관계 사전 작업

```shell
$ rails g model Like user:belongs_to article:belongs_to
```

- `20180710071552_create_likes.rb` : **timestamp 를 사용하지 않도록 설정**

  ```ruby
  class CreateLikes < ActiveRecord::Migration[5.1]
    def change
      create_table :likes do |t|
        t.belongs_to :user, foreign_key: true
        t.belongs_to :article, foreign_key: true
      end
    end
  end
  ```

```shell
$ rails db:migrate
```

- `user.rb` / `article.rb` : 해당 코드 추가

  ```ruby
  has_many :likes
  ```

#### 2.2 article 컨트롤러 및 뷰 설정

- `articles_controller.rb`

  ```ruby
    def create
      @article = Article.new(article_params)
      # 생성할때만 article 의 user 입력
      @article.user = current_user
        ...
          
        private
        ...
         def article_params
        	 params.require(:article).permit(:title, :content) # user_id 삭제
         end
    end
  ```

- articles/`_form.html.erb` : 해당 코드 삭제

  ```erb
    <div class="field">
      <%= form.label :user_id %>
      <%= form.text_field :user_id, id: :article_user_id %>
    </div>
  ```

- articles/`show.html.erb`

  ```erb
  ...
  <p>
    <strong>User:</strong>
    <%= @article.user.email %>
  </p>
  ...
  ```

#### 2.3 좋아요/좋아요 취소 구현

```shell
$ rails g controller Likes
```

```shell
$ rails c
$ u = User.first
$ a = Article.first
$ Like.create user: u, article: a
$ u.likes
$ a.likes
```

- articles/`show.html.erb`

  ```erb
  	...
  <p>
    <strong>Title:</strong>
    <%= @article.title %>
  </p>
  
  <p>
    <strong>좋아요:</strong>
    <%= @article.likes.count %>
  </p>
  
  ...
  
  <form action="/articles/<%= @article.id %>/like" method="POST">
    <button>좋아요</button>
  </form>
  
  <%= link_to 'Edit', edit_article_path(@article) %> |
  <%= link_to 'Back', articles_path %>
  ```

- `likes_controller.rb`

  ```ruby
  class LikesController < ApplicationController
    def like_toggle
      like = Like.find_by(user: current_user, article_id: params[:id])
      if like.nil?
        Like.create(user: current_user, article_id: params[:id])   
      else
        like.destroy
      end
        redirect_to article_url(params[:id])
    end
  end
  ```

- `routes.rb`

  ```ruby
  post 'articles/:id/like' => 'likes#like_toggle', as: 'like'
  ```

> 이제 show 페이지에서 좋아요를 누르면 1이 되고 다시 누르면 0이 되는 걸 확인할 수 있다.

- articles/`show.html.erb`

  ```erb
  	...
  <%= link_to @button, like_path(@article), method: :POST %>
  
  <%= link_to 'Edit', edit_article_path(@article) %> |
  <%= link_to 'Back', articles_path %>
  ```

- `articles_controller.rb`

  ```ruby
    def show
      if Like.find_by(user: current_user, article: @article).nil?
        @button = '좋아요'
      else
        @button = '좋아요 취소'
      end
    end
  	...
  ```

> 좋아요와 좋아요 취소 링크가 바뀌는지 확인