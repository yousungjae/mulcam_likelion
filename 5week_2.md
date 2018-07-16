# 5week_2

## 07.03(화)

## 실습 : practice_app

### 0. 준비 

* Model : Posting - title: string, content: text, (img: string)
  * Controller(CRUD) : Posting with 7 action 
* Model : Feedback - content: string
  * Controller(CRD) : Feedbacks with 2 actions (create, destroy)
* gems : tinymce, carrierwave & minimagick, bootstrap etc....

### 1. Modeling 

* `$ rails g model Posting title:string, content:text`

* `$ rails g model Feedback content:string, posting:belongs_to`

* `$ rails db:migrate`

* validates

  * postring.rb

    ```ruby
    class Posting < ApplicationRecord
      has_many :feedbacks, dependent: :destroy
      
      validates :title, presence: true, length: { minimum: 2, maximum: 100 }
      validates :content, presence: true, length: { minimum: 2 }
    end
    ```

  * feedback.rb

    ```ruby
    class Feedback < ApplicationRecord
      belongs_to :posting
      
      validates :content, presence: true, length: { maximum: 200 }
    end
    ```

### 2. Postings Controller ( CRUD ) 

* `$ rails g controller postings`

* routes.rb

  ```ruby
  Rails.application.routes.draw do
    resources :postings
  end
  ```

* postrings_controller.rb

  ```ruby
  class PostingsController < ApplicationController
    before_action :set_posting, only: [:show, :edit, :update, :destroy]
    
    def new
      @posting = Posting.new
    end
    
    def create
      @posting = Posting.new(posting_params)
      if @posting.save
        redirect_to posting_url(@posting.id)
      else
        redirect_to new_posting_url
      end
    end
  
    def show
    end
  
    def index
      @postings = Posting.all
    end
  
    def edit
    end
    
    def update
      if @posting.update(posting_params)
        redirect_to posting_url(@posting)
      else
        redirect_to edit_posting_url(@posting)
      end
    end
    
    def destroy
      @posting.destroy
      redirect_to postings_url
    end
    
    private
      def set_posting
        @posting = Posting.find params[:id]
      end
      
      def posting_params
        params.require(:posting).permit(:title, :content)
      end
  end
  ```

* index.html.erb

  ```erb
  <h1>Postings List</h1>
  <%= link_to 'New posting', new_posting_path %>
  <ul>
    <% @postings.each do |posting| %>
      <li>
        <%= link_to posting.title, posting_path(posting) %>
      </li>
    <% end %>
  </ul>
  ```

* show.html.erb

  ```erb
  <h1><%= @posting.title %></h1>
  <p>
    <%= @posting.content.html_safe %>
  </p>
  
  <div>
  <%= link_to 'List', postings_path %>
  |
  <%= link_to 'edit', edit_posting_path %>
  |
  <%= link_to 'del', posting_path(@posting), method: :delete %>
  </div>
  ```

* _form.html.erb

  ```erb
  <%= form_for(posting) do |f| %>
    <div>
      <%= f.label :title, '제목' %>
      <%= f.text_field :title %>
    </div>
    
    <div>
      <%= f.label :content, '내용' %>
      <%= f.text_area :content, rows: 40, cols: 120 %>
    </div>
    
    <div>
      <%= f.submit %>
    </div>
  <% end %>
  ```

* new.html.erb

  ```erb
  <h1>New Posting</h1>
  <%= render 'form', posting: @posting %>
  ```

* edit.html.erb

  ```erb
  <h1>Edit Posting</h1>
  <%= render 'form', posting: @posting %>
  ```

### 3. Feedback Controller ( CRD )

* `$ rails g controller feedbacks`

* route.rb

  ```ruby
  Rails.application.routes.draw do
    resources :postings
    resources :feedbacks, only: [:create, :destroy]
  end
  ```

