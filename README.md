### `User` **becomes** `SuperUser` OR  `SuperUser` **becomes** `User`

There are many cases in the rails project when we have STI models, like GuestUser, SuperUser and PremiumUser(may be many more)

```ruby
class User < ActiveRecord::Base
end

class GuestUser < ActiveRecord::Base
end

class SuperUser < ActiveRecord::Base
end

class PremiumUser < ActiveRecord::Base
end
```


Now people create controllers(super_users, premium_users, guest_users) and view files for each type of user for implementing CRUD functionality,
e.g.


Now will talk about Create feature. only

### Controllers
in app/controllers/super_users_controller.rb
```ruby
  class SuperUsersController < ApplicationController
    def new
      @user = SuperUser.new
    end

    # params[:super_user] # => { :username=>"kuldeepaggarwal", :email=>"kd.engineer@yahoo.co.in", :mobile=> '+91 9015182145' }
    def create
      @user = SuperUser.new(resource_params)
      if @user.save
        # success code
      else
        render :new
      end
    end

    private
      def resource_params
         params[:super_user].permit(:username, :mobile, :email)
      end
  end
```
in app/controllers/guest_users_controller.rb
```ruby
  class GuestUsersController < ApplicationController
    def new
      @user = GuestUser.new
    end

    # params[:guest_user] # => { :username=>"kuldeepaggarwal", :email=>"kd.engineer@yahoo.co.in", :mobile=> '+91 9015182145' }
    def create
      @user = GuestUser.new(resource_params)
      if @user.save
        # success code
      else
        render :new
      end
    end

    private
      def resource_params
         params[:guest_user].permit(:username, :mobile, :email)
      end
  end
```

### Views
in app/views/super_users/new.html.erb
```erb
<%= form_for @user do |f| %>
  <%= f.text_field :username %>
  <%= f.email_field :email %>
  <%= f.text_field :mobile %>
<% end %>
```
in app/views/guest_users/new.html.erb
```erb
<%= form_for @user do |f| %>
  <%= f.text_field :username %>
  <%= f.email_field :email %>
  <%= f.text_field :mobile %>
<% end %>
```

But **What if**, we need a single form to allow user to create different type of users in a single form.....

Rails has given us a feature **becomes** (`SuperUser => User`)

Instead of creating 3 different but similar controllers create a UsersConroller.

in app/controllers/users_controller.rb
```ruby
class UsersController < ApplicationController
  before_action :filter_user_type

  def new
    @user = User.new
  end

  # params[:user] # => { :username=>"kuldeepaggarwal", :email=>"kd.engineer@yahoo.co.in", :mobile=> '+91 9015182145', :type=>"SuperUser" }
  def create
    @user = params[:user][:type].constantize.new(resource_params)
    if @user.save
      # success code
    else
      @user = @user.becomes(User) # This will transform SuperUser object to User object, but type will be same as `SuperUser`.
      render :new
    end
  end

  private
    # We just need to ensure that a valid user type is present, i.e. no body has changed the value from form.
    def filter_user_type
      unless %w(Super GuestUser PremiumUser).include?(resource_params[:type])
        flash[:error] = "Invalid User type"
        redirect_to new_user_path
      end
    end
    def resource_params
       params[:user].permit(:username, :mobile, :email, :type)
    end
end
```
same with views, intstead of creating 3 different views create a single form with select option, let user would decide which user he wants to create.

in app/views/users/new.html.erb
```
<%= form_for @user do |f| %>
  <%= f.select :type, [['Super User', 'SuperUser'], ['Guest User', 'GuestUser'], ['Premium User', 'PremiumUser']] %>
  <%= f.text_field :username %>
  <%= f.email_field :email %>
  <%= f.text_field :mobile %>
<% end %>
```
And its done, you need not make new controller or view if you ever have any new type of user, just add in the select options and to before_action.

p.s. Code can be optimised. But its okay to duplicate the code for demonstartion. :smile:


Kuldeep Aggarwal<br>
+91 9015182145<br>
[GitHub](https://github.com/kuldeepaggarwal) |  [Twitter](http://twitter.com/kdengineer)
