# 3week_4

## 06.21(목)

### Sample_app 세팅

- `rails new sample_app`

```ruby
# 개발, 테스트로 이동
gem 'sqlite3'

# gem 추가
gem 'pry-rails'
gem 'jquery-rails'

group :test do
  gem 'rails-controller-testing'
  gem 'minitest'
  gem 'minitest-reporters'
  gem 'guard'
  gem 'guard-minitest'
end

group :production do
  gem 'pg'
end
```

- `bundle`
- `git add .`
- `git commit -m ''`
- `git remote add origin <개인 비트버킷 주소>`
- `git push -u origin master` 
- `heroku login`
- `heroku create`
- `git push heroku master`
- `git checkout -b static-pages`
- `rails g controller StaticPages home help`
- heroku run 뒤에 명령어 작성 시 heroku 서버에서 실행됨
  - ex) `heroku run rails c`

### Test-drigen Development 테스트 주도 개발

#### 1. html title 을 이용한 테스트 개발 체험 

- 명령어 `rails test` | `rails t`
- 테스트로 오류를 확인하면서 about 의 view action routing 을 완성해본다.
- `app/test/controllers/static_pages_controller_test`

```ruby
require 'test_helper'

class StaticPagesControllerTest < ActionDispatch::IntegrationTest
  def setup
    @base_title = "Harry's blog"
  end
  
  test "should get home" do
    get static_pages_home_url
    assert_response :success
    assert_select "title", "Home | #{@base_title}"
  end

  test "should get help" do
    get static_pages_help_url
    assert_response :success
    assert_select "title", "Help | #{@base_title}"
  end
  
  test "should get about" do
    get static_pages_about_url
    assert_response :success
    assert_select "title", "About | #{@base_title}"
  end
end
```

- `application.html.erb`

```erb
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield(:title) %> | Harry's blog </title>
      
	...
      
  </head>

  <body>
    <%= yield %>
  </body>
</html>
```

- `home.html.erb`

```erb
<% provide(:title, "Home") %>
<h1>Harry's blog</h1>
<p>
  This is the blog
</p>
```

- `help.html.erb`

```erb
<% provide(:title, "Help") %>
<h1>Help</h1>
<p>
  Do u need help ?
  <a href="https://github.com/neovansoarer" target="_blank">Harry's github</a>
</p>
```

- `about.html.erb`

```erb
<% provide(:title, "About") %>
<h1>About</h1>
<p>
  My name is Harry
</p>
```

페이지 별로 확인해보면 브라우저 tab 에 뜨는 문구가 달라진 것을 확인 할 수 있다. 

같은 방법으로 contact 페이지를 테스트를 통해 절차적으로 만들어 본다.

`git push`

`git push heroku master` 는 되지 않는다. 왜냐하면 현재 브랜치가 master 가 아니기 때문이다.

현재 master branch 가 아니기 때문에 master 브랜치로 이동 한 후 merge 를 해주어야한다.

`git checkout master`

`git merge static-pages` 

`git push`

`git push heroku master`

---

#### 2. test 를 예쁘게 & SETTING

- `test/test_helper.rb`

```ruby
ENV['RAILS_ENV'] ||= 'test'
require File.expand_path('../../config/environment', __FILE__)
require 'rails/test_help'

# 아래 2줄 추가
require 'minitest/reporters'
Minitest::Reporters.use!

...
    
end
```

- `bundle exec` : 정확한 버전으로 실행 하겠다.
- `bundle exec guard init`
- 주 강사의 guardfile 복붙해오기
- `guardfile`

```ruby
# Defines the matching rules for Guard.
guard :minitest, spring: "bin/rails test", all_on_start: false do
  watch(%r{^test/(.*)/?(.*)_test\.rb$})

  watch('test/test_helper.rb') { 'test' }

  watch('config/routes.rb')    { integration_tests }

  watch(%r{^app/models/(.*?)\.rb$}) do |matches|
    "test/models/#{matches[1]}_test.rb"
  end

  watch(%r{^app/controllers/(.*?)_controller\.rb$}) do |matches|
    resource_tests(matches[1])
  end

  watch(%r{^app/views/([^/]*?)/.*\.html\.erb$}) do |matches|
    ["test/controllers/#{matches[1]}_controller_test.rb"] +
        integration_tests(matches[1])
  end

  watch(%r{^app/helpers/(.*?)_helper\.rb$}) do |matches|
    integration_tests(matches[1])
  end

  watch('app/views/layouts/application.html.erb') do
    'test/integration/site_layout_test.rb'
  end

  watch('app/helpers/sessions_helper.rb') do
    integration_tests << 'test/helpers/sessions_helper_test.rb'
  end

  watch('app/controllers/sessions_controller.rb') do
    ['test/controllers/sessions_controller_test.rb',
     'test/integration/users_login_test.rb']
  end

  watch('app/controllers/account_activations_controller.rb') do
    'test/integration/users_signup_test.rb'
  end

  watch(%r{app/views/users/*}) do
    resource_tests('users') +
        ['test/integration/microposts_interface_test.rb']
  end
end

# Returns the integration tests corresponding to the given resource.
def integration_tests(resource = :all)
  if resource == :all
    Dir["test/integration/*"]
  else
    Dir["test/integration/#{resource}_*.rb"]
  end
end

# Returns the controller tests corresponding to the given resource.
def controller_test(resource)
  "test/controllers/#{resource}_controller_test.rb"
end

# Returns all tests for the given resource.
def resource_tests(resource)
  integration_tests(resource) << controller_test(resource)
end
# Live reload (Chrome Extension)
guard 'livereload' do
  extensions = {
    css: :css,
    scss: :css,
    sass: :css,
    js: :js,
    coffee: :js,
    html: :html,
    png: :png,
    gif: :gif,
    jpg: :jpg,
    jpeg: :jpeg,
    # less: :less, # uncomment if you want LESS stylesheets done in browser
  }

  rails_view_exts = %w(erb haml slim)

  # file types LiveReload may optimize refresh for
  compiled_exts = extensions.values.uniq
  watch(%r{public/.+\.(#{compiled_exts * '|'})})

  extensions.each do |ext, type|
    watch(%r{
          (?:app|vendor)
          (?:/assets/\w+/(?<path>[^.]+) # path+base without extension
           (?<ext>\.#{ext})) # matching extension (must be first encountered)
          (?:\.\w+|$) # other extensions
          }x) do |m|
      path = m[1]
      "/assets/#{path}.#{type}"
    end
  end

  # file needing a full reload of the page anyway
  watch(%r{app/views/.+\.(#{rails_view_exts * '|'})$})
  watch(%r{app/helpers/.+\.rb})
  watch(%r{config/locales/.+\.yml})
  # Rails Assets Pipeline
  watch(%r{(app|vendor)(/assets/\w+/(.+\.(css|js|html))).*}) { |m| "/assets/#{m[3]}" }
end
```

- `gemfile`

```ruby
# 개발 그룹에 추가
gem 'guard-livereload', '~> 2.5', require: false
```

- `.gitignore`

```ruby
# 마지막 행에 추가
/spring/*.pid
```

- `bundle`
- `guard` 명령어를 켜면 guard 가 펴지는데 파일을 바꾸면 자동으로 테스트 해주는 역할을 한다.
- 즉, 이제 터미널은 기본용/guard/서버용 `3개`가 켜져있게 된다.

---

