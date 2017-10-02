# Modeling current item in a collection

Imagine you have a `Todos` table and it's managing todos (how thrilling). Now you want to implement the feature that you can display at the task that you currently working on. You have several options to model that in the database:

## The smart way

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

* Sometimes wierd behavior, e.g. when a rake task also changes updated_at
