# Cat Tinder API Validations

#### Overview

As developers we have to think about what happens when things don't go as we expect. What if data is submitted to our API that isn't complete, or has something else that causes it to be invalid? This could cause harm to our database or affect the user experience.

#### Previous Lecture (46 min)

[![YouTube](http://img.youtube.com/vi/r6dWvvc6P5k/0.jpg)](https://www.youtube.com/watch?v=r6dWvvc6P5k)

#### Learning Objectives

- can validate model attributes
- can create specs for corresponding model validations

#### Additional Resources

- [Response Codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

### Troubleshooting Tips

- Did you create your database?
- Did you migrate?
- Errors? Always look at the first error in the list.

---

### Validations

As developers, it is our job to ensure no matter what, that our app responds in predictable ways to every request, and over many requests.

The primary tool we have to assure predictable results is to assure that the data we commit to our database is in a form that we expect it to be, and for this we use validations in a Rails application. When the incoming data looks to be correct, we commit it, when it isn't correct, we reject it, and respond with a reason why it was not accepted.

### Why Validations?

We've started all of our backend coding with a failing test, and we now have good test coverage of what we expect the API to do when it is passed good data. What we need now is a test of what we expect it to do when passed bad data. What happens when a user submits a cat without a name? Do we accept it, or do we reject. Same question goes for the cat's age and what our cat enjoys. It may be acceptable to create a record without the enjoys data, but we should require an age.

Let's write a test for each of these cases.

### Model Specs

We can create a test that will look for an error if a cat is created without any attributes.

**spec/models/cat_spec.rb**

```ruby
RSpec.describe Cat, type: :model do
  it "should validate name" do
    cat = Cat.create(age: 2, enjoys: 'Walks in the park', image: 'https://images.unsplash.com/photo-1529778873920-4da4926a72c2?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1036&q=80')
    expect(cat.errors[:name]).to_not be_empty
  end
end
```

$ `rspec spec/models`

```bash
1 example, 1 failure

Failures:

  1) Cat should validate name
     Failure/Error: expect(cat.errors[:name]).to_not be_empty
       expected `[].empty?` to return false, got true
```

As we run our specs, we see the test fails. But we can make it pass with one line of code in the model.

**app/models/cat.rb**

```ruby
class Cat < ApplicationRecord
  validates :name, presence: true
end
```

$ `rspec spec/models`

```bash
1 example, 0 failures
```

Green!

This process can be repeated to ensure our specs cover all the attributes of our cat model.

### Request Specs

Here's a test to assure that we get the correct response status when we submit a create request without a name for a cat.

**spec/requests/cats_request_spec.rb**

```ruby
 it "doesn't create a cat without a name" do
   cat_params = {
     cat: {
       age: 2,
       enjoys: 'Walks in the park',
       image: 'https://images.unsplash.com/photo-1529778873920-4da4926a72c2?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1036&q=80'
     }
   }
   # Send the request to the  server
   post '/cats', params: cat_params
   # expect an error if the cat_params does not have a name
   expect(response.status).to eq 422
   # Convert the JSON response into a Ruby Hash
   json = JSON.parse(response.body)
   # Errors are returned as an array because there could be more than one, if there are more than one validation failures on an attribute.
   expect(json['name']).to include "can't be blank"
 end
```

Not only do we want to make sure we are only allowing good data to be added to the database, we want to check that the API lets us know what is wrong, so the frontend can prompt the user to fix it.

$ `rspec spec/requests`

```bash
Failures:

  1) Cats does not create a cat without a name
     Failure/Error: expect(response.status).to eq 422

       expected: 422
            got: 200

       (compared using ==)
     # ./spec/requests/cats_request_spec.rb:40:in `block (2 levels) in <main>'
```

And the results from running the full test suite give us a failure. Great! We expected a 422 response which is the server letting us know that we submitted an "Unprocessable Entity", but that's not what we got back. So how do we make that test pass? Let's add a conditional statement to our create method in the controller to throw a 422 error if the cat is not valid.

**app/controllers/cats_controller.rb**

```ruby
def create
  cat = Cat.create(cat_params)
  if cat.valid?
    render json: cat
   else
     render json: cat.errors, status: 422
   end
end
```

$ rspec spec/requests

```bash
1 example, 0 failures
```

And now we're green!

---

### 🐱 Challenge: Cat Tinder API Validations

As a developer, I have been commissioned to create an application where a user can see cute cats looking for friends. As a user, I can see a list of cats. I can click on a cat and see more information about that cat. I can also add cats to the list of cats looking for friends. If my work is acceptable to my client, I may also be asked to add the ability to remove a cat from the list as well as edit cat information.

- As a developer, I can add the appropriate model specs that will ensure an incomplete cat throws an error.
- As a developer, I can add the appropriate model validations to ensure the user submits a name, an age, what the cat enjoys, and an image.
- As a developer, I can add the appropriate model specs that will ensure a cat enjoys entry is at least 10 characters long.
- As a developer, I can add a validation that will ensure a cat enjoys entry is at least 10 characters long.
- As a developer, I can add the appropriate request validations to ensure the API is sending useful information to the frontend developer if a new cat is not valid.
- As a developer, I can add the appropriate request spec that will look for a 422 error if the create validations are not met.
- STRETCH: As a developer, I can add the appropriate request validations to ensure the API is sending useful information to the frontend developer if an updated cat is not valid.
- STRETCH: As a developer, I can add the appropriate request spec that will look for a 422 error if the update validations are not met.

---

[Back to Syllabus](../../README.md#cat-tinder-backend)
