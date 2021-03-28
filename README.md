### Practices I have come to enjoy - ruby edition

## Events

```ruby
Engine 1 public
Publisher.new.user_logged_in(current_user.id)

Engine 2 restricted access
def user_logged_in(user_id)
  board_member = BoardMember.find_by(
    org_number: User.find(user_id)&.org_number
  )
  if board_member
    notify_organization(board_member)
    setup_access(board_member)
    ...
  end
end
```

Pros
- Keep engines decoupled
- Easy to test
```ruby
it "gains access to organization" do
  Publisher.new.user_logged_in(user)
  expect(board_member.organizations).to include(organization)
end
```

## Keep controllers, models, rakefiles slim to some extent
```ruby
desc 'Sync boardmembers with bolagsverket for paying organizations'
batch_task sync_board_members: :environment do
  Bolagsverket::Sync::Task.new(Organization.customers).run
end
```
Pros
- Easier to maintain
- Better overview

## Log all communications with third party
```ruby
module CompXApi

  def log
    @log ||= Logger.new(Rails.root.join('log/comp_x.log'))
  end

  def headers(req, custom_headers)
    req.headers['Content-Type']  = 'application/json'
    req.headers['Accept']        = 'application/json'
  end

  def do_get(customer, path, custom_headers = {})
    statement = "Customer : #{customer.id}, GET #{path}"
    resp = Faraday.get(HOST + path) do |req|
      headers(req, custom_headers)
    end
    handle_response(statement, resp)
  end

  def handle_response(statement, resp)
    deserialize(resp.body).tap do |response|
      log.info(statement + " | #{resp.status} | RESP: #{resp.body.truncate(MAX_LOG_MSG_SIZE)}")
      if response.error?
        raise Error, response[:message]
      end
    end
  end
end
```
Pros
- DRY
- Easy to troubleshoot
