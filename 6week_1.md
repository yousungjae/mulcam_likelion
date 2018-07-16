# 6week_1

## 07.09(월)

### 1. 카카오톡 챗봇 만들기

> Rails 템플릿으로 새로운 c9 프로젝트 생성

[강의 전반부](https://github.com/5chang2/kakao_bot_sample)

[카카오톡 플러스친구 API v. 2.0 개요 README](https://github.com/plusfriend/auto_reply/blob/master/README.md#63-photo)

#### 1.1 친구 추가/차단 알림

[5.3 친구 추가/차단 알림 API](https://github.com/plusfriend/auto_reply/blob/master/README.md#53-%EC%B9%9C%EA%B5%AC-%EC%B6%94%EA%B0%80%EC%B0%A8%EB%8B%A8-%EC%95%8C%EB%A6%BC-api)

```shell
$ rails g model User chat_room:integer user_key:string
$ rake db:migrate
```

- `routes.rb`

  ```ruby
  Rails.application.routes.draw do
  	...
    post 'friend' => 'kakao#friend_add'
    delete "friend/:user_key" => 'kakao#friend_delete'
  end
  ```

- `kakao_controller.rb`

  ```ruby
  	... 
    def friend_add
      User.create(chat_room: 0, user_key: params[:user_key])
      render nothing: true
    end
    
    def friend_delete
      User.find_by(user_key: params[:user_key]).destroy
      render nothing: true
    end
  end
  ```

>  친구 삭제/추가 해서 모델에 유저 생성/삭제 확인

---

#### 1.2 사용자의 채팅방 나간 횟수 카운트

[5.4 채팅방 나가기](https://github.com/plusfriend/auto_reply/blob/master/README.md#54-%EC%B1%84%ED%8C%85%EB%B0%A9-%EB%82%98%EA%B0%80%EA%B8%B0)

- `user.rb`

  ```ruby
  class User < ActiveRecord::Base
      def plus
          self.chat_room += 1
      end
  end
  ```

- `routes.rb`

  ```ruby
  Rails.application.routes.draw do
  	...
    delete "chat_room/:user_key" => "kakao#chat_room"
  end
  ```

- `kakao_controller.rb`

  ```ruby
  	...
    def chat_room
      @user = User.find_by(user_key: params[:user_key])
      @user.plus
      @user.save
      render nothing: true
    end
  end
  ```

> 채팅방을 나갈 때마다 user 의 chat_room  숫자가 1 씩 증가
>
> 챗봇을 차단할 경우 모델에서 사라짐