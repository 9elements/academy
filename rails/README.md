# Rails Best Practices

Work in progress

## Motivation

While Rails has served well for smaller web projects the architecture comes to its limits when applications are growing. We identified the following problems while developing a large scale Rails application:

- Helpers grew beyond manageability; no real vertical separability; (Business) Logic in views
- Too much logic in models
- Controllers tend to do much, way too much. Same applies to rake tasks!
- Frontend editing concerns creep into models
- (Over)use of Callbacks, especially in models
- Too big helpers; Too many helpers
- Application Logic spreads over the Rails app

You know the game. When you start developing your web application you might have some logic that won’t fit in the model since it heavily deals with the presentation (HTML & CSS). The classic Rails way is to put this logic in a helper method (that can be directly used within a view). In our experience this is a code smell since these helpers will have a lot of concerns - they also will grow beyond manageability and testability. They will also spread a lot of application logic over a lot of places.

Example code (TODO migrate):
https://gitlab.9elements.com/employour/meinpraktikum/blob/6c58133fabda326a978b6d0bac5203a168a47eee/app/helpers/companies_helper.rb

# Solution

In ausbildung.de we introduced a widget system. Widgets will be passed all the desired data for rendering. They also have instance methods that encapsulate the logic that is needed in the views. This reduces logic in the templates to a bare minimum.


Example code (TODO come up with a better one):
https://gitlab.9elements.com/employour/ausbildung-3/blob/develop/app/widgets/corporation_tab_bar/widget.rb

## Too much logic in models

When you first start developing with Rails 99% of your models will be models that have to be persisted and ActiveRecord will take care of that. When your application grows you will have business logic that will be difficult to assign to a certain model. Imagine you have a customer and the customer will have many invoices and the invoices have many invoice items (@TODO: UML). Now you want to implement a logic for calculating the VAT - where does it go?

Will it be implemented in the invoice? Since the VAT is an aspect of the invoice?
Will it be implemented in the invoice item? Since there can be different VAT rates for different items?
Will it be implemented in the user? Since it depends on the country the user lives in (at least digital services)?

We have seen code in the past that puts the logic in the model that was last opened in the editor - which is bad.

```ruby
class InvoiceItem < Application
  belongs_to :invoice
  belongs_to :product

  VAT_RATE = 19

public

  def vat
    (vat_rate.to_f / 100 * amount).round
  end

private

  def within_eu?
    [
      'AT', 'BE', 'BG', 'CY', 'CZ', 'DE', 'DK', 'EE', 'GR', 'ES', 'FI', 'FR', 'GB', 'HU', 'IE', 'IT', 'LT', 'LU', 'LV', 'MT', 'NL', 'PL', 'PT', 'RO', 'SE', 'SI', 'SK'
    ].include?(invoice.user.country)
  end

  def vat_rate
    within_eu? ? VAT_RATE : 0
  end
end
```

This code is suboptimal in many ways - but honestly we’ve seen many times. So what is wrong with this code:

The developer has many the responsibility for calculating the VAT as a concern of the InvoiceItem (probably because he needed the vat value right there for the first time).
The VAT calculation depends on the invoice user’s country, but this dependency is now implicit and deeply hidden.
It also violates the law of demeter: it needlessly depends on the invoice’ user.
(not shown above) A part of the VAT calculation code lives in the invoice model (simply enumerating) TODO maybe just add the code

### How could we do it better?

What if you put this code in neither of these models. What if you create a new model (which we better call a service) that doesn’t depend on ActiveRecord that calculates the tax of an invoice item.

```ruby
# single fucking responsibility
class InvoiceItemVatCalculator
 VAT_RATE = 19

 def initialize(invoice_item, country)
   @invoice_item = invoice_item

   # fixing law of demeter
   @country = country
 end

public

 # public interface called by the view
 def vat
   (vat_rate.to_f / 100 * @invoice_item.amount).round
 end

 def within_eu?
   [
     'AT', 'BE', 'BG', 'CY', 'CZ', 'DE', 'DK', 'EE', 'GR', 'ES', 'FI', 'FR', 'GB', 'HU', 'IE', 'IT', 'LT', 'LU', 'LV', 'MT', 'NL', 'PL', 'PT', 'RO', 'SE', 'SI', 'SK'
   ].include?(country)
 end

private

 def vat_rate
   within_eu? ? VAT_RATE : 0
 end
end
```

Additionally, write a second service that knows about how an invoice and a country are connected and uses the above service as a dependency for calculating the VAT numbers needed for an invoice.

The InvoiceItemVatCalculator service is not tied to our persistence domain any more! Imagine writing tests for it. You do not need to touch ActiveRecord in any way and can pass simple Ruby objects into it instead.

## Controllers tend to do much, way too much

In our history we've seen many reasons why controllers got too big and difficult to maintain:

- params envy
- form logic leaks into the controller
- business logic that should have in a model or service
- unneeded coupling to views (too many instance variables)

Let's shed some light on the different problems:

### Params Envy

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
