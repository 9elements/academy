# Controllers tend to do much, way too much

In our history we've seen many reasons why controllers got too big and difficult to maintain:

- Params Envy, or with other words form logic leaks into the controller
- Business logic that should have located in a model or service
- unneeded coupling to views (too many instance variables)

Let's shed some light on the different problems:

## Params Envy

```ruby
def index
  @products = Product.all
  @products = @products.where("name like ?", "%" + params[:name] + "%") if params[:name]
  @products = @products.where("price >= ?", params[:price_gt]) if params[:price_gt]
  @products = @products.where("price <= ?", params[:price_lt]) if params[:price_lt]
end
```

In this example the logic on how we select products is depending on http params. If you want to test this locic you'll have to wire up a controller test. The initial refactoring would be to move that logic into the model:

```ruby
class Product
  ...
  def self.search(params)
    products = Product.all
    products = products.where("name like ?", "%" + params[:name] + "%") if params[:name]
    products = products.where("price >= ?", params[:price_gt]) if params[:price_gt]
    products = products.where("price <= ?", params[:price_lt]) if params[:price_lt]
    products
  end
  ...
end

def index
  products = Products.search(params)
end
```

This model method is way easier to test - but we we're not happy. The search is still relying on an arbitrary structured params hash, instead well defined named parameters:

```ruby
class Product
  ...
  def self.search(name:, price_gt:, price_lt:)
    products = Product.all
    products = products.where("name like ?", "%" + name + "%") if name
    products = products.where("price >= ?", price_gt) if price_gt
    products = products.where("price <= ?", price_lt) if price_lt
    products
  end
  ...
end

def index
  products = Products.search(name: params[:name], price_gt: params[:price_gt], price_lt: params[:price_lt])
end
```

But we're still not happy - imagine you have 10 parameters or more to filter or even complex logic between them:

```ruby
class ProductSearchForm < OffTheRecord::Base
  attribute :name
  attribute :price_gt, type: Integer
  attribute :price_lt, type: Integer
  ...
end

class Product
  ...
  def self.search(product_search_form)
    products = Product.all
    products = products.where("name like ?", "%" + product_search_form.name + "%") if product_search_form.name.present?
    products = products.where("price >= ?", product_search_form.price_gt) if product_search_form.price_gt
    products = products.where("price <= ?", product_search_form.price_lt) if product_search_form.price_lt
    ...
    products
  end
  ...
end

def index
  product_search_form = ProductSearchForm.from_params(params)
  products = Products.search(product_search_form)
end
```

With the last we've modeled an explicit interface between the params and an form object that can validated and which can additional logic how to process the params. This form object will be passed to the search method in the model. Note that the domain logic in the model building the query does not care it's parameter being a form model - it just uses it's attributes. It does not need to change when the form model needs to be changed to satisfy of the frontend - think about we rename the params and add a validation:

```ruby
class ProductSearchForm < OffTheRecord::Base
  attribute :name
  attribute :price_minimum, type: Integer
  attribute :price_maximum, type: Integer

  alias_method :price_gt, :price_minimum
  alias_method :price_lt, :price_maximum

  validates :name, presence: true
  ...
end

class Product
  ...
  def self.search(product_search_form)
    products = Product.all
    products = products.where("name like ?", "%" + product_search_form.name + "%") if product_search_form.name.present?
    products = products.where("price >= ?", product_search_form.price_gt) if product_search_form.price_gt
    products = products.where("price <= ?", product_search_form.price_lt) if product_search_form.price_lt
    ...
    products
  end
  ...
end

def index
  product_search_form = ProductSearchForm.from_params(params)
  products = Products.search(product_search_form)
end
```
Now the search input is decoupled from the frontend.

Credit: http://railscasts.com/episodes/212-refactoring-dynamic-delegator
