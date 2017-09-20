### Business logic that should have located in a model or service

Imagine this controller action. The user wants to buy a product that costs $100. We're leveraging the 3rd party service stripe to process a payment. Since stripe is also managing customers we will need to assign our user a stripe customer id (we don't want to create another customer in stripe for a repurchase). In this case the logic for retrieving a stripe customer id leaked into the controller:

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

The naive refactoring would be to move that logic in the User model:

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

  def create_charge(stripe_customer_id)
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
    stripe_customer_id = stripe_service.stripe_customer_id
    @stripe_charge = stripe_service.create_charge(stripe_customer_id)
  end
end
```

Now the controller operates on a single level of abstractions instead of jumping around between high level and low level code.