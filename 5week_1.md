# 5week_1

## 07.02(월)

### 1. 1 : N (Song - Comment)

#### 1.1 모델 관계 설정

- Song (1) : Comment (N) - 하나의 노래에 여러 글이 달릴 수 있다.

```shell
$ rails g resource Comment content
# or
$ rails g resource Comment content song:belongs_to
```

- `20180702013939_create_comments.rb`

  ```ruby
  class CreateComments < ActiveRecord::Migration[5.1]
    def change
      create_table :comments do |t|
        t.string :content
        t.belongs_to :song
  
        t.timestamps
      end
    end
  end
  ```

```shell
$ rails db:migrate
```

- `comment.rb`

  ```ruby
  class Comment < ApplicationRecord
      # song 에 속해 있다.
      belongs_to :song
  end
  ```

- `song.rb`

  ```ruby
  class Song < ApplicationRecord
      # 여러 개의 comments를 가지고 있다.    
      has_many :comments
      ...
  end
  ```

#### 1.2 댓글 써보기

- `comment.rb`

  ```ruby
  class Comment < ApplicationRecord
    # song 에 속해 있다.
    belongs_to :song
    
    validates :content, presence: true, length: { minimum: 2, maximum: 50 }
  end
  ```

- 콘솔실습

  ```shell
  $ c = Comment.new
  $ song = Song.create! title: '제목', artist: '가수', lyric: '가사'
  $ c.content = '인생노래'
  $ c.valid?
  => false
  $ c.errors.full_messages
  => ["Song must exist"]
  
  $ c.song = song 
  $ c.song_id = song.id
  => 같은 결과
  
  $ c.save
  # 위 과정 반복, 각 노래에 여러 댓글 달아보기
  
  # 해당 노래가 가지고 있는 모든 Comments 가져오기
  $ Comment.where(song_id: 2)
  $ Song.find(2).comments
  => 같은 결과
  
  # 해당 댓글이 어떤 글에 속해 있는지 확인
  $ comment.song
  ```

#### 1.3 댓글 나타내기

- `song_controller.rb`

  ```ruby
  class SongsController < ApplicationController
  	...
    def show
      @comment = Comment.new
      @comments = @song.comments
    end
  	...
  end
  ```

- `show.html.erb`

  ```erb
  	...
  <!--댓글 new-->
  <%= form_for(@comment) do |f| %>
    <%= f.label :content %>
    <%= f.text_field :content %>
    <%= f.hidden_field :song_id, value: @song.id %>
    <%= f.submit %>
  <% end %>
  
  <!--댓글 show-->
  <ul id="comments">
    <% @comments.each do |comment| %>
    <li><%= comment.content %></li>
    <% end %>
  </ul>
  ```

- `comments_controller.rb`

  ```ruby
  class CommentsController < ApplicationController
    def create
      @comment = Comment.new  
      @comment.content = params[:comment][:content]
      @comment.song_id = params[:comment][:song_id]
      if @comment.save
        redirect_to song_url @comment.song_id
        # == redirect_to song_url @comment.song
        # == redirect_to @comment.song
      else
        redirect_to song_url @comment.song_id
      end
    end
  end
  ```

- `routes.rb`

  ```ruby
  Rails.application.routes.draw do
    resources :songs
    resources :comments
  	...
  end
  ```

#### 1.4 댓글 삭제하기

- `show.html.erb`

  ```erb
  <!--댓글 show-->
  <ul id="comments">
    <% @comments.each do |comment| %>
    <li>
      <%= comment.content %>
      <%= link_to '삭제', comment_path(comment), method: :delete %>
    </li>
    <% end %>
  </ul>
  ```

- `comments_controller.rb`

  ```ruby
  	...
  def destroy
    @comment = Comment.find params[:id]
    @comment.destroy
    redirect_to song_url @comment.song_id
  end
  ```

- `routes.rb`

  ```ruby
  Rails.application.routes.draw do
    resources :songs
    resources :comments, only: [:create, :destroy]
  	...
  end
  ```

- `song.rb`

  ```ruby
  class Song < ApplicationRecord
    # 여러 개의 comments를 가지고 있다.
    # 그리고 song 이 삭제되면, comment 도 삭제된다.
    has_many :comments, dependent: :destroy
      ...
  end
  ```

---

### 2. TinyMCE (text editor)

#### 2.1 [Getting Started](https://github.com/spohlenz/tinymce-rails)

```ruby
gem 'tinymce-rails'
```

```shell
$ bundle
```

- config/`tinymce.yml` 생성

  ```yaml
  toolbar:
    - styleselect | bold italic | undo redo
    - image | link
  plugins:
    - image
    - link
  ```

- `application.js`

  ```javascript
  //= require tinymce-jquery
  //= require rails-ujs
  //= require turbolinks
  //= require_tree .
  ```

- `_form.html.erb`

  ```erb
  <%= form_for(song) do |f| %>
  	...
    <div>
      <%= f.label :lyric,'노래가사' %>
      <%= f.text_area :lyric, class: "tinymce", rows: 40, cols: 120 %>
      <%= tinymce %>
    </div>
  	...
  <% end %>
  ```

- `show.html.erb`

  ```erb
  	...
  <p>
    <%= @song.lyric.html_safe %>
  </p>
  	...
  ```

---

### 3. Pagination (by `gem Kaminari`)

#### 3.1 [Getting Started](https://github.com/kaminari/kaminari)

```ruby
gem 'kaminari'
```

```shell
$ bundle
```

- `songs_controller.rb`

  ```ruby
  	...
    def index
      @songs = Song.all.page params[:page]
    end
  	...
  ```

- `index.html.erb`

  ```erb
  	...
  <%= paginate @songs %>
  ```

- db/`seeds.rb`

  ```ruby
  500.times do |i|
    Song.create(title: "#{i+1}번 노래", artist: 'harry', lyric: 'Hello World !')
  end
  ```

```shell
$ rails db:seed
# index 페이지 확인
```

```shell
$ rails db:reset
$ rails db:migrate
```
---

### 4. 가짜 데이터 넣기 (by `gem faker`)

#### 4.1 [Getting Started](https://github.com/stympy/faker)

```ruby
gem 'faker'
```

```shell
$ bundle
```

- `seeds.rb`

  ```ruby
  500.times do |i|
    if i.even?
      Song.create!(title: Faker::Lorem.word, artist: Faker::Artist.name, lyric: Faker::Lorem.paragraph(5))
    else
      Song.create!(title: Faker::Pokemon.name, artist: Faker::RockBand.name, lyric: Faker::Lorem.paragraph(5))
    end
  end
  ```

  