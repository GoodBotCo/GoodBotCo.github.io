---
title: Using PostgreSQL Array Type with Rails 4
---

With Rails 4 came support for array fields for PostgreSQL. However, this pattern of storing data is not quite popular. So we thought it would be nice to share where this can be used in a project.

Say you have a `News` model, which belongs to `Category`, and has the fields `content`, and `category_id` (as the foriegn key). Also, let's assume, you want to tag certain news with certain keywords. These keywords can be stored in a PostgreSQL Array type.

Writing migrations for these tables is pretty straightforward, except for the `tags` column, which is an array type in this case. We use the following syntax to create our migration.

```ruby
create_table :categories do |t|
  t.string :name, null: false
end

create_table :news do |t|
  t.string :content, null: false
  t.references :category, null: false
  t.string :tags, array: true, default: []
end
```

A few things to note here are:

- Specify datatype as `string` or `text`, with `array: true`
- Optionally, you can specify a default value for the array by adding: `default: []`.

The Model code should look like this:

```ruby
class News < ActiveRecord::Base
  validates :content, presence: true
  
  belongs_to :category
end

class Category < ActiveRecord::Base
  validates :name, presence: true
  
  has_many :news
end
```

`ActiveRecord` returns PostgreSQL arrays as a Ruby Array. Here is an example of that.

```ruby
$ bundle exec rails c

> news = News.create(content: "Lorem Ipsum", category: Category.first, tags: ['Awesome', 'Rails', 'Tips'])
=> #<News id: nil, content: "Lorem Ipsum", created_at: "2016-09-17 08:21:17", updated_at: "2016-09-17 08:21:17", category_id: 1, tags: ["Awesome", "Rails", "Tips"]>

> news.tags
=> ["Awesome", "Rails", "Tips"]

> news.tags.class
=> Array
```

Following is an example of how you could interact with the `Tags` column. Because its a Ruby Array, all the Array methods are available to interact it.

```ruby
2.2.0 :011 > news.tags
 => ["Awesome", "Rails", "Tips"]
 
2.2.0 :012 > news.tags << "Now"
 => ["Awesome", "Rails", "Tips", "Now"]
 
2.2.0 :013 > news.save!
   (0.2ms)  BEGIN
  SQL (0.3ms)  UPDATE "news" SET "tags" = $1, "updated_at" = $2 WHERE "news"."id" = $3  [["tags", "{Awesome,Rails,Tips,Now}"], ["updated_at", "2016-09-22 13:52:40.184161"], ["id", 221]]
   (6.1ms)  COMMIT
 => true
```

I hope this helps you create better Models for your project, and you don't end up creating a whole `Tags` table, like we were about to do for one of our projects. :)

As they say, *a little knowledge is dangerous*.

Happy Postgresing!
