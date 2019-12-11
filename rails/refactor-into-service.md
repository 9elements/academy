In a project we found a lot of code duplication spread over several controllers. The method under refactoring is dealing with the creation of a Leady entity and an underlying User.

The logic of the method is the following:

  - Initialilly there is a spam check
  - Then we musst look if there is already a user in the database
  - If not a new User will be created
  - Then the lead will be created
  - Some emails will be send
  - and if we're runnin in production the User will be send to the CRM API

Here is the corresponding controller code:

```ruby
def create
  unless user_params[:content].present? || user_params[:phone].start_with?("+74")
    @user = User.new(user_params)
    possible_existing_user = User.find_by(email: @user.email)

    if possible_existing_user.present?
      possible_existing_user.update(requested_from_page: @user.requested_from_page)
      @lead = Lead.create(user_id: possible_existing_user.id, requested_from_page: @user.requested_from_page)
      auto_login(possible_existing_user)

      ContactMailer.with(user: possible_existing_user).contact_mail.deliver_now
      ContactMailer.with(user: possible_existing_user).contact_mail_to_customer.deliver_now

      redirect_to edit_lead_path(possible_existing_user)

    else
      @user.save
      @user.update(
        crypted_password: SecureRandom.base64(20)
      )

      @lead = Lead.new(
        user_id: @user.id,
        requested_from_page: @user.requested_from_page,
        request_reason: "other"
      )

      @lead.save
      auto_login(@user)

      ContactMailer.with(user: @user).contact_mail.deliver_now
      ContactMailer.with(user: @user).contact_mail_to_customer.deliver_now

      if Rails.env.production? || Rails.env.staging?
        begin

          # TODO: Think about moving this to background job
          CRMService.new.create(
            firstname: mail_info[:first_name],
            lastname: mail_info[:last_name],
            email: mail_info[:email],
            phone: mail_info[:phone],
            requested_from_page: mail_info[:requested_from_page]
          )
        rescue
          # TODO: Error handling

        end
      end

      # flash[:success] = 'Ihre Anfrage wurde gesendet.'
      redirect_to edit_lead_path(@lead.access_token)
    end
  end
end
```

This code violates many clean code rules:

1. Code duplication (the exact same code has been found in three controllers)
2. For a controller method it is way too long
3. Heavy mixing of concerns
4. Heavy mixing of low level and high level code
5. Edge cases may lead to ignored code

Refactoring these ~50 lines of code was quite a journey and we would like to guide you through the steps (and mistakes) thas has been made.

## Fixing 1. and 2.

The first step was to refactor the controller code into a service so that it can be called from multiple controllers:

```ruby
class UserRegistrationService
  def create_new_lead_or_user(user_params)
    unless user_params[:content].present? || user_params[:phone].start_with?("+74")
      @user = User.new(user_params)
      possible_existing_user = User.find_by(email: @user.email)

      if possible_existing_user.present?
        possible_existing_user.update(requested_from_page: @user.requested_from_page)
        @lead = Lead.create(user_id: possible_existing_user.id, requested_from_page: @user.requested_from_page)
        yield(possible_existing_user)

        ContactMailer.with(user: possible_existing_user).contact_mail.deliver_now
        ContactMailer.with(user: possible_existing_user).contact_mail_to_customer.deliver_now

        possible_existing_user

      else
        @user.save
        @user.update(
          crypted_password: SecureRandom.base64(20)
        )

        @lead = Lead.new(
          user_id: @user.id,
          requested_from_page: @user.requested_from_page,
          request_reason: "other"
        )

        @lead.save
        yield(@user)

        ContactMailer.with(user: @user).contact_mail.deliver_now
        ContactMailer.with(user: @user).contact_mail_to_customer.deliver_now

        if Rails.env.production? || Rails.env.staging?
          begin

            # TODO: Think about moving this to background job
            CRMService.new.create(
              firstname: mail_info[:first_name],
              lastname: mail_info[:last_name],
              email: mail_info[:email],
              phone: mail_info[:phone],
              requested_from_page: mail_info[:requested_from_page]
            )
          rescue
            # TODO: Error handling

          end
        end

        # flash[:success] = 'Ihre Anfrage wurde gesendet.'
        @lead.access_token
      end
    end
  end
end
```

This was the most minimal refactoring to fix the code duplication issue. Noticing the yield in the middle of the method? This was due to the `auto_login` method is only available in the controller context so the developer found a creative way to expose the user to calling method. In the controller the code is used like this:

```ruby
def create
  user_registration_service = UserRegistrationService.new
  lead_or_possible_existing_user = user_registration_service.create_new_lead_or_user(user_params) do |user|
    auto_login(user)
  end
  redirect_to edit_lead_path(lead_or_possible_existing_user)
end
```

## Fixing 3. and 4.

To avoid one lengthy method and to seperate the concerns (detecting spam, sending emails)

