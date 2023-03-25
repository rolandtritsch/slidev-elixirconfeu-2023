# Lecture 3: Phoenix & Ecto

## From the bottom up

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

- will also cover
  - test case files for common setup
  - test data generation, or "factory" modules

-->

---
---
## Motivation

* Let's see a whole API call stack tested
* Let's talk about test data generation
* 

<!--

- first note

-->
---
---
# Intro to the app

* not skull is a game: [not_skull repo](https://github.com/idlehands/not_skull)
* it originally started as me working on a sample app, making a copy of one of
  my favorite card games, Skull
* we will be focused on the api for user creation

<!--
- emphasis is on testing, and not necessarily a recommendation of how to write your code.
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

- first, just the top of the file, defining the module
- here's a reference to our custom schema file. We will dig into this in a few minutes.
- last, here is our actual field and type definitions. standard stuff
- let's look at that schema file

-->
---
---
# Custom Schema defaults
lib/not_skull/schema.ex

```elixir{|4|6|10-12}
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

- first thing to note is that we're defining __using__ so that our file is pulled in the same what that Ecto.Schema would be
- next thing is that it "uses" Ecto.Schema, everything here is just for overrrides
- These are the overrides. (next slide)

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

- the implicit field called "id" and the timestamps are impacted by those overrides.

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
- Here's where we start
- note that we're using DataCase. As I mentioned earlier, we will talk a little about test cases later
- Ever hear people complain that testing is like writing their code twice? Here
  that's intentional. By having explicit definitions for the schema, we are able
  to assure that the definitions themselves are a reliable source of information
  and we can use those definitions dynamically. Just hold on and we'll show
  examples in both our test and our code under test.
- Here we have a macro that will go through the schema and make sure they match.
- We will skip looking at the macro (but it's all yours) but let's see how it fails
-->

---
---
# Title

```bash{|7}
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
- we're missing our standard color output
- this error is telling us that email is showing up in the schema definition and not the test
- now that we have our definition locked down pretty well, let's take a look at testing a changeset function.
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
- notice we have a list of fields that are optional
- The schema can use it's own definition dynamically because it is locked down by the test we looked at.
- each of these lines will need to be covered in a test:
  - cast
  - validate_required
  - cleanup email
  - hash password
  - unique constraint on email
-->
---
---
# Happy Path

```elixir{|7-9|10|12-19|21-22}
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
- happy path will cover:
  - cast
  - hash password
- here is our first place for using test data generation. we will look into that in a few minutes
- 7-9 is setup, no writing to the db
- line 10 is the kick off
- 14-19 is assert values for. that's our next digression
- 21-22 is a light check to make sure that the pw hashing is working
- let's look at assert_values_for
-->
---
---
# CheckerCab.assert_values_for/1

```elixir{|4|5|6|8|11-12}
      mutated = [:password]

      assert_values_for(
        expected: {valid_params, :string_keys},
        actual: changes, # {changes, :atom_keys} is allowed, too
        fields: Keyword.keys(@expected_fields_with_types),
	# fields: fields_for(User) would also work
        skip_fields: mutated
      )

      assert "$argon2id$v=19$m=131072,t=8,p=4" <> _hashed_pw =
               changes.password
```

<!--
speaker notes:
- assert_values_for comes from CheckerCab. Once you get used to the api, it's super useful to make large explicit checks between near matching key'd data structures (maps and structs)
- line 4 is where we set up our expected. CheckerCab assumes atom keys unless told otherwise
- line 5 is what we actually got back
- line 6 just needs a list of atoms. here we are using the definition from the test
  - we could also use the fields_for helper and pass in the User. It's the same list.
- Not only do we have different key types, but there is a field we know will not match
  - anything that is skipped should be explicitly tested below
-->
---
---
# CheckerCab output

```bash{|5-4|8-11}
  1) test create_changeset/1 success: returns a valid changeset when given valid arguments (NotSkull.Accounts.UserTest)
     test/accounts/user_test.exs:26
     There were issues the comparison:
     
     Key(s) missing:
       field: :email didn't exist in actual
     
     Values did not match for:
       field: :name
         expected: "Jeffrey"
         actual: "fugit"
     code: assert_values_for(
     stacktrace:
       (checker_cab 1.1.0) lib/checker_cab.ex:159: CheckerCab.assert_values_for/1
       test/accounts/user_test.exs:36: (test)
```

<!--
speaker notes:
- assert values for will tell you if either one is missing a field
- and it will tell you which values don't match
-->
---
---
# Happy Path in review

```elixir
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
- because there are no side effects, our test ends there
-->
---
---
# Check the email cleanup

```elixir{|2-3|5-6|8-11|13}
    test "success: it downcases and strips email address" do
      bad_email = "Jeffrey@matthias.org "
      params = Factory.string_params(:user, %{email: bad_email})

      assert %Changeset{valid?: true, changes: changes} =
               User.create_changeset(params)

      expected_email =
        bad_email
        |> String.downcase()
        |> String.trim()

      assert changes.email == expected_email
    end
```

<!--
speaker notes:
- pretty straightforward
- explicitly passing in the one field that we need set, randomizing the rest of the data
- kickoff + assert
- constructing the expected change
  - this may be the exact code on the other side, don't worry. this is an example of basic regression proofing
  - assertion
-->
---
---
# Let's talk Test Data Generation

- Often referred to as a "Factory"
  - Based off of old Java pattern, but it isn't exactly the right name
- Most common library is ExMachina
- Number one rule: don't nest the creation of resources
- I wrote my own here

<!--
speaker notes:
-
-->
---
---
# Seeing how the sausage is made

```elixir{|6-12|14-24}
defmodule Support.Factory do
  alias NotSkull.Accounts.User
  alias NotSkull.GameEngine
  alias NotSkull.Repo
# ...
  def params(:user) do
    %{
      email: email(),
      name: first_name(),
      password: password()
    }
  end
# ...
  def string_params(factory_name, overrides \\ %{})
      when is_atom(factory_name) and is_map(overrides) do
    atom_params(factory_name, overrides)
    |> convert_atom_keys_to_strings()
  end

  def atom_params(factory_name, overrides \\ %{})
      when is_atom(factory_name) and is_map(overrides) do
    defaults = params(factory_name)
    Map.merge(defaults, overrides)
  end
  # ...
end
```

<!--
speaker notes:
- these are the minimum required fields
- and just basic code to return an atom or string keyed map
-->
---
---
# Valid Params

```elixir{|2-8|10-12}
  def valid_params(fields_with_types) do
    valid_value_by_type = %{
      date: fn -> to_string(Faker.Date.date_of_birth()) end,
      float: fn -> :rand.uniform() * 10 end,
      string: fn -> Faker.Lorem.word() end,
      utc_datetime_usec: fn -> DateTime.utc_now() end,
      binary_id: fn -> Ecto.UUID.generate() end
    }

    for {field, type} <- fields_with_types, into: %{} do
      {Atom.to_string(field), valid_value_by_type[type].()}
    end
  end
```

<!--
speaker notes:
- we have a generation function that will provide a valid value for the type
- it's then converted to a map, using the provided field name for the key
-->

---
---
# Invalid Params

```elixir{|2-8|10-12}
  def invalid_params(fields_with_types) do
    invalid_value_by_type = %{
      date: fn -> Faker.Lorem.word() end,
      float: fn -> Faker.Lorem.word() end,
      string: fn -> DateTime.utc_now() end,
      utc_datetime_usec: fn -> Faker.Lorem.word() end,
      binary_id: fn -> 1 end
    }

    for {field, type} <- fields_with_types, into: %{} do
      {Atom.to_string(field), invalid_value_by_type[type].()}
    end
  end
```

<!--
speaker notes:
- we have a generation function that will provide a value that CAN NOT be cast to the correct type
- it's then converted to a map, using the provided field name for the key
-->

---
---
# When to randomize data?

* intentional data: will impact the behavior of the code
  * DO NOT randomize
* incidental data: will not impact the behavior of the code
  * DO randomize

<!--
speaker notes:
-
-->
---
---
# Testing Cast

```elixir{|2|4-5|7-13}
    test "error: returns an error changeset when given un-castable values" do
      invalid_params = Factory.invalid_params(@expected_fields_with_types)

      assert %Changeset{valid?: false, errors: errors} =
               User.create_changeset(invalid_params)

      for {field, _} <- @expected_fields_with_types do
        assert errors[field], "The field :#{field} is missing from errors."
        {_, meta} = errors[field]

        assert meta[:validation] == :cast,
               "The validation type, #{meta[:validation]}, is incorrect."
      end
    end
```

<!--
speaker notes:
- make un-cast-able data
- kickoff
- systematically go through the errors and check that there is a cast error for each one
- We are not matching against error strings, allowing the library to change without breaking our test
-->

---
---

# Testing required fields

```elixir{|2|4-5|7-14|8|7-14}
    test "error: returns error changeset when required fields are missing" do
      params = %{}

      assert %Changeset{valid?: false, errors: errors} =
               User.create_changeset(params)

      for {field, _} <- @expected_fields_with_types,
          field not in @optional_for_create do
        assert errors[field], "The field :#{field} is missing from errors."
        {_, meta} = errors[field]

        assert meta[:validation] == :required,
               "The validation type, #{meta[:validation]}, is incorrect."
      end
    end
```

<!--
speaker notes:
- empty params
- kickoff
- check against the fields, but note that we are skipping the ones that the test knows as optional
- is there a way we can take this further? Is it worth it?
- 
-->
---
---
# Testing Unique Constraint

```elixir{|2|4-7|9-10|12-16|}
    test "error: returns error changeset when an email address is reused" do
      {:ok, existing_user} = Factory.insert(:user)

      changeset_with_repeated_email =
        Factory.valid_params(@expected_fields_with_types)
        |> Map.put("email", existing_user.email)
        |> User.create_changeset()

      assert {:error, %Changeset{valid?: false, errors: errors}} =
               NotSkull.Repo.insert(changeset_with_repeated_email)

      assert errors[:email], "The field :email is missing from errors."
      {_, meta} = errors[:email]

      assert meta[:constraint] == :unique,
             "The validation type, #{meta[:validation]}, is incorrect."
    end
```

<!--
speaker notes:
- this is our first test that actually interacts with db specifically because
unique constraints aren't called until the db reports a conflict. This assumes familiarity with the Ecto Sandbox.
- inserting an existing user
- creating a changeset for a user with the same email
- inserting to get that db feedback and generate the error
- assert the error is there
-->
---
---
# Summary on testing the Schema

- We locked down the definition
- We used the definition from both the test and the schema as a reference
- We made sure there was a test to cover every part of the changeset
- We have covered all of the code paths, allowing us to be more lax higher up the test stack

<!--
speaker notes:
-
-->
---
---
# Testing the Context

- We've covered all of the code paths and possible errors
- Just need a happy path and 1 error path to make sure the context handles it correctly

<!--
speaker notes:
-
-->
---
---
# Context Code

```elixir{|8-12|10|11}
defmodule NotSkull.Accounts do
  @moduledoc false
  alias NotSkull.Repo
  alias NotSkull.Accounts.User
  alias NotSkull.Errors.{ResourceNotFoundError, InvalidEmailOrPasswordError}

# ...
  def create_user(params) do
    params
    |> User.create_changeset()
    |> Repo.insert()
  end
# ...
end
```

<!--
speaker notes:
- super basic create function
- The changeset code is well tested
- the new thing here is that we're making a call to the db
- we don't mock the db, because it's faster and conceptually easier to use the sandbox
- Repo.insert will either work or return a error changeset
- We have both return value and side effects
-->
---
---
# Testing the happy path

```elixir{|9|11|13-15|17-22}
defmodule NotSkull.AccountsTest do
  use NotSkull.DataCase
  alias NotSkull.Accounts
  alias NotSkull.Accounts.User

# ...
  describe "create_user/1" do
    test "success: it creates and returns a user when given valid params" do
      params = Factory.string_params(:user)

      assert {:ok, %User{} = returned_user} = Accounts.create_user(params)

      user_from_db = Repo.get(User, returned_user.id)

      assert user_from_db == returned_user

      assert_values_for(
        expected: {params, :string_keys},
        actual: user_from_db,
        fields: fields_for(User) -- db_assigned_fields(plus: [:password])
      )
    end
# ...
  end
# ...
end
```

<!--
speaker notes:
- standard setup. no specific values should impact this, as long as everything is valid
- kickoff and assertion
- getting the inserted record from the db: checking the side effect
- once we've validated that the return value and the side effect are perfect matches
- the only thing left is to verify that the actual data is what was expected.
- db_assigned_fields is just me playing around, looking for better ways to write things
- we aren't checking the skipped fields here, because we can't predict the values for any of them
-->
---
---
# Sad Path

```elixir
    test "error: it returns an error/changeset tuple when user can't be created" do
      bad_params = %{}

      assert {:error, %Changeset{}} = Accounts.create_user(bad_params)
    end
```

<!--
speaker notes:
- That's it! that's the whole sad path.
- We will pass the error changeset up the call stack to let the controller decide how to render it.
-->
---
---
# Controller Test

- Single happy path test, single error path test

<!--
speaker notes:
-
-->

---
---
# The Controller Code

```elixir{|8|9-10|12-14|16-19}
defmodule NotSkullWeb.JsonApi.UserController do
  use NotSkullWeb, :controller

  alias NotSkull.Accounts
  alias NotSkull.ExternalServices.Email

  def create(conn, params) do
    case Accounts.create_user(params) do
      {:ok, user} ->
        Email.send_welcome(user)

        conn
        |> put_status(201)
        |> json(user_map_from_struct(user))

      {:error, error_changeset} ->
        conn
        |> put_status(422)
        |> json(errors_from_changeset(error_changeset))
    end
  end
# ...

end
```

<!--
speaker notes:
- a straight call to the context code
- if successful, an email is sent
- and the new user is rendered with a 201, created
- otherwise, the changeset is rendered
- there are two "view" functions here. let's look at them, too
-->
---
---
# "View" code

```elixir{1-5|7-13}
  defp user_map_from_struct(user) do
    user
    |> Map.from_struct()
    |> Map.drop([:__struct__, :__meta__])
  end

  defp errors_from_changeset(changeset) do
    serializable_errors =
      for {field, {message, _}} <- changeset.errors,
          do: %{"field" => to_string(field), "message" => message}

    %{errors: serializable_errors}
  end
```

<!--
speaker notes:
- I chose not to use phoenix views here. It's just a style choice.
- When I have small responses, I tend to dislike the "magic" of view modules
-->

---
---
# Happy Path Test - return value

```elixir{|}
defmodule NotSkullWeb.JsonApi.UserControllerTest do
  use NotSkullWeb.ConnCase, async: false

  alias NotSkull.Accounts.User

  describe "POST /users" do
    test "success: creates user, returns user and 201",
         %{conn: conn} do
      params = Factory.string_params(:user)

      # welcome email should get sent
      expect_email_to(params["email"])

      conn = post(conn, "/api/users", params)

      # check return value (tests "view" code)
      assert response_body = json_response(conn, 201)

      assert_values_for(
        expected: {params, :string_keys},
        actual: {response_body, :string_keys},
        fields: fields_for(User),
        skip_fields: db_assigned_fields(plus: [:password])
      )
# ...
```

<!--
speaker notes:
-
-->
---
---
# Happy Path Test - side effect

```elixir{|}

      # check side effects
      user_from_db = Repo.get(User, response_body["id"])

      assert_values_for(
        expected: {params, :string_keys},
        actual: user_from_db,
        fields: fields_for(User),
        skip_fields: db_assigned_fields(plus: [:password])
      )
    end
```

<!--
speaker notes:
-
-->

