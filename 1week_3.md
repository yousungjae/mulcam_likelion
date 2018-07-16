# 1week_3

## 06.08(금)

#### Codecademy Ruby 따라해보기(ch4 까지)

#### 1. Lotto

- `jackpot`

  - `check_lotto.rb`

  ```ruby
  # version 1
  
  real_numbers = [*1..6]
  bonus_number = 7
  lucky_numbers = (1..45).to_a.sample(6).sort
  
  match_numbers = real_numbers & lucky_numbers
  jack = match_numbers.count
  
  # 1
  if jack == 6 then puts '1등'
  elsif jack == 5 && lucky_numbers.include?(bonus_number)
      puts '2등'
  elsif jack == 5 then puts '3등'
  elsif jack == 4 then puts '4등'
  elsif jack == 3 then puts '5등'
  else puts '땡'
  end
  
  # 2
  result = 
      case [jack, lucky_numbers.include?(bonus_number)]
      when [6, false] then '1등'
      when [5, true] then '4등'
      when [5, false] then '3등'
      when [4, true||false] then '4등'
      when [3, true||false] then '5등'
      else '땡'
      end
  puts result
  ```

  ```ruby
  # version 2
  
  require 'json'
  require 'open-uri'
  
  url = 'http://www.nlotto.co.kr/common.do?method=getLottoNumber&drwNo=809'
  
  info = JSON.parse(open(url).read)
  
  lucky_numbers = [*1..45].sample(6)
  real_numbers = []
  bonus_number = info['bnusNo']
  
  info.each do |k, v|
      real_numbers << v if k.include?('drwtNo')
  end
  
  match_numbers = real_numbers & lucky_numbers
  jack = match_numbers.count
  
  # 1
  # if jack == 6 then puts '1등'
  # elsif jack == 5 && lucky_numbers.include?(bonus_number)
  #     puts '2등'
  # elsif jack == 5 then puts '3등'
  # elsif jack == 4 then puts '4등'
  # elsif jack == 3 then puts '5등'
  # else puts '땡'
  # end
  
  # 2
  result = 
      case [jack, lucky_numbers.include?(bonus_number)]
      when [6, false] then '1등'
      when [5, true] then '4등'
      when [5, false] then '3등'
      when [4, true||false] then '4등'
      when [3, true||false] then '5등'
      else '꽝'
      end
  puts result
  ```

#### 2. Forecast

- fore_cast

  - `forecast.rb`

  - `Mash` : Method 와 Hash 의 합성어로 hash 를 method 처럼 사용하기 위해 만들어진 것 (gem hashie)

    ```ruby
    require 'forecast_io'
    
    ForecastIO.configure do |configuration|
      configuration.api_key = 'API-KEY'
    end
    
    forecast = ForecastIO.forecast(37.504635, 127.040147)
    
    def f_to_c(f)
        ((f - 32) / 1.8).round(2)
    end
        
    
    puts forecast['currently']['summary']
    # == puts forecast.currently.summary
    temp = forecast.currently.apparentTemperature
    puts f_to_c(temp)
    ```

  - `geocoder.rb`

    ```ruby
    require 'geocoder'
    
    input = gets.chomp
    cord = Geocoder.coordinates(input)
    
    puts cord
    puts cord.last
    ```

  - `forecast_geocoder.rb`

    ```ruby
    require 'forecast_io'
    require 'geocoder'
    
    ForecastIO.configure do |configuration|
      configuration.api_key = 'API-KEY'
    end
    
    print "위치를 입력해주세요: "
    input = gets.chomp
    cord = Geocoder.coordinates(input)
    
    forecast = ForecastIO.forecast(cord.first, cord.last)
    
    def f_to_c(f)
        ((f - 32) / 1.8).round(2)
    end
        
    
    puts forecast['currently']['summary']
    temp = forecast.currently.apparentTemperature
    puts f_to_c(temp)
    ```

    ```ruby
    # 확장 버전
    
    require 'forecast_io'
    require 'geocoder'
    
    ForecastIO.configure do |configuration|
      configuration.api_key = '1363e7c52834a5e73fd1ff99e86a57ce'
    end
    
    print "위치를 입력해주세요: "
    input = gets.chomp
    cord = Geocoder.coordinates(input)
    
    forecast = ForecastIO.forecast(cord.first, cord.last)
    
    
    class Float
      def convert_temp(to)
        if to == 'c'
          ((self - 32) / 1.8).round(2)
        elsif to == 'f'
          ((self * 1.8) + 32).round(2)
        else
          raise ArgumentError, "Arument must be 'c' or 'f', '#{to}' is can't"
        end
      end
    end
        
    puts forecast['currently']['summary']
    puts forecast.currently.apparentTemperature.convert_temp('c')
    ```

    