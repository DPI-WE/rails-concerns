# Using Modules to Organize and DRY Up Your Code 🧱

## Introduction to Modules
In your application, you might encounter scenarios where multiple classes share the same functionality. Managing this shared functionality can be cumbersome and might lead to code duplication. You may also encounter classes that become extremely long and difficult to manage. To tackle this, you can add the logic to **Modules** and then mix-in these features as needed. This lesson will guide you through understanding Modules, how they work, and how you can utilize them to keep your codebase clean, readable and DRY (Don't Repeat Yourself).

## Modules vs Classes
- **Classes** are used when you need to model objects that have both state (instance variables) and behavior (methods).
- **Modules** are used when you want to encapsulate behaviors that can be shared across multiple classes, or when you need to namespace similar groups of classes, methods, or constants to avoid naming conflicts.

```ruby
# Class example
class Dog
  def speak
    "Woof!"
  end
end

buddy = Dog.new  # Creating an instance of Dog
puts buddy.speak # Outputs: Woof!
```

```ruby
# Module example
module Bark
  def bark
    "Woof!"
  end
end

class Dog
  include Bark
end

buddy = Dog.new
puts buddy.bark # Outputs: Woof!
```

## What are Rails Concerns?
Rails provides a powerful feature called [Concerns](https://api.rubyonrails.org/v7.0/classes/ActiveSupport/Concern.html) which extends modules with specific features for Ruby on Rails applications. Rails Concerns are a way to make your Ruby on Rails application's code more modular. This is achieved by extracting code into modules that can be mixed into classes as needed. Concerns go into the `/concerns` directory and are used primarily to house shared methods between classes, but they can also contain validations, associations, or even callbacks.

## Key Benefits of Using Concerns:
- **DRY**: Reduces code duplication.
- **Organized**: Keeps related features together, making the codebase easier to navigate.
- **Reusable**: Allows functionalities to be reused across different models.

## How to Define a Concern
Let's create a concern for Taggable functionality that can be included in any model that handles tagging. This is how you would structure it:

1. Create the Concern Module
Define a module inside the `app/models/concerns` directory.

```ruby
# app/models/concerns/taggable.rb
module Taggable
  # ActiveSupport::Concern provides the framework for defining class-level and instance-level methods, along with the ability to run additional code when the module is included. 
  extend ActiveSupport::Concern

  included do
    has_many :tags, as: :taggable
  end

  def add_tag(tag_name)
    tags.create(name: tag_name)
  end
end
```

2. Include the Concern in Your Model
Now, simply include this concern in any model that requires tagging. Imagine you have a blog application where both `Article` and `Comment` models could benefit from tagging. By creating a `Taggable` concern, you can share the tagging functionality between these models without repeating code.

```ruby
# app/models/article.rb
class Article < ApplicationRecord
  include Taggable
end
```

```ruby
# app/models/comment.rb
class Comment < ApplicationRecord
  include Taggable
end
```

## Best Practices for Concerns
- **Keep Them Focused**: Each concern should serve a single, well-defined purpose.
- **Use `included` and `class_methods` Blocks**: These blocks help you define what code is executed when the concern is included, and allow you to neatly organize class methods.

```ruby
module Archivable
  extend ActiveSupport::Concern

  included do
    scope :archived, -> { where(archived: true) }
  end

  def archive!
    update(archived: true)
  end

  class_methods do
    def archive_all
      update_all(archived: true)
    end
  end
end
```

- **Consider Dependencies**: Make sure to handle any dependencies between concerns carefully.
- **SomeAction + able**: it's a common practice to name concerns `SomeAction`+`able` (e.g. `Csvable`, `Taggable`, etc.)

## Namespaced Concern: Order::Csvable
Concerns can be a great tool to leverage when organizing specific features in your codebase. Let's say you want to add CSV export functionality to the `Order` model in your application. Instead of adding the CSV-related methods directly into the `Order` model, you can create a concern under the `Order` namespace. Here's how you can structure this:

1. Create the Concern Module
Place the module inside the `app/models/concerns` directory, following the namespace of the model it pertains to.

```ruby
# app/models/concerns/order/csvable.rb
module Order::Csvable
  extend ActiveSupport::Concern

  class_methods do
    def to_csv
      CSV.generate(headers: true) do |csv|
        csv << column_names
        all.each do |order|
          csv << order.attributes.values_at(*column_names)
        end
      end
    end
  end
end
```

2. Add to the `Order` Model
Now, you can include this concern in the `Order` model to add CSV exporting capabilities.

```ruby
# app/models/order.rb
class Order < ApplicationRecord
  include Csvable
end
```

```ruby
class OrdersController < ApplicationController
  def index
    respond_to do |format|
      format.csv { send_data(@orders.to_csv, filename: "orders.csv") }
    end
  end
```

### Key Points in Using Namespaced Concerns
- **Organization**: The concern is kept under the `Order` namespace, which clearly indicates that this concern is specifically designed for the `Order` model.
- **Modularity**: It keeps the `Order` model lean and focused, while allowing you to extend its functionality in a modular way.
- **Scalability**: As your application grows, you can easily manage and update functionalities related to `Order` without overcrowding the model file.
- **File Location**: Ensure the concern is stored in the right directory (`app/models/concerns/order/csvable.rb`) to maintain the structure and avoid loading issues.

## Conclusion
Rails Concerns are a fantastic tool for any Ruby on Rails developer looking to refine their codebase. They promote cleaner, more maintainable code by reducing duplication and increasing modularity. By integrating concerns into your development workflow, you enhance the scalability and readability of your application.

## Resources
- [How Rails Concerns Work and How To Use Them](https://www.writesoftwarewell.com/how-rails-concerns-work-and-how-to-use-them/)
- [37 Signals: Vanilla Rails Is Plenty](https://dev.37signals.com/vanilla-rails-is-plenty)
- [Rails API documentation on ActiveSupport::Concern](https://api.rubyonrails.org/v7.1.3.2/classes/ActiveSupport/Concern.html)
