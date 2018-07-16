# 1week_2

## 06.07(목)

codecademy HTML 맛보기

### terminal

CLI - command line interface

$ - 프롬프트 (명령어 받을 준비 완료)

`ctrl + c` : 강제 종료

`ctrl + p` : 이전 명령어

`clear / ctrl + l` : 커맨드 창 정리

`ctrl + a` : 커서를 명령어 가장 앞으로

`ctrl + e` : 커서를 명령어 가장 뒤로

`ctrl + u` : 명령어 삭제

`ctrl + w` : 커서가 있는 단어를 삭제

```bash
$ echo 'Hello, world'	// 출력
$ man echo 				// echo 매뉴얼
```

```bash
실습 #1
1. man sleep
2. 5초간 잠재우기
$ sleep 5
3. 100초 재운 후 깨워보기
$ sleep 100
ctrl + c
```

```bash
// redirect (덮어씌여지기 때문에 내용을 덧붙일 수 없다.)
$ echo 'i cant stop to keep myself from thinking' > practice.txt

// 파일 내용 확인 (concatate)
$ cat practice.txt

// append
$ echo 'Im your father' >> practice.txt
```

```bash
실습 #2
$ echo 'hello !' > line_1.txt
$ echo 'bye !' > line_2.txt
$ cat line_1.txt line_2.txt > song.txt 
$ cat line_2.txt line_1.txt > song_reverse.txt
```

```bash
$ ls -a -l -t
$ ls -alt

// 파일 만들기
$ touch not_hidden.txt
$ touch .hidden.txt (숨김 파일)

// 확장자 붙이기 / 이름 바꾸기
$ echo 'text text' > test
$ mv test text.txt
$ mv text.txt text_file.txt

// 복사
$ cp test_file.txt copy_file.txt

// 삭제
$ rm copy_file.txt
$ rm *.txt
```

### Intro to Web service

Web Site 가 아닌 Web Service 를 만든다.

HTML / CSS / JS

**Static Web**

**Dynamic Web (Web application program)**

**URL (Uniform Resource Locator)** - 서버에 요청할 수 있는 유일한 방법

---

### Play Ruby

#### 1. stock_currency

- `stock.rb`

  - [gem stock_quote](https://github.com/tyrauber/stock_quote)
  - `gem install stock_quote`

  ```ruby
  # 1
  require 'stock_quote'
  
  stock = StockQuote::Stock.quote('amzn')
  
  puts stock
  puts stock.company_name
  puts stock.latest_price
  ```

  ```ruby
  # 2
  require 'stock_quote'
  
  stocks = StockQuote::Stock.quote('amzn, fb, tsla, aapl')
  
  stocks.each do |s|
      puts s.company_name
      puts s.latest_price
      puts '====='
  end
  ```

  ```ruby
  # 3
  require 'stock_quote'
  
  print '검색할 주식의 symbol을 입력하세요: '
  
  input = gets.chomp
  input = input.split
  
  stocks = StockQuote::Stock.quote(input)
  
  stocks.each do |s|
      puts s.company_name
      puts s.latest_price
      puts '====='
  end
  ```

  ```ruby
  # 4
  require 'stock_quote'
  
  print '검색할 주식의 symbol을 입력하세요: '
  
  input = gets.chomp
  input = input.split
  
  stocks = StockQuote::Stock.quote(input)
  
  if stocks.class == Array  # stocks.is_a? Array
      stocks.each do |s|
          puts "#{s.company_name}의 1 주당 가격은 $#{s.latest_price} 입니다."
      end
  else
      puts "#{stocks.company_name}의 1 주당 가격은 $#{stocks.latest_price} 입니다."
  end
  ```

- `currency.rb`

  - [gem eu_central_bank](https://github.com/RubyMoney/eu_central_bank)
  - `gem install eu_central_bank`

   ```ruby
  require 'eu_central_bank'
  
  bank = EuCentralBank.new
  bank.update_rates
  
  rate = bank.exchange(100, "USD", "KRW")
  
  puts rate
   ```

- `stock_currency.rb`

  - 





