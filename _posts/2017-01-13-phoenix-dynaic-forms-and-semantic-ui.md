---
title: Phoenix dynamic forms and Semantic UI
---

If you are a Ruby on Rails developer, you might be familiar with the [Simple Form](https://github.com/plataformatec/simple_form) gem. In this post, we'll look at how to build that functionality in Phoenix. Dynamic forms will properly show input fields based on our schema information and also add validations, classes, and errors. We'd be using [Semantic UI](https://github.com/plataformatec/simple_form) as our CSS Framework. We are also using [Slim-lang](https://github.com/slime-lang/phoenix_slime) as our templating engine. We aim to make our forms look something like this.

```elixir
= form_for @changeset, @action, [class: "ui form"], fn f ->
  = input f, :name, label: "Your Name"
  = input f, :email
  = input f, :password
  = submit "Continue", class: "ui primary button"
```

### Let's go

We'll not dive into how to build a Phoenix application from scratch, you can follow the [Up and Running](http://www.phoenixframework.org/docs/up-and-running) Phoenix guide for that.

Without dynamic forms, the above form would look something like this.

```elixir
= form_for @changeset, @action, [class: "ui form"], fn f ->
  .field
    = label f, :name, "Your Name"
    = text_input f, :name
    = error_tag f, :name
  .field
    = label f, :email
    = email_input f, :email
    = error_tag f, :email
  .field
    = label f, :password
    = password_input f, :password
    = error_tag f, :password
 .field
    = submit "Continue", class: "ui primary button"
```

In order to automatically show validations in the form, we'll have to declare the validations in the changeset. Let's open `web/models/user.ex`, and add the validations.

```elixir
def changeset(struct, params \\ %{}) do
  struct
  |> validate_required([:name, :email, :password])
  |> validate_length(:password, min: 8, max: 25)
  |> validate_length(:email, min: 1, max: 255)
  |> validate_format(:email, ~r/@/)
  |> unique_constraint(:email)
end
```

### Adding the `input` function

We are ready with validations, let's add our `input` function. We'll add a new module `YourApp.InputHelpers`, and place it inside `web/views/input_helpers.ex`.

```elixir
defmodule YourApp.InputHelpers do
  use Phoenix.HTML
  
  def input(form, field) do
    # function definition pending
  end
end
```

Also, for our `InputHelpers` to be automatically available in all views, we'll have to add it to `web/web.ex`.

```elixir
defmodule YourApp.Web do
  ...
  
  def view do
    quote do
      use Phoenix.View, root: "web/templates"

      # Import convenience functions from controllers
      import Phoenix.Controller, only: [get_csrf_token: 0, get_flash: 2, view_module: 1]

      # Use all HTML functionality (forms, tags, etc)
      use Phoenix.HTML

      import YourApp.Router.Helpers
      import YourApp.ErrorHelpers
      import YourApp.InputHelpers # We'll add it here
      import YourApp.Gettext
    end
  end
  
  ...
end
```

Now that `InputHelpers` is setup, and imported. Let's change our form to use the `input` function.

```elixir
= form_for @changeset, @action, [class: "ui form"], fn f ->
  = input f, :name, label: "Your Name"
  = input f, :email
  = input f, :password
  = submit "Continue", class: "ui primary button"
```

### Implementing `input` function

After the above step, your page would be displaying nothing at the moment, that's ok! Let's get the input fields back.

```elixir
def input(form, field) do
  type = Phoenix.HTML.Form.input_type(form, field)
  apply(Phoenix.HTML.Form, type, [form, field])
end
```
Saving this should make the fields appear again on the page. Next step, is to add the right wrappers, labels, and errors. Also, let's add classes for Semantic UI.

```elixir
def input(form, field) do
  type = Phoenix.HTML.Form.input_type(form, field)
  wrapper_opts = [class: "field"]

  content_tag :div, wrapper_opts do
    label = label(form, field, humanize(field))
    input = apply(Phoenix.HTML.Form, type, [form, field])
    error = YourApp.ErrorHelpers.error_tag(form, field) || ""
    [label, input, error]
  end
end
```

That's mostly it. Your page should be generating the same markup it was generating with default Phoenix forms.

To take this further, let's add some customizing options for `input`. Let's start with adding error classes for Semantic UI.

```elixir
def input(form, field) do
  type = Phoenix.HTML.Form.input_type(form, field)
  wrapper_opts = [class: "field #{state_class(form, field)}"]

  content_tag :div, wrapper_opts do
    label = label(form, field, humanize(field))
    input = apply(Phoenix.HTML.Form, type, [form, field])
    error = YourApp.ErrorHelpers.error_tag(form, field) || ""
    [label, input, error]
  end
end

defp state_class(form, field) do
  cond do
    form.errors[field] -> "error"
    true -> nil
  end
end
```

### Adding validations to input fields

We'll be using `Phoenix.HTML.Form.input_validations` function to retreive the validations in our changeset. We'll declare `input_opts`, and then pass `input_opts` with `[form, field, input_opts]` to `input` inside the `content_tag`.

```elixir
def input(form, field) do
  type = Phoenix.HTML.Form.input_type(form, field)
  wrapper_opts = [class: "field #{state_class(form, field)}"]

  input_opts = Phoenix.HTML.Form.input_validations(form, field)

  content_tag :div, wrapper_opts do
    label = label(form, field, humanize(field))
    input = apply(Phoenix.HTML.Form, type, [form, field, input_opts])
    error = YourApp.ErrorHelpers.error_tag(form, field) || ""
    [label, input, error]
  end
end

defp state_class(form, field) do
  cond do
    form.errors[field] -> "error"
    true -> nil
  end
end
```

### Passing options with `input`

Remember above we had a custom label for `name`?

```elixir
.field
  = label f, :name, "Your Name"
  = text_input f, :name
  = error_tag f, :name
```

Let's get that back! We'll add the ability to pass options per input, like if you want to specify the `input_type` of an input, or a custom `label`. Let's add options to handle these cases:

```elixir
def input(form, field, opts \\ []) do
  type = opts[:using] || Phoenix.HTML.Form.input_type(form, field)
  label_value = opts[:label] || humanize(field)
```

And then let's update the `label` to use the `label_value` variable.

```elixir
content_tag :div, wrapper_opts do
  label = label(form, field, label_value)
```

Alright, finally. Let's see how our `InputHelpers` look after all this.

```elixir
defmodule YourApp.InputHelpers do
  use Phoenix.HTML

  def input(form, field, opts \\ []) do
    type = opts[:using] || Phoenix.HTML.Form.input_type(form, field)
    label_value = opts[:label] || humanize(field)

    wrapper_opts = [class: "field #{state_class(form, field)}"]
    input_opts = [] # To pass custom options to input

    validations = Phoenix.HTML.Form.input_validations(form, field)
    input_opts = Keyword.merge(validations, input_opts)

    content_tag :div, wrapper_opts do
      label = label(form, field, label_value)
      input = input(type, form, field, input_opts)
      error = YourApp.ErrorHelpers.error_tag(form, field) || ""
      [label, input, error]
    end
  end

  defp state_class(form, field) do
    cond do
      form.errors[field] -> "error"
      true -> nil
    end
  end

  defp input(type, form, field, input_opts) do
    apply(Phoenix.HTML.Form, type, [form, field, input_opts])
  end
end
```

To sum it all up. We can essentially write Simple Form in Phoenix in approximately less than 30 minutes. If you have any value additions, please leave it a comment below, thanks!
