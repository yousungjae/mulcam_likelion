# 3week_3

## 06.20(수)

### crud 복습

- `rails g model Team`
- `20180620011019_create_teams.rb`

```ruby
class CreateTeams < ActiveRecord::Migration[5.1]
  def change
    create_table :teams do |t|
      
      t.string :name
      t.string :member_1
      t.string :member_2
      t.string :member_3
      t.text :description

      t.timestamps
    end
  end
end
```

- `rails db:migrate`
- `rails g controller Teams new show index edit `

---

#### url 과 path 의 차이

- redirect_to 는 정확한 주소를 명시해줘야하기 때문에 url 을 쓴다.
  (path 를 사용해도 동작은 한다.)

|                   url                    |    path    |
| :--------------------------------------: | :--------: |
| https://rails-chris.c9users.io/teams/new | /teams/new |

#### Semantic markup (의미론적 마크업)

|        태그         |                  사용하는 이유                  |
| :-----------------: | :---------------------------------------------: |
|      `<b></b>`      |               단순히 글자를 굵게                |
| `<strong></strong>` |  **중요하거나 강조의 의미를 위해** 글자를 굵게  |
|      `<i></i>`      |              단순히 글자를 기울게               |
|     `<em></em>`     | **중요하거나 강조의 의미를 위해** 글자를 기울게 |

>  현대 웹은 의미론적인 태그들을 사용하는 것을 지향한다.

----

- `routes.rb`

```ruby
Rails.application.routes.draw do
  resources :teams
end
```

- `teams_conroller.rb`

```ruby
class TeamsController < ApplicationController
  def new
    @team = Team.new
  end
  
  def create
    @team = Team.new(params.require(:team).permit(:name, :member_1, :member_2, :member_3, :description))
    @team.save
    redirect_to team_url(@team)
    # 더 생략하면, redirect_to @team
  end

  def show
    @team = Team.find params[:id]
  end

  def index
    @teams = Team.all
  end

  def edit
    @team = Team.find params[:id]
  end
  
  def update
    @team = Team.find params[:id]
    @team.update(params.require(:team).permit(:name, :member_1, :member_2, :member_3, :description))
    redirect_to team_url(@team)
  end
  
  def destroy
    @team = Team.find params[:id]
    @team.destroy
    redirect_to teams_url
  end
end
```

- `index.html.erb`

```ruby
<h1>Teams#index</h1>
<h2><%= link_to 'Make New Team', new_team_path %></h2>
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Member 1</th>
      <th>Member 2</th>
      <th>Member 3</th>
    </tr>
  </thead>
  <tbody>
    <% @teams.each do |team| %>
      <tr>
        <td><%= link_to team.name, team_path(team) %></td>
        <td><%= team.member_1 %></td>
        <td><%= team.member_2 %></td>
        <td><%= team.member_3 %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

- `show.html.erb`

```ruby
<h1>Teams#show</h1>
<h2><%= link_to 'Back to index', teams_path %></h2>
<p>
  <strong>Name: </strong>
  <%= @team.name %>
</p>
<p>
  <strong>Member 1: </strong>
  <%= @team.member_1 %>
</p>
<p>
  <strong>Member 2: </strong>
  <%= @team.member_2 %>
</p>
<p>
  <strong>Member 3: </strong>
  <%= @team.member_3 %>
</p>
<p>
  <strong>Des: </strong>
  <%= @team.description %>
</p>
<%= link_to 'Edit', edit_team_path(@team) %> | 
<%= link_to 'Delete', team_path(@team), method: :DELETE, data: { confirm: 'Are you sure?' }  %>

<%#= link_to 'Delete', @team, method: :DELETE, data: { confirm: 'Are you sure?' }  %>
```

- `new.html.erb`

```ruby
<h1>Teams#new</h1>
<h2><%= link_to 'Back to Index', teams_path %></h2>
<%= form_for @team do |f| %>
  <div>
    <%= f.label :name, '팀명' %>
    <%= f.text_field :name %>
  </div>
  <div>
    <%= f.label :member_1 %>
    <%= f.text_field :member_1 %>
  </div>
  <div>
    <%= f.label :member_2 %>
    <%= f.text_field :member_2 %>
  </div>
  <div>
    <%= f.label :member_3 %>
    <%= f.text_field :member_3 %>
  </div>
  <div>
    <%= f.label :description %>
    <%= f.text_area :description, cols: 50, rows: 20 %>
  </div>
  <div>
    <%= f.submit %>
  </div>
<% end %>
```

- `edit.html.erb`

```ruby
<h1>Teams#edit</h1>
<h2><%= link_to 'Back to Show', team_path(@team) %></h2>
<%= form_for @team do |f| %>
  <div>
    <%= f.label :name, '팀명' %>
    <%= f.text_field :name %>
  </div>
  <div>
    <%= f.label :member_1 %>
    <%= f.text_field :member_1 %>
  </div>
  <div>
    <%= f.label :member_2 %>
    <%= f.text_field :member_2 %>
  </div>
  <div>
    <%= f.label :member_3 %>
    <%= f.text_field :member_3 %>
  </div>
  <div>
    <%= f.label :description %>
    <%= f.text_area :description, cols: 50, rows: 20 %>
  </div>
  <div>
    <%= f.submit %>
  </div>
<% end %>
```

---

### Refactoring

- `teams_controller.rb`

```ruby
class TeamsController < ApplicationController
  before_action :set_team, only: [:show, :edit, :update, :destroy]
  def new
    @team = Team.new
  end
  
  def create
    @team = Team.new(team_params)
    @team.save
    redirect_to team_url(@team)
    # redirect_to @team
  end

  def show
  end

  def index
    @teams = Team.all
  end

  def edit
  end
  
  def update
    @team.update(team_params)
    redirect_to team_url(@team)
  end
  
  def destroy
    @team.destroy
    redirect_to teams_url
  end
  
  private
  def set_team
    @team = Team.find params[:id]
  end
  def team_params
    params.require(:team).permit(:name, :member_1, :member_2, :member_3, :description)
  end
end
```

- `_form.html.erb`

```erb
<%= form_for team do |f| %>
  <div>
    <%= f.label :name, '팀명' %>
    <%= f.text_field :name %>
  </div>
  <div>
    <%= f.label :member_1 %>
    <%= f.text_field :member_1 %>
  </div>
  <div>
    <%= f.label :member_2 %>
    <%= f.text_field :member_2 %>
  </div>
  <div>
    <%= f.label :member_3 %>
    <%= f.text_field :member_3 %>
  </div>
  <div>
    <%= f.label :description %>
    <%= f.text_area :description, cols: 50, rows: 20 %>
  </div>
  <div>
    <%= f.submit %>
  </div>
<% end %>
```

- `new.html.erb`

```erb
<h1>Teams#new</h1>
<h2><%= link_to 'Back to Index', teams_path %></h2>
<%= render 'form', team: @team %>
```

- `edit.html.erb`

```erb
<h1>Teams#edit</h1>
<h2><%= link_to 'Back to Show', team_path(@team) %></h2>
<%= render 'form', team: @team %>
```

---

#### Deploy

`$ git add .`

`$ git commit -m ''`

`$ git push`

`$ git push heroku master`

`$ heroku run rails db:migrate` (새로운 모델이 생겼기 때문에 heroku 에서도 migrate 해주어야 한다.)

