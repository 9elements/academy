# Business logic that should have located in a model or service

Imagine this controller action (it is somewhat simplified - but you'll get the point). The user wants to buy a product that costs $100 and we're leveraging the 3rd party service Stripe to process the payment. The Stripe service is also internally managing customers and we would like to have a mapping between these two (we don't want to create another customer in Stripe for a repurchase - we want to assign multiple purchases to the same customer). To implement this behaviour we have to store the `stripe_customer_id` in the `User` object when we create the Stripe customer so that we can reference it later.

```ruby
class PaymentController < ApplicationController
  def create
    @user = current_user

    if @user.stripe_customer_id.blank?
      customer = Stripe::Customer.create(
        description: @user.full_name,
        email: @user.email,
        source: @user.stripe_card_token,
        metadata: { 'user_id' => @user.id, 'env' => Rails.env.to_s }
      )

      @user.stripe_customer_id = customer.id
      @user.save
    end

    # do the billing
    @stripe_charge = Stripe::Charge.create(
      amount: 100,
      currency: "usd",
      customer: @user.stripe_customer_id,
      metadata: { 'charge_token' => @user.stripe_card_token, 'env' => Rails.env.to_s }
    )
  end
end
```

There are two things that are wrong with this code:

- The logic of referencing the stripe customer id with the user has _leaked_ into the controller action
- There is some _high level_ and some _low level_ stuff that we're doing in the controller action

Let's deal with the first problem first. The naive refactoring would be to move that logic in the User model:

```ruby
class User < ApplicationRecord

  def ensure_stripe_customer_id
    return self unless stripe_customer_id.blank?

    customer = Stripe::Customer.create(
      description: full_name,
      email: email,
      source: stripe_card_token,
      metadata: { 'user_id' => id, 'env' => Rails.env.to_s }
    )

    update(stripe_customer_id: customer.id)
    self
  end
end
```

Is that good place for this piece of code? No it's not - when you want to replace Stripe with a different service you would have to change the User model to do so. A better way would be to create a service that solely deals with Stripe:

```ruby
class StripeService
  def initialize(user)
    @user = user
  end

  def stripe_customer_id
    return @user.stripe_customer_id if @user.stripe_customer_id.present?

    customer = Stripe::Customer.create(
      description: @user.full_name,
      email: @user.email,
      source: @user.stripe_card_token,
      metadata: { 'user_id' => @user.id, 'env' => Rails.env.to_s }
    )

    @user.update(stripe_customer_id: customer.id)
    @user.stripe_customer_id
  end
end
```

Now our controller looks like this

```ruby
class PaymentController < ApplicationController
  def create
    @user = current_user
    stripe_customer_id = StripeService.new(@user).stripe_customer_id

    # do the billing
    @stripe_charge = Stripe::Charge.create(
      amount: 100,
      currency: "usd",
      customer: stripe_customer_id,
      metadata: { 'charge_token' => @user.stripe_card_token, 'env' => Rails.env.to_s }
    )
  end
end
```

What is win of that refactoring:

- The dependencies of the service is explicit
- The controller action becomes much easier to read

Now we also extract the charging part into the stripe service:

```ruby
class StripeService
  def initialize(user)
    @user = user
  end

  def stripe_customer_id
    return @user.stripe_customer_id if @user.stripe_customer_id.present?

    customer = Stripe::Customer.create(
      description: @user.full_name,
      email: @user.email,
      source: @user.stripe_card_token,
      metadata: { 'user_id' => @user.id, 'env' => Rails.env.to_s }
    )

    @user.update(stripe_customer_id: customer.id)
    @user.stripe_customer_id
  end

  def create_charge
    stripe_charge = Stripe::Charge.create(
      amount: 100,
      currency: "usd",
      customer: stripe_customer_id,
      metadata: { 'charge_token' => @user.stripe_card_token, 'env' => Rails.env.to_s }
    )
  end
end

class PaymentController < ApplicationController
  def create
    @user = current_user
    stripe_service = StripeService.new(@user)
    @stripe_charge = stripe_service.create_charge
  end
end
```

Now the controller operates on a single level of abstractions instead of jumping around between high level and low level code.

## Further Refactorings

The `StripeService` has two concerns, the first one is to create a Stripe customer and the second one is to create a charge. This could be refactored into a `StripeCreateUserService` and `StripeCreateCharge` service.

Also the `@stripe_charge` is passed into the view - so the view needs to know how to deal with Stripe responses. If you want to switch to a different payment processor you'll need to refactor the view too. To fix that we could introduce a `PaymentService` that uses the `StripeCreateUserService` and the `StripeCreateCharge` to communicate with Stripe. The result should be abstracted in a [ValueObject](https://medium.com/@franzejr/value-object-in-ruby-89b5e3b6b5f9) and this should be passed to the view. With that refactoring you could swap out the payment processor without touching model, controller or view.
