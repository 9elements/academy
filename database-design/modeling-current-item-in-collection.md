# Modeling current item in a collection

Imagine you have a `Todos` table and it's managing todos (how thrilling). Now you want to implement the feature that you can display at the task that you currently working on. You have several options to model that in the database:

* [The naive way](#The-naive-way)
* [The smartass way](#the-smartass-way)
* [The best practice (in 98% of all cases this is what you want)](#the-best-practice)
* [The flexible way](#the-flexible-way)

## The naive way

Intuitively you might want to add a `current boolean` field to your `Todo` model. The mechanic to select a new current todo would be to:

1. Set all current flags to of all user related todos to `false`.
2. Set `current` flag to `true`.
3. Reload `user.todos` to invalidate ActiveRecords identity cache.

```ruby
class Todo < ApplicationRecord
  belongs_to :user

  default_scope { order(updated_at: :desc) }

  def select_as_current_todo
    user.todos.update_all("current = 'false'")
    self.current = true
    self.save

    user.todos.reload
    self
  end
end
```

### Pros

* Very simple implementation

### Cons

* No application mechanism protects you from creating invalid data (e.g. two items with `current` set to `true`).
* Rails rookies often forget to reload the collection - this is error prone.
* Selecting a todo means N+1 operations on the database table.
* You'll be not very flexible for future features (e.g. multiple current todos).

## The smartass way

Let's say you don't care much about the order of your todos then you could declare the current todo as the latest todo that you've created / touched.

```ruby
class Todo < ApplicationRecord
  belongs_to :user

  default_scope { order(updated_at: :desc) }
end
```

if you want to select another todo you simply touch it in the controller:

```ruby
class TodosController < ApplicationController
  def select
    todo = Todo.find(params[:id])
    todo.touch(:updated_at)
    ...
  end
end
```

### Pros

* Very simple implementation

### Cons

* Sometimes wierd behavior, e.g. when a rake task also changes updated_at.
* If you want to sort by a second field you're screwed.
* You'll be not very flexible for future features (e.g. multiple current todos).

## The best practice

The best practice is to model the current todo in the `User`. You would have to add a `current_todo_id` to the `User` model and create convenience functions to access them:

```ruby
class User < ApplicationRecord
  has_many :todos, dependent: :destroy

  def current_todo
    Todo.find(current_todo_id)
  end

  def current_todo=(todo)
    current_todo_id = todo.id
  end
end
```

if you want to select another todo you'll use the convenience functions:

```ruby
class TodosController < ApplicationController
  def select
    todo = Todo.find(params[:id])
    current_user.current_todo = todo
    current_user.save
    ...
  end
end
```

### Pros

* Very simple implementation.
* Very fast (just one database update).
* You barely able to create inconsistent state.

### Cons

* You'll be not very flexible for future features (e.g. multiple current todos).

## The flexible way

You could create a join model (@TODO: use through) to implement the feature:

```ruby
class User < ApplicationRecord
  has_many :todos, dependent: :destroy
  has_many :current_todos, dependent: :destroy
end

class Todos < ApplicationRecord
  belongs_to :user
  has_many :current_todos, dependent: :destroy
end

class CurrentTodo < ApplicationRecord
  belongs_to :user
  belongs_to :todo
end
```

### Pros

* You'll be open for future features (e.g. multiple current todos).
* You'll be able to even attach more data to a current todo.

### Cons

* You'll need a complex N..M relation.
