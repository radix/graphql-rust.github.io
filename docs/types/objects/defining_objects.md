# Defining objects

While any type in Rust can be exposed as a GraphQL object, the most common one
is a struct.

There are two ways to create a GraphQL object in Juniper. If you've got a simple
struct you want to expose, the easiest way is to use the custom derive
attribute. The other way is described in the [Complex fields](complex_fields.md)
chapter.

```rust
extern crate juniper;
#[macro_use] extern crate juniper_codegen;

#[derive(GraphQLObject)]
struct Person {
    name: String,
    age: i32,
}

# fn main() {}
```

This will create a GraphQL object type called `Person`, with two fields: `name`
of type `String!`, and `age` of type `Int!`. Because of Rust's type system,
everything is exported as non-null by default. If you need a nullable field, you
can use `Option<T>`.

We should take advantage of the
fact that GraphQL is self-documenting and add descriptions to the type and
fields:

```rust
# extern crate juniper;
# #[macro_use] extern crate juniper_codegen;
# 
#[derive(GraphQLObject)]
#[graphql(description="Information about a person")]
struct Person {
    #[graphql(description="The person's full name, including both first and last names")]
    name: String,

    #[graphql(description="The person's age in years, rounded down")]
    age: i32,
}

# fn main() {}
```

## Relationships

You can only use the custom derive attribute under these circumstances:

* The annotated type is a `struct`,
* Every struct field is either
  * A primitive type (`i32`, `f64`, `bool`, `String`, `juniper::ID`), or
  * A valid custom GraphQL type, e.g. another struct marked with this attribute,
    or
  * A container/reference containing any of the above, e.g. `Vec<T>`, `Box<T>`,
    `Option<T>`

Let's see what that means for building relationships between objects:

```rust
extern crate juniper;
#[macro_use] extern crate juniper_codegen;

#[derive(GraphQLObject)]
struct Person {
    name: String,
    age: i32,
}

#[derive(GraphQLObject)]
struct House {
    address: Option<String>, // Converted into String (nullable)
    inhabitants: Vec<Person>, // Converted into [Person!]!
}

# fn main() {}
```

Because `Person` is a valid GraphQL type, you can have a `Vec<Person>` in a
struct and it'll be automatically converted into a list of non-nullable `Person`
objects.

## Renaming fields

By default, struct fields are converted from Rust's standard `snake_case` naming
convention into GraphQL's `camelCase` convention:

```rust
# extern crate juniper;
# #[macro_use] extern crate juniper_codegen;
# 
#[derive(GraphQLObject)]
struct Person {
    first_name: String, // Would be exposed as firstName in the GraphQL schema
    last_name: String, // Exposed as lastName
}

# fn main() {}
```

You can override the name by using the `graphql` attribute on individual struct
fields:

```rust
# extern crate juniper;
# #[macro_use] extern crate juniper_codegen;
# 
#[derive(GraphQLObject)]
struct Person {
    name: String,
    age: i32,
    #[graphql(name="websiteURL")]
    website_url: Option<String>, // Now exposed as websiteURL in the schema
}

# fn main() {}
```

## Deprecating fields

To deprecate a field, you specify a deprecation reason using the `graphql`
attribute:

```rust
# extern crate juniper;
# #[macro_use] extern crate juniper_codegen;
# 
#[derive(GraphQLObject)]
struct Person {
    name: String,
    age: i32,
    #[graphql(deprecation="Please use the name field instead")]
    first_name: String,
}

# fn main() {}
```

The `name`, `description`, and `deprecation` arguments can of course be
combined. Some restrictions from the GraphQL spec stil applies though; you can
only deprecate object fields and enum values.