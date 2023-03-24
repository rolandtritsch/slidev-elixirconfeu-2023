# Lecture 3: Phoenix & Ecto

## From the bottom up

[not_skull repo](https://github.com/idlehands/not_skull)

* Schema
  * Ecto Schema
    * Definition
    * Changeset function
* Context
  * Ecto Queries
* Controller
  * Mock
  * Endpoint
<!--

The anatomy of a test ...

-->

---
---
## Motivation

* Let's see a whole API call stack tested
* Let's talk about test data generation
* 

<!--

The anatomy of a test ...

-->

---
---
# Schema
lib/not_skull/accounts/user.ex

```elixir{|1|2|8-14|}
defmodule NotSkull.Accounts.User do
  use NotSkull.Schema
  import Ecto.Changeset

  @optional_create_fields [:id, :inserted_at, :updated_at]
  @forbidden_update_fields [:id, :inserted_at, :updated_at]

  schema "users" do
    field(:email, :string)
    field(:name, :string)
    field(:password, :string)

    timestamps()
  end
```

<!--

The anatomy of a test ...

-->
---
---
# Custom Schema defaults
lib/not_skull/schema.ex

```elixir{|4|10-12}
defmodule NotSkull.Schema do
  @moduledoc false

  defmacro __using__(_) do
    quote do
      use Ecto.Schema
      import NotSkull.Schema
      import Ecto.Changeset

      @primary_key {:id, :binary_id, autogenerate: true}
      @foreign_key_type :binary_id
      @timestamps_opts type: :utc_datetime_usec
    end
  end
end
```

<!--

The anatomy of a test ...

-->

---
---
# Schema
lib/not_skull/accounts/user.ex

```elixir{8-14}
defmodule NotSkull.Accounts.User do
  use NotSkull.Schema
  import Ecto.Changeset

  @optional_create_fields [:id, :inserted_at, :updated_at]
  @forbidden_update_fields [:id, :inserted_at, :updated_at]

  schema "users" do
    field(:email, :string)
    field(:name, :string)
    field(:password, :string)

    timestamps()
  end
```

<!--

The anatomy of a test ...

-->

---
---
# Locking down the Schema Definition

```elixir{|2|5-12|16-19}
defmodule NotSkull.Accounts.UserTest do
  use NotSkull.DataCase
  alias NotSkull.Accounts.User

  @expected_fields_with_types [
    {:id, :binary_id},
    {:email, :string},
    {:name, :string},
    {:password, :string},
    {:inserted_at, :utc_datetime_usec},
    {:updated_at, :utc_datetime_usec}
  ]

  # ... other attributes

  describe "fields and types" do
    @tag :schema_definition
    validate_schema_fields_and_types(User, @expected_fields_with_types)
  end
  # ...
end
```

<!--
speaker notes:
-->
---
---
# Title

```bash
  4) test fields and types Elixir.NotSkull.Accounts.User: it has the correct fields and types (NotSkull.Accounts.UserTest)
     test/accounts/user_test.exs:22
     Assertion with == failed
     code:  assert Enum.sort(expected) == Enum.sort(actual)
     left:  [{:id, :binary_id}, {:inserted_at, :utc_datetime_usec}, {:name, :string}, {:password, :string}, {:updated_at, :utc_datetime_usec}]
     right: [
              {:email, :string},
              {:id, :binary_id},
              {:inserted_at, :utc_datetime_usec},
              {:name, :string},
              {:password, :string},
              {:updated_at, :utc_datetime_usec}
            ]
     stacktrace:
       test/accounts/user_test.exs:22: (test)
```

<!--
speaker notes:
but with color, assuming 'Nix system
-->
---
---
# Testing a Changeset

```elixir{|4|11-13|17|18|19|20|21|}
defmodule NotSkull.Accounts.User do
  use NotSkull.Schema

  @optional_create_fields [:id, :inserted_at, :updated_at]
  @forbidden_update_fields [:id, :inserted_at, :updated_at]

  schema "users" do
    # schema definition here ...
  end

  defp all_fields do
    __MODULE__.__schema__(:fields)
  end

  def create_changeset(params) do
    %__MODULE__{}
    |> cast(params, all_fields())
    |> validate_required(all_fields() -- @optional_create_fields)
    |> cleanup_email()
    |> hash_password()
    |> unique_constraint(:email)
  end
```

<!--
speaker notes:
The schema can use it's own definition dynamically because it is locked down by the test we looked at.
-->
---
---
# Happy Path

```elixir{|}
defmodule NotSkull.Accounts.UserTest do
  use NotSkull.DataCase
  alias NotSkull.Accounts.User
# ...
  describe "create_changeset/1" do
    test "success: returns a valid changeset when given valid arguments" do
      valid_params = Factory.valid_params(@expected_fields_with_types)

      changeset = User.create_changeset(valid_params)
      assert %Changeset{valid?: true, changes: changes} = changeset

      mutated = [:password]

      assert_values_for(
        expected: {valid_params, :string_keys},
        actual: changes,
        fields: Keyword.keys(@expected_fields_with_types),
        skip_fields: mutated
      )

      assert "$argon2id$v=19$m=131072,t=8,p=4" <> _hashed_pw =
               changes.password
    end
# ...
end
```

<!--
speaker notes:
-->