```ruby
class UserRegistrationService
  def initialize(user_params)
    @user_params = user_params
    @user = User.new(user_params)
  end

  def create_new_lead_or_user
    unless check_if_spam?
      possible_existing_user = User.find_by(email: @user.email)
      random_distributor = Employee.includes(:leads).where(role: :distributor).sample

      if possible_existing_user.present?
        possible_existing_user.update(
          requested_from_page: @user.requested_from_page
        )

        create_new_lead(possible_existing_user, random_distributor)

        yield(possible_existing_user)

        send_contact_mail(possible_existing_user)

        possible_existing_user

      else
        @user.save
        @user.update(
          crypted_password: SecureRandom.base64(20)
        )

        create_new_lead(@user, random_distributor)

        yield(@user)

        send_contact_mail(@user)

        if Rails.env.production? || Rails.env.staging?
          begin

            # TODO: Think about moving this to background job
            CRMService.new.create(
              firstname: mail_info[:first_name],
              lastname: mail_info[:last_name],
              email: mail_info[:email],
              phone: mail_info[:phone],
              requested_from_page: mail_info[:requested_from_page]
            )
          rescue
            # TODO: Error handling

          end
        end
      end
    end
  end

  def create_new_lead(user, random_distributor)
    @lead = Lead.create(
      user_id: user.id,
      requested_from_page: user.requested_from_page,
      employee_id: random_distributor.id
    )
  end

  def send_contact_mail(user)
    ContactMailer.with(user: user).contact_mail.deliver_now
    ContactMailer.with(user: user).contact_mail_to_customer.deliver_now
  end

  def check_if_spam?
    @user_params[:content].present? || @user_params[:phone].start_with?("+74")
  end

end
```

but there was still that ugly (yet creative yield) and things could even be improved further. Let's annotate the code with concern and how high level or low level the code is.

```ruby
class UserRegistrationService

  def initialize(user_params)
    @user_params = user_params
    @user = User.new(user_params)
  end

  def create_new_lead_or_user
    unless check_if_spam? # user / high level
      possible_existing_user = User.find_by(email: @user.email) # user / low level
      random_distributor = Employee.includes(:leads).where(role: :distributor).sample # lead / low level

      if possible_existing_user.present? # user / low level
        possible_existing_user.update( # user / low level
          requested_from_page: @user.requested_from_page
        )

        create_new_lead(possible_existing_user, random_distributor) # lead / high level

        yield(possible_existing_user) # usre / high level

        send_contact_mail(possible_existing_user) # user / high level

        possible_existing_user

      else
        @user.save # user / low level
        @user.update(
          crypted_password: SecureRandom.base64(20)
        ) # user / low level

        create_new_lead(@user, random_distributor) # lead / high level

        yield(@user) # user / high level

        send_contact_mail(@user) # user / high level

        if Rails.env.production? || Rails.env.staging? # user / low level
          begin

            # TODO: Think about moving this to background job
            CRMService.new.create(
              firstname: mail_info[:first_name],
              lastname: mail_info[:last_name],
              email: mail_info[:email],
              phone: mail_info[:phone],
              requested_from_page: mail_info[:requested_from_page]
            ) # user / high level
          rescue
            # TODO: Error handling

          end
        end
      end
    end
  end

  def create_new_lead(user, random_distributor)
    @lead = Lead.create(
      user_id: user.id,
      requested_from_page: user.requested_from_page,
      employee_id: random_distributor.id
    )
  end

  def send_contact_mail(user)
    ContactMailer.with(user: user).contact_mail.deliver_now
    ContactMailer.with(user: user).contact_mail_to_customer.deliver_now
  end

  def check_if_spam?
    @user_params[:content].present? || @user_params[:phone].start_with?("+74")
  end

end
```

Now let's split up `create_new_lead_or_user` method into two methods.

```ruby
class UserRegistrationService

  def find_or_create_user(user_params)
    user = User.where(email: user_params[:email]).first
    if user.present?
      user.update(requested_from_page: user_params[:requested_from_page])
    else
      user = User.create(user_params.merge(crypted_password: SecureRandom.base64(20)))

      send_user_to_crm if Rails.env.production? || Rails.env.staging?
    end
    return user
  end

  def create_new_lead(user, random_distributor = Employee.includes(:leads).where(role: :distributor).sample)
    lead = Lead.create(
      user_id: user.id,
      requested_from_page: user.requested_from_page,
      employee_id: random_distributor.id
    )
    send_contact_mail(user)
    return lead
  end

  def send_contact_mail(user)
    ContactMailer.with(user: user).contact_mail.deliver_now
    ContactMailer.with(user: user).contact_mail_to_customer.deliver_now
  end

  def check_if_spam?(user_params)
    user_params[:content].present? || user_params[:phone].start_with?("+74")
  end

  private

  def send_user_to_crm(user_params)
    begin

      # TODO: Think about moving this to background job
      CRMService.new.create(
        firstname: user_params[:first_name],
        lastname: user_params[:last_name],
        email: user_params[:email],
        phone: user_params[:phone],
        requested_from_page: user_params[:requested_from_page]
      )
    rescue
      # TODO: Error handling
    end
  end

end
```

The controller now looks like this:

```ruby
def create
  user_registration_service = UserRegistrationService.new()
  unless user_registration_service.check_if_spam?(user_params)
    user = user_registration_service.find_or_create_user(user_params)
    auto_login(user)
    lead = user_registration_service.create_new_lead(user)
    redirect_to edit_lead_path(lead)
  end
end
```

Looks better? Decide for yourself.

# Fixing 5.

Rescueing an exception and then doing nothing is a huge antipattern - think of an error like an important message to the developer and rescuing the exception without handling the error code is like shooting the messenger.

```ruby
def send_user_to_crm(user_params)
  begin

    # TODO: Think about moving this to background job
    CRMService.new.create(
      firstname: user_params[:first_name],
      lastname: user_params[:last_name],
      email: user_params[:email],
      phone: user_params[:phone],
      requested_from_page: user_params[:requested_from_page]
    )
  rescue StandardError => exception
    # TODO: Error handling
    Raven.capture_exception(exception)
    # or
    raise "Can't send user to CRM #{user_params[:email]}"
  end
end
```

You have two options. Swallowing the exception and propagating it to an error service like Sentry or raising a translation of the error to your system.

