# Modeling a friendship relation

The concrete term is self referential associations. The idea is to create a join model that takes care of the N..M association. The models look like this:

```ruby
class User < ApplicationRecord
  has_many :friendships
  has_many :friends, through: :friendships
end
```

```ruby
class Friendship < ApplicationRecord
  belongs_to :user
  belongs_to :friend, class_name: 'User'
end
```

The controller roughly looks like this.

```ruby
class FriendshipsController < ApplicationController
  def create
    @friendship = current_user.friendships.build(:friend_id => params[:friend_id])
    if @friendship.save
      flash[:notice] = "Added friend."
      redirect_to root_url
    else
      flash[:notice] = "Unable to add friend."
      redirect_to root_url
    end
  end
```

This code snippets were extracted by Ryan Bates excellent [Railscasts episode](http://railscasts.com/episodes/163-self-referential-association?view=asciicast).