* postings_controller.rb

  ```ruby
  class PostingsController < ApplicationController
  	...
    def show
      @feedbacks = @posting.feedbacks # Feedback.where(posting_id: @posting.id)
      @feedback = Feedback.new
    end
    ...
  end
  ```

* feedbacks_controller.rb

  ```ruby
  class FeedbacksController < ApplicationController
    def create
      feedback_params = params.require(:feedback).permit(:content, :posting_id)
      @feedback = Feedback.new(feedback_params)
      if @feedback.save
        redirect_to posting_url(@feedback.posting)
      else
        redirect_to posting_url(params[:feedback][:posting_id])
      end
    end
    
    def destroy
      @feedback = Feedback.find params[:id]
      @feedback.destroy
      redirect_to posting_url(@feedback.posting)
    end
  end
  ```

* postings/show.html.erb

  ```erb
  <h1><%= @posting.title %></h1>
  <p>
    <%= @posting.content.html_safe %>
  </p>
  
  <div>
    <%= image_tag @posting.image.url if @posting.image? %>
  </div>
  
  <div>
  <%= link_to 'List', postings_path %>
  |
  <%= link_to 'edit', edit_posting_path %>
  |
  <%= link_to 'del', posting_path(@posting), method: :delete %>
  </div>
  
  <div>
    <%= form_for(@feedback) do |f| %>
      <%= f.hidden_field :posting_id, value: @posting.id %>
      <%= f.label :content, 'comment' %>
      <%= f.text_field :content %>
      <%= f.submit %>
    <% end %>
  </div>
  
  <div>
    <ul>
      <% @feedbacks.each do |feedback| %>
        <li>
          <%= feedback.content %> |
          <%= link_to 'del', feedback_path(feedback), method: :delete %>
        </li>
      <% end %>
    </ul>
  </div>
  ```

### 4. gem 'carrierwave'

* Posting에 column 추가

  * `$ rails g migration add_image_to_postings image`
  * `$ rails db:migrate`

* Gemfile - gem 추가

  ```ruby
  ...
  #(Uploader)form 에서 input type=file 을 업로드 가능케 해줌 
  gem 'carrierwave'
  #이미지 프로세싱(resize, thumbnail 제작)
  gem 'mini_magick'
  ...
  ```

* `$ bundle`

* config/environment.rb - 

  ```ruby
  # Load the Rails application.
  require_relative 'application'
  
  require 'carrierwave/orm/activerecord' # 추가
  
  # Initialize the Rails application. 
  Rails.application.initialize!
  ```

* `$ rails generate uploader picture`

  * picture_uploader.rb

    ```ruby
    class PictureUploader < CarrierWave::Uploader::Base
      include CarrierWave::MiniMagick
        
      if Rails.env.production?
        storage :fog
      else
        storage :file
      end
      
      def store_dir
        "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
      end
      
      version :thumb do
        process resize_to_fit: [75, 75]
      end
      
      def extension_whitelist
        %w(jpg jpeg gif png)
      end
    end
    ```

* Posting.rb - column mout & validate 추가

  ```ruby
  class Posting < ApplicationRecord
    has_many :feedbacks
    mount_uploader :image, PictureUploader
    
    validates :title, presence: true, length: { minimum: 2, maximum: 100 }
    validates :content, presence: true, length: { minimum: 2 }
    
    validate :image_size
    
    private
      def image_size
        if image.size > 10.megabyte
          error.add(:image, "10mb 보다 작아야 합니다")
        end
      end
  end
  ```

* posting_controller.rb - params permit 추가

  ```ruby
  class PostingsController < ApplicationController
  ...
     def posting_params
        params.require(:posting).permit(:title, :content, :image)
      end
  end
  ```

