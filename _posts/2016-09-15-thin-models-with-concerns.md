---
title: How to make fat models thin again with Rails Concerns
tags: [RailsTips, Programming, Refactoring]
---

One of the most common principles of Rails you might hear when starting off is - *Fat Models, Thin Controllers*. For sure, fat models are way better than having fat controllers, it helps create a better API for any model that you might have, but I am sure your `User` model is already going out of hand.

That’s where Concerns come in helpful. They are a great way to extract a part of a model that does not seem to belong there, what belongs there and doesn’t is purely subjective and depends on your project, but you would surely see patterns. This will also help you go full-bore on Single Responsibility Principle, the benefits of which can be found all over the internet.

Here is a typical scenario of where Concerns come in useful, with an all famous example of a News application in the works. Of course, the news app will have a `News` model, but let’s also have a simple `Comment` model.

```ruby
class News < ActiveRecord::Base
  validates :content, presence: true
  validates :title, presence: true
  validates :category_id, presence: true
  
  has_many :comments
end

class Comment < ActiveRecord::Base
  validates :content, presence: true
  
  belongs_to :news
end
```

Pretty straightforward. We soon realise that news sure seems nice, we should give our users a way to like them.

```ruby
class News < ActiveRecord::Base
  validates :content, presence: true
  validates :title, presence: true
  
  has_many :comments
  has_many :likes
  
  def like!
    Like.create news: self
  end
end

class Comment < ActiveRecord::Base
  validates :content, presence: true
  
  belongs_to :news
end

class Like < ActiveRecord::Base
  belongs_to :news
end
```

Likes are flowing in your news, and so are comments. You think it might be cool to add likes for comments too. So, the first thing that might come to your mind would be to implement a basic polymorphic relation.

```ruby
class News < ActiveRecord::Base
  validates :content, presence: true
  validates :title, presence: true
  
  has_many :comments
  has_many :likes, as: likeable
  
  def like!
    Like.create likeable: self
  end
end

class Comment < ActiveRecord::Base
  validates :content, presence: true
  belongs_to :news
  has_many :likes, as: :likeable
  
  def like!
    Like.create likeable: self
  end
end

class Like < ActiveRecord::Base
  belongs_to :likeable, polymorphic: true
end
```

You see, here the method `#like!` is already duplicate code between the `News`, and `Comment` models. In the future, if you decide to update the Like API, you’ll probably have to add more duplicate code in `News` and `Comment`. Well, have no Concern young one, Concern to the rescue.

Here is how you can refactor the above code `ActiveModel::Concern`.

```ruby
class News < ActiveRecord::Base
  include Likeable
  
  validates :content, presence: true
  validates :title, presence: true
  
  has_many :comments
end

class Comment < ActiveRecord::Base
  include Likeable
  
  validates :content, presence: true
  
  belongs_to :news
end

class Like < ActiveRecord::Base
  belongs_to :likeable, polymorphic: true
end

# Likeable Concern
module Likeable
  extend ActiveModel::Concern
  
  included do
    has_many :likes, as: :likeable
  end
  
  def like!
    likes.create
  end
end
```

Concerns are nothing but modules that can encapsulate APIs related to different models into a single file, they drastically reduce the redundancy in your codebase, while making it easy to test and maintain. The most simplest examples above, both News and Comment are Likeable, hence, through the Likeable concern they share the behaviour.
To see how Concerns might be used in a big live project, checkout this blog post by 37Signals, on how they use concerns in Basecamp - [Link](https://signalvnoise.com/posts/3372-put-chubby-models-on-a-diet-with-concerns)
