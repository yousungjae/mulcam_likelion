# 2week_4

## 06.15(금)

- toy_app

`rails g controller stock`

`rails g controller fake`

컨트롤러 삭제 : `rails d controller fake`

- routes.rb

  ```ruby
  Rails.application.routes.draw do
      root 'home#index'
      get '/hello', to: 'home#hello'  
      get '/lotto', to: 'home#lotto'
      
      get '/stock/input', to: 'stock#input'
      get '/stock/output', to: 'stock#output'
      
      get '/stock/one_step/:data', to: 'stock#one_step'
  end
  ```

- stock_controller.rb

  ```ruby
  class StockController < ApplicationController
    def input
    end
    
    def output
      input_data = params[:symbol]
      @stock_info = StockQuote::Stock.quote(input_data)
    end
    
    def one_step
      input_data = params[:data]
      @stock_info = StockQuote::Stock.quote(input_data)
    end
  end
  ```

---

- `rails g controller Forecast input output one_step`

- forecast_controller.rb

  ```ruby
  class ForecastController < ApplicationController
    before_action :setup, only: [:output, :one_step]
    # before_action(:setup, { only: [:output, :one_step] })
    # before_action(:setup, { :only => [:output, :one_step] })
    # 마지막으로 들어오는 인자가 해쉬라면 {} 생략 가능
    
    def input
    end
  
    def output
      setup
    end
  
    def one_step
      setup
    end
    
    
    private
    def f_to_c(f)
      ((f - 32) / 1.8).round(2)
    end
    
    def setup
      ForecastIO.configure do |configuration|
        configuration.api_key = '1363e7c52834a5e73fd1ff99e86a57ce'
      end
      cord = Geocoder.coordinates(params[:location])
      currently = ForecastIO.forecast(cord.first, cord.last).currently
      @temp = f_to_c(currently.apparentTemperature)
      @summary = currently.summary
    end
  end
  ```

- input.html.erb

  ```erb
  <h1>Forecast#input</h1>
  <form action="/forecast/output">
    <input type="text" name="location"></input>
    <input type="submit"></input>
  </form>
  ```

- output.html.erb

  ```erb
  <h1>Forecast#output</h1>
  <p><%= @summary %></p>
  <p><%= @temp %></p>
  ```

---

### Symbol

- 루비 symbol도 string과 밀접하며, 루비 인터프리터는 심볼 테이블에 모든 클래스, 매서드, 변수를 저장한다. 
  이 테이블에 자신만의 심볼을 추가할 수 있다. 특별히 심볼은 이름 앞에 콜론`:`을 붙여 만든다.
- 루비 symbol 은 이름과 문자열을 대표하곤 한다.
  그러나 `String` 객체와는 다르게 같은 이름의 Symbol 은 하나의 루비 세션 동안 단 한번만 초기화되며 메모리에 존재한다.
- Symbol 은 메모리에 단 한번만 저장되어 메모리 공간 활용에 유리하다.

```ruby
puts :name.object_id   # => yields 20488 
puts :name.object_id   # => yields 20488 
puts "name".object_id  # => yields 2168472820 
puts "name".object_id  # => yields 2168484060 
```

- Parameter 의 마지막 인자는 보통 hash 이며, 만일 마지막 인자가 해쉬인 경우 중괄호 {} 생략 가능.

```ruby
before_action :setup, only: [:output, :one_step]
before_action(:setup, { only: [:output, :one_step] })
before_action(:setup, { :only => [:output, :one_step] })
```

- hash 내부에서 심볼과 해쉬 로켓의 변경된 문법

```ruby
h1 == h2
h1 = { :name => 'yty', :email => 'neo@gmail.com' }
h2 = { name: 'yty', email: 'neo@gmail.com' }		# 최신 문법
```

```bash
$ h2
$ h2 = { :name => 'yty', :email => 'neo@gmail.com' }  # 인식은 예전 문법으로 알아서 한다.
```

---

### Scaffold 맛보기

- `rails g scaffold Micropost writer title content`
- `rails db:migrate`

---

### Model

- 생성(Create)

  ```ruby
  member = Member.new
  member.name = 'harry'
  member.email = 'asddasd@naver.com'
  ...
  member.save
  ```

- 조회(Read, Retrieve)

  ```ruby
  member = Member.find 1
  member
  ```

- 수정(Update) 

  ```ruby
  m = Member.find 1
  m.name = 'neo'
  m.save
  ```

- 삭제(Delete, Destroy)

  ```ruby
  m = Member.find 1
  m.delete
  ```

> 모델이 데이터베이스 그 자체는 아니다.

- `rails g model Member`

- 20180615072254_create_members.rb

  ```ruby
  class CreateMembers < ActiveRecord::Migration[5.1]
    def change
      create_table :members do |t|
        t.string :name
        t.string :email
        t.integer :age
        t.boolean :married
  
        t.timestamps
      end
    end
  end
  ```

- `rails db:migrate`

- 배포 & 테스트 그룹에  `gem 'rails_db'` /  `gem 'pry-rails'` 추가 => `bundle`

- `rails c`

  - 레일즈 콘솔

- `rails c --sandbox`

  - 테스트용 콘솔 (실제 프로젝트에 반영되지 않음)