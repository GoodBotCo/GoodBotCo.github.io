---
title: Nested Associations with Phoenix Forms
---

Elixir/Phoenix is still a fairly new territory for most developers, as it is for us, and it can be hard to do the most trivial things when starting off with a new framework. So sharing one of those experiences and the solution.

This blog post would share a simple example of storing nested associations (mainly a `has_many` association), with Phoenix forms. A more detailed version of this is available at [blogpost](http://blog.plataformatec.com.br/2015/08/working-with-ecto-associations-and-embeds/). We have a `Company`, and it has many `People`.

### `Company`

```ruby
defmodule Myapp.Company do
  use Myapp.Web, :model

  schema "companies" do
    field :name, :string

    has_many :people, Myapp.Person

    timestamps()
  end

  @doc """
  Builds a changeset based on the `struct` and `params`.
  """
  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:name])
    |> cast_assoc(:people)
    |> validate_required([:name])
  end
end
```

Note that we are using `cast_assoc/1` here. `cast_assoc/1` is used when you want to manage associations based on external parameters, like Phoenix forms. Ecto compares the data existing in the struct with the data sent through the form and generates the proper operation.

### `Person`

```ruby
defmodule Myapp.Person do
  use Myapp.Web, :model
  import Comeonin.Bcrypt, only: [checkpw: 2, hashpwsalt: 1]

  schema "people" do
    field :first_name, :string
    field :last_name, :string
    field :email, :string

    belongs_to :company, Myapp.Company

    timestamps()
  end

  def changeset(struct, params \\ %{}) do
    struct
    |> cast(params, [:first_name, :last_name, :email, :company_id])
    |> validate_required([:first_name, :last_name, :email])
    |> validate_length(:email, min: 1, max: 255)
    |> validate_format(:email, ~r/@/)
    |> unique_constraint(:email)
  end
end

```

### Template

We use Dynamic Forms and [Slim-Lang](https://github.com/slime-lang/phoenix_slime) for templates. Follow this [amazing blogpost](http://blog.plataformatec.com.br/2016/09/dynamic-forms-with-phoenix/) from [José Valim](http://blog.plataformatec.com.br/author/josevalim/) to learn how to build Dynamic Forms. For the sake of simplicity, we'll only save one person at the moment.

```ruby
= form_for @changeset, @action, fn c ->
  = input c, :name, label: "Company Name"
  = inputs_for c, :people, fn f ->
  	= input f, :first_name
  	= input f, :last_name
  	= input f, :email
  = submit "Continue"
```

### `CompanyController`

The `create` method was the trickiest to get right. I actually had to dig into the code for [Ecto Tests](https://github.com/elixir-ecto/ecto/blob/6f1971f4120b84e1a441792feb77ba451c4fc783/integration_test/cases/repo.exs#L633) to find a way to make this work. Again, for the sake of simplicity we are just storing one person.

```ruby
defmodule Myapp.CompanyController do
  use Myapp.Web, :controller

  alias Myapp.Company
  alias Myapp.Person

  plug :scrub_params, "company" when action in [:create]

  def new(conn, _params) do
    person = Person.changeset(%Person{})
    changeset = Company.changeset(%Company{people: [person]})
    
    render conn, "new.html", changeset: changeset
  end

  def create(conn, %{"company" => company_params}) do
    person_changeset = Person.changeset(%Person{}, company_params["people"]["0"])
    changeset =
      Company.changeset(%Company{}, %{name: company_params["name"]})
      |> Ecto.Changeset.put_assoc(:people, [person_changeset])

    case Repo.insert(changeset) do
      {:ok, company} ->
        person = Enum.at(company.people, 0) |> Repo.preload(:company)

        conn
        |> put_flash(:info, "Welcome to Myapp. #{person.first_name}!")
        |> redirect(to: page_path(conn, :index))
      {:error, changeset} ->
        conn
        |> render "new.html", changeset: changeset
    end
  end
end
```

`put_assoc` is typically used when we already have the associations as structs and changesets, and we tell Ecto to take those entries as is.

And that's it. Please do share your feedback in the comments below, or if there are better ways to approach this.
