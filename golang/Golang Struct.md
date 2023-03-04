## What is a Struct?

A struct is a user-defined data type in Golang that allows you to group together values of different types under a single name. It's like a class in object-oriented programming languages. You can define a struct by specifying a name and the types of its fields.

## Defining a Struct

To define a struct in Golang, we use the `type` keyword followed by the name of the struct and the fields it contains. For example, to define a struct called `Person` with fields for the person's name and age, we can do the following:

```
type Person struct {
    Name string
    Age int
}
```

In this example, we're defining a struct called `Person` with two fields: `Name` of type string and `Age` of type int.

## Creating a Struct

To create a struct in Golang, we use the `Person` struct we defined earlier and create a new instance of it using the `Person{}` syntax. We can then assign values to the fields of the struct using dot notation. For example:

```
person := Person{
    Name: "John",
    Age: 30,
}

```

In this example, we're creating a new instance of the `Person` struct called `person` with the `Name` field set to "John" and the `Age` field set to 30.

## Accessing Struct Fields

To access the fields of a struct, we use dot notation. For example:

```
fmt.Println(person.Name) // Output: John
fmt.Println(person.Age) // Output: 30

```

In this example, we're accessing the `Name` and `Age` fields of the `person` struct using dot notation.

## Modifying Struct Fields

To modify the fields of a struct, we use dot notation and assign new values to them. For example:

```
person.Name = "Jane"
person.Age = 25
fmt.Println(person) // Output: {Jane 25}

```

In this example, we're modifying the `Name` and `Age` fields of the `person` struct using dot notation.

## Struct Embedding

Struct embedding allows you to include one struct as a field of another struct. This is useful when you want to reuse the fields and methods of an existing struct in a new struct. For example:

```
type Address struct {
    Street string
    City string
    State string
}

type Person struct {
    Name string
    Age int
    Address
}

```

In this example, we're defining a new `Address` struct with fields for the person's street address, city, and state. We're then defining the `Person` struct to embed the `Address` struct, which allows us to reuse its fields in the `Person` struct.

## Conclusion

Structs are a powerful tool in Golang that allow you to group together values of different types under a single name. They can be used to represent complex data structures and are often used to define the data model for a program. With the knowledge you gained in this article, you should be able to define, create, and modify structs, as well as understand how to use struct embedding.