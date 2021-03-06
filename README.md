# purescript-untagged-union

A data type for untagged unions.

# Overview

Consider a the following type:

```purescript
import Untagged.Union

type ISB = Int |+| String |+| Boolean
```

The type `ISB` describes values which can either be an `Int`, a `String` or a `Boolean`. Moreover, it is guaranteed that at runtime, values of type `ISB` is directly stored as an `Int`, a `String` or a `Boolean` without any wrappers. This makes it especially useful for FFI.

Note that `|+|` is an alias for `OneOf`.

## Creating a `OneOf`

In order to create a value of `OneOf`, use `asOneOf`.

```purescript

isb1 :: ISB
isb1 = asOneOf 20

isb2 :: ISB
isb2 = asOneOf "foo"


--isb3 :: ISB
--isb3 = asOneOf 3.5

-- isb3 will fail since 3.5 is a Number which is neither
-- an Int, String nor Boolean

```

## Usage with Undefined

The library also defines `Undefined`. Combined with `OneOf`, it can represent an optional type:

```purescript
import Literals.Undefined

type OptionalInt = Undefined |+| Int
```

An alias `UndefinedOr` is also provided.

```purescript
type OptionalInt' = UndefinedOr Int -- Same as `Undefined |+| Int`
```

## Getting a value

To get a purescript value back, use `fromOneOf`:

```purescript
import Data.Maybe

valInt :: Maybe Int
valInt = fromOneOf isb1 -- evaluates to `Just 20`

valString :: Maybe String
valString = fromOneOf isb1 -- evalutes to `Nothing` since isb1 contains an Int

```

## Usage with records

Using `OneOf`, describing JS types as records becomes intuitive:

```purescript
type Props =
  { text :: String -- a required field

  -- Optional Fields
  , width :: UndefinedOr Number
  , height :: UndefinedOr Number

  -- Optional and Varying types
  , fontSize :: Undefined |+| String |+| Number
  }
```

A `cast` helper is made available to convert records with the same runtime value:

```purescript
import Untagged.Castable (cast)

sampleProps :: Props
sampleProps =
  cast { text: "foo" -- text is required and should be a string

       , width: 30.0 -- width is optional, and may be defined, but should be a Number
       -- height is optional and may be omitted
       , fontSize: "100%" -- fontSize may be defined, and should either be a string or number
       }

```

## Nested Unions

Nesting is possible but requires some hints for the compiler.

Consider the following:

```purescript
type Props =
  { text :: String -- a required field
  , width :: UndefinedOr Number
  , height :: UndefinedOr Number
  , fontSize :: Undefined |+| String |+| Number
  }


type ContainerProps =
  { titleProps :: UndefinedOr Props
  , opacity :: UndefinedOr Number
  }
```

In order to create an instance of `ContainerProps`, the type of titleProps must be specified.

```purescript
sampleContainerProps :: ContainerProps
sampleContainerProps =
  cast { titleProps: (cast { text: "Foo" } :: Props)
       }
```

Alternatively, helper constructors may be used to aid type inference.

```purescript
import Untagged.Castable (Castable)

props :: forall r. Castable r Props => r -> Props
props = cast

containerProps :: forall r. Castable r ContainerProps => r -> ContainerProps
containerProps = cast

sampleContainerProps' :: ContainerProps
sampleContainerProps' =
  containerProps { titleProps: props { text: "Foo" }
                 }
```
