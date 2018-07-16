# 4week_5

## 06.29(금)

### 1. 이미지

#### 1.1 이미지를 저장하는 방법

1) DB 에 억지로 저장

2) 컴퓨터에 저장

3) 외부 storage 저장

#### 1.2 컴퓨터에 저장 (사전 작업)

```shell
sudo apt-get install -y imagemagick
```

> -y : 설치 중 묻는 동의 여부에 모두 yes 라고 자동 응답.

```ruby
gem 'carrierwave'
gem 'mini_magick'
```

```shell
$ rails g uploader Picture
$ rails g migration add_cover_to_songs cover:string
$ rails db:migrate
```

- `songs_controller.rb` : `:cover` 파라미터 값 추가

  ```ruby
   								... 
    def song_params
      params.require(:song).permit(:title, :artist, :lyric, :cover)
    end
  end
  ```

- `song.rb`

  ```ruby
  class Song < ApplicationRecord
    mount_uploader :cover, PictureUploader
    
  										...
          
    validate :cover_size
    
    private 
    def cover_size
      error.add(:cover, "should be less then 5 MB") if cover.size > 5.megabyte
    end
  end
  ```

- `environment.rb` : [참고](https://github.com/carrierwaveuploader/carrierwave)

  ```ruby
  # Load the Rails application.
  require_relative 'application'
  require 'carrierwave/orm/activerecord'
  
  # Initialize the Rails application.
  Rails.application.initialize!
  ```

- `_form.html.erb` : file__field 추가

  ```erb
  										...
    <div>
      <%= f.label :lyric,'노래가사' %>
      <%= f.text_area :lyric, cols: 200, rows: 20 %>
    </div>
    <div>
      <%=f.file_field :cover, accept: 'image/jpeg, image/gif, image/png' %>
    </div>
    <div>
      <%= f.submit %>
    </div>
  <% end %>
  ```

- `show.html.erb`

  ```erb
  <h1><%= @song.title %></h1>
  <h2><%= @song.artist %></h2>
  <%= image_tag @song.cover.url if @song.cover? %>
  <p>
    <%= @song.lyric %>
  </p>
  										...
  ```

- `picture_uploader.rb`

  ```ruby
  class PictureUploader < CarrierWave::Uploader::Base
    include CarrierWave::MiniMagick
    # 가로/세로 크기 제한 부여
    # process resize_to_limit: [400, 400]
  
    # Choose what kind of storage to use for this uploader:
    if Rails.env.production?
      storage :fog
    else
      storage :file
    end
  
    # Override the directory where uploaded files will be stored.
    # This is a sensible default for uploaders that are meant to be mounted:
    def store_dir
      "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
    end
  
  										...
          
          
    # Create different versions of your uploaded files:
    version :thumb do
      process resize_to_fit: [50, 50]
    end
      
    def extension_whitelist
      %w(jpg jpeg gif png)
    end
  
  										...
  end
  ```

#### 1.3 썸네일도 만들어 보기

- `index.html.erb`

  ```erb
  <h1>Playlist</h1>
  <h2><%= link_to 'New song', new_song_path %></h2>
  <ul>
    <% @songs.each do |song| %>
      <li><%= image_tag song.cover.thumb.url if song.cover? %><%= link_to "#{song.title} - #{song.artist}", song_path(song) %></li>
    <% end %>
  </ul>
  ```

  

