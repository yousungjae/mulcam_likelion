# 5week_5

## 07.06(금)

### 1. devise 기초

#### 1.1 getting started

> practice_app => review_app 으로 작업 위치 이동 !

[gem devise](https://github.com/plataformatec/devise)

```ruby
gem 'devise'
```

```shell
$ bundle
$ rails g devise:install
$ rails g devise User
```

- `routes.rb` : root 라우트 만들기

  ```ruby
  root 'songs#index'
  ```

- `20180706011908_devise_create_users.rb` 둘러보기

```shell
$ rails db:migrate
```

- `application_conroller.rb` : (모든 액션들이 수행하기 전에 ) 해당 사용자의 인증여부 확인

  ```ruby
  class ApplicationController < ActionController::Base
    protect_from_forgery with: :exception
    before_action :authenticate_user!
  end
  ```

- 이제 페이지 새로고침하면 어떤 페이지에서도 로그인 화면으로 돌아간다. (사용자가 로그인되어 있지 않기 때문)

- 확인 후 `before_action :authenticate_user!` 삭제

- `songs_controller.rb`

  ```ruby
  class SongsController < ApplicationController
    before_action :set_song, only: [:show, :edit, :update, :destroy]
    before_action :authenticate_user!, except: [:show, :index]
      ...
  end
  ```

- `comments_controller.rb`

  ```ruby
  class CommentsController < ApplicationController
    before_action :authenticate_user!
      ...
  end
  ```

#### 1.2 devise view

##### 1) devise view 둘러보기

```shell
$ rails g devise:views
```

- views/devise/`registrations` 와 /`sessions`  폴더 둘러보기
- bootstrap devise 를 사용하기 위해 기존 devise views 삭제

```shell
$ rails d devise:views
```

#### 1.3 bootstrap devise view 

[Devise Bootstrap Views Gem](https://github.com/hisea/devise-bootstrap-views)

```ruby
gem 'devise-bootstrap-views'
```

```shell
$ bundle
```

- `custom.scss`

  ```css
  @import "bootstrap";
  @import "devise_bootstrap_views";
  ```

```shell
$ rails g devise:views:bootstrap_templates
$ rails g devise:views:locale ko # 한글자막 파일 만들기
```

- config/locales/devise.views.ko.yml 파일 생성 확인

##### 1) 지역설정

[Rails Locale Data Repository](https://github.com/svenfuchs/rails-i18n)

```ruby
gem 'rails-i18n', '~> 5.1'
```

```shell
$ bundle
```

- config/`application.rb`

  ```ruby
  module ReviewApp
    class Application < Rails::Application
      # Initialize configuration defaults for originally generated Rails version.
      config.load_defaults 5.1
      config.i18n.default_locale = :ko
        ...
  ```

##### 2) flash 메세지 출력

- `application.html.erb`

  ```erb
  <body>
    <div class="container">
      <% flash.each do |msg_type, msg| %>
        <div class="alert alert-<%= msg_type %>" role="alert">
          <%= msg %>
        </div>
      <% end %>
      <%= yield %>
    </div>
  </body>
  ```

- `_form.html.erb`

  ```erb
  <% if song.errors.any? %>
    <ul>
      <% song.errors.full_messages.uniq.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
    </ul>
  <% end %>
  	...
  ```

- `songs_controller.rb`

  ```ruby
  class SongsController < ApplicationController
  	...
    def create
      @song = Song.new(song_params)
      if @song.save
        flash[:success] = "#{@song.title} 이 Playlist 에 저장되었습니다."
        redirect_to song_url @song.id
      else
        flash[:danger] = "다음과 같은 이유로 저장하지 못했습니다."
        render 'new'
      end
    end
      ...
    def update
      if @song.update(song_params)
        if !@song.cover?
          @song.cover = nil
          @song.save
        end
        redirect_to song_url(@song)
      else
        flash[:danger] = "다음과 같은 이유로 저장하지 못했습니다."
        render 'edit'
      end
    end
      
    end
   	...
    def destroy
      if @song.destroy
        flash[:success] = "#{@song.title} 이 삭제되었습니다."
        redirect_to songs_url
      end
    end
  	...
  ```

### 2. devise 활용

#### 2.1 db 정리 및 모델관계 설정

```shell
$ rails d migration add_cover_to_songs
```

- `20180629013729_create_songs.rb`

  ```ruby
  class CreateSongs < ActiveRecord::Migration[5.1]
    def change
      create_table :songs do |t|
        t.string :title
        t.string :artist
        t.text :lyric
        t.string :cover
        t.belongs_to :user
  
        t.timestamps
      end
    end
  end
  ```

- `20180702013939_create_comments.rb`

  ```ruby
  class CreateComments < ActiveRecord::Migration[5.1]
    def change
      create_table :comments do |t|
        t.string :content
        t.belongs_to :song
        t.belongs_to :user
  
        t.timestamps
      end
    end
  end
  ```

```shell
$ rails db:drop
$ rails db:migrate
```

- `user.rb`

  ```ruby
  class User < ApplicationRecord
  	...
    has_many :songs, dependent: :destroy
    has_many :comments, dependent: :destroy
  end
  ```

- `comment.rb`

  ```ruby
  class Comment < ApplicationRecord
    belongs_to :song
    belongs_to :user
  ...
  end
  ```

- `song.rb`

  ```ruby
  class Song < ApplicationRecord
    has_many :comments, dependent: :destroy
    belongs_to :user
  	...
  ```

- `songs_controller.rb`

  ```ruby
    ...
    def song_params
      params.require(:song).permit(:title, :artist, :lyric, :cover, :user_id)
    end
  end
  ```

- `comments_controller.rb`

  ```ruby
    ...
    def comment_params
      params.require(:comment).permit(:content, :song_id, :user_id)
    end
  end
  ```

#### 2.2 노래와 댓글 작성시에 user_id 추가

- `show.html.erb`

  ```erb
    <%= form_for(@comment) do |f| %>
      <%= f.label :content, '댓글: ' %>
      <%= f.text_field :content %>
      <%= f.hidden_field :song_id, value: @song.id %>
      <%= f.hidden_field :user_id, value: current_user.id %>
      <%= f.submit %>
    <% end %>
  ```

- `_form.html.erb`

  ```erb
  	...
  <%= form_for(song) do |f| %>
    <%= f.hidden_field :user_id, value: current_user.id %>
    <div>
  	...
  ```

#### 2.3 접속자 상태에 따른 권한 정리하기

- `show.html.erb`

  ```erb
  	...
  <% if @song.user == current_user %>				<!-- 작성자만 노래를 수정 삭제 하도록 -->
    <p>
      <%= link_to('수정', edit_song_path(@song)) %>
      |
      <%= link_to '삭제', song_path(@song), 
                          method: :delete,
                          data: {
                            confirm: 'Remove song'
                          } %>
    </p>
  <% end %>
  
  <!--댓글 New-->
  <% if user_signed_in? %>					<!-- 로그인된 유저만 댓글을 작성 하도록 -->
    <%= form_for(@comment) do |f| %>
      <%= f.label :content, '댓글: ' %>
      <%= f.text_field :content %>
      <%= f.hidden_field :song_id, value: @song.id %>
      <%= f.hidden_field :user_id, value: current_user.id %>
      <%= f.submit %>
    <% end %>
  <% else %>
    <%= link_to "댓글을 작성하려면, 먼저 로그인 하세요.", new_user_session_path %>
  <% end %>
  
  <!--댓글 Show-->
  <ul id="comments">
    <% @comments.each do |comment| %>
      <li>
        <%= comment.user.email %> : 
        <%= comment.content %> |
        <%= link_to('삭제', comment_path(comment), method: :delete) if comment.user == current_user %>				<!-- 댓글 작성자만 삭제할 수 있도록-->
      </li>
    <% end %>
  </ul>
  <%= link_to 'back', songs_path %>
  ```

  - `index.html.erb`

  ```erb
  <h1>Playlist</h1>
  <!-- 로그인된 유저만 노래를 등록할 수 있도록 -->
  <h2><%= link_to('New song', new_song_path) if user_signed_in? %></h2>
  <ul>
    <% @songs.each do |song| %>
      <li>
        <%= image_tag song.cover.thumb.url if song.cover? %>
        <%= link_to "#{song.title} - #{song.artist}", song_path(song) %>
      </li>
    <% end %>
  </ul>
  <%= paginate @songs %>
  ```

- `songs_controller.rb` : 사진 수정 시 새로운 사진을 넣지 않으면 자동 삭제

  ```ruby
    def update
      @song.remove_cover! if !params[:song][:cover]
      if @song.update(song_params)
        if !@song.cover?
            	...
  ```

#### 2.4 header 및 css 다듬기

- layouts/`_header.html.erb`

  ```erb
  <header class="navbar navbar-dark bg-dark navbar-expand-lg fixed-top">
    <div class="container">
      <%= link_to "Jukebox", root_path, id: 'logo', class: 'navbar-brand' %>
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
          <li class="nav-item"><%= link_to 'Playlist', root_path, class: 'nav-link' %></li>
          <li class="nav-item"><%= link_to 'New song', new_song_path, class: 'nav-link' %></li>
          <li class="nav-item">
            <% if !user_signed_in? %>
              <%= link_to 'Log in', new_user_session_path, class: 'nav-link' %></li>
            <% else %>
              <%= link_to 'Log out', destroy_user_session_path, class: 'nav-link', method: :DELETE %></li>
            <% end %>
        </ul>
      </nav>
    </div>
  </header>
  ```

- `application.html.erb`

  ```erb
  <body>
    <%= render 'layouts/header' %>
    <div class="container">
      <% flash.each do |msg_type, msg| %>
        <% msg_type = 'info' if msg_type == 'notice' %>
  	...
  ```

- `custom.scss`

  ```scss
  @import "bootstrap";
  @import "devise_bootstrap_views";
  
  body {
    margin-top: 70px;
  }
  ```

- `index.html.erb`

  ```erb
  <div class="row">
    <% @songs.each do |song| %>
      <div class="card" style="width: 18rem;">
        <%= image_tag(song.cover.url, class: 'card-img-top', alt: song.title) if song.cover? %>
        <div class="card-body">
          <h5 class="card-title"><%= song.title %></h5>
          <p class="card-text">
            <%= song.artist %>
          </p>
          <%= link_to 'See this song', song_path(song), class: 'btn btn-primary' %>
        </div>
      </div>
    <% end %>
  </div>
  
  <%= paginate @songs %>
  ```

```ruby
gem "font-awesome-rails"
```

- `custom.scss`

  ```scss
  @import "font-awesome";
  ```

---

> devise 의 숨은 컨트롤러를 확인

```shell
$ rails g devise:controllers users
```