* _form.html.erb - image 항목 추가

  ```erb
  <%= form_for(posting) do |f| %>
    <div>
      <%= f.label :title, '제목' %>
      <%= f.text_field :title %>
    </div>
    
    <div>
      <%= f.label :content, '내용' %>
      <%= f.text_area :content, rows: 40, cols: 120 %>
    </div>
    
    <div>
      <%= f.label :image, '이미지' %>
      <%= f.file_field :image, accept: "image/jpeg, image/jpg, image/gif, image/png" %>
    </div>
    
    <div>
      <%= f.submit %>
    </div>
  <% end %>
  ```

* show.html.erb

  ```erb
  <h1><%= @posting.title %></h1>
  ...
  <div>
    <%= image_tag @posting.image.url if @posting.image? %>
  </div>
  ...
  ```

* index.html.erb - thumbnail 추가

  ```erb
  <h1>Postings List</h1>
  <%= link_to 'New posting', new_posting_path %>
  <ul>
    <% @postings.each do |posting| %>
      <li>
        <%= image_tag posting.image.thumb.url if posting.image? %>
        <%= link_to posting.title, posting_path(posting) %>
      </li>
    <% end %>
  </ul>
  ```

### 5. gem bootstrap & tinymce & Font Awesome

* Gemfile - gem 추가

  ```ruby
  ...
  # Rails 용 JQuery
  gem 'jquery-rails'
  # Bootstrap v4
  gem 'bootstrap', '~> 4.1.1'
  # Font Awesome
  gem "font-awesome-rails"
  # 게시판의 form 태그안에 textarea 를 에디터로 바꿔줌
  gem 'tinymce-rails'
  ...
  ```

* bootstrap - 설정 및 적용

  * app/assets/javascripts/application.js

    ```javascript
    //
    //= require rails-ujs
    //= require turbolinks
    //= require jquery3
    //= require popper
    //= require bootstrap-sprockets
    ...
    ```

  * app/assets/stylesheets/custom.scss - 생성

    ```scss
    @import "bootstrap";
    ```

  * application.html.erb - 수정

    ```erb
    ...
      <body>
        <div class="container">
          <%= yield %>
        </div>
      </body>
    ...
    ```

    

* tinymce - 설정 및 적용

  * config/tinymce.yml 생성

    ```yaml
    toolbar:
      - styleselect | bold italic | undo redo
      - image | link
    plugins:
      - image
      - link
    ```

  * app/assets/javascripts/application.js

    ```
    //
    //= require rails-ujs
    //= require turbolinks
    //= require jquery3
    //= require popper
    //= require bootstrap-sprockets
    //= require tinymce-jquery
    ...
    ```

  * _form.html.erb - tinymce form 변경

    ```erb
    ...
      <div>
        <%= f.label :content, '내용' %>
        <%= f.text_area :content, class: 'tinymce', rows: 40, cols: 120 %>
        <%= tinymce %>
      </div>
     ...
    ```

* Font Awesome - 설정 및 적용

  * app/assets/stylesheets/custom.scss - 수정

    ```scss
    @import "bootstrap";
    @import "font-awesome";
    ```

  * show.html.erb - 수정

    ```erb
    <h1><%= @posting.title %></h1>
    <p>
      <%= @posting.content.html_safe %>
    </p>
    
    <div>
      <%= image_tag @posting.image.url if @posting.image? %>
    </div>
    
    <div>
    <%= link_to fa_icon('list'), postings_path %>
    |
    <%= link_to fa_icon('edit'), edit_posting_path %>
    |
    <%= link_to fa_icon('trash'), posting_path(@posting), method: :delete %>
    </div>
    
    <div>
      <%= form_for(@feedback) do |f| %>
        <%= f.hidden_field :posting_id, value: @posting.id %>
        <%= f.label :content, fa_icon('comment') %>
        <%= f.text_field :content %>
        <%= f.submit %>
      <% end %>
    </div>
    
    <div>
      <ul>
        <% @feedbacks.each do |feedback| %>
          <li>
            <%= feedback.content %> |
            <%= link_to (fa_icon 'trash'), feedback_path(feedback), method: :delete %>
          </li>
        <% end %>
      </ul>
    </div>
    ```

    