# General code guidelines

- [General code guidelines](#general-code-guidelines)
  - [The wrong abstraction is much worse than copy-pasting](#the-wrong-abstraction-is-much-worse-than-copy-pasting)
    - [Copy-paste your first implementation if you need to](#copy-paste-your-first-implementation-if-you-need-to)
    - [External testimonials/opinions about abstractions](#external-testimonialsopinions-about-abstractions)
  - [Data is the ideal](#data-is-the-ideal)
    - [Data is the same when it looks the same](#data-is-the-same-when-it-looks-the-same)
    - [Data does not have behavior](#data-does-not-have-behavior)
      - [Shared behavior for a known set of types is just a function](#shared-behavior-for-a-known-set-of-types-is-just-a-function)
      - [Shared behavior for an unknown set of types](#shared-behavior-for-an-unknown-set-of-types)
    - [Avoid abstracting data when you can](#avoid-abstracting-data-when-you-can)
    - [External testimonials/opinions about data](#external-testimonialsopinions-about-data)
  - [Dependencies](#dependencies)
    - [Data dependencies](#data-dependencies)
      - [Constructors are king](#constructors-are-king)
      - [Your data's validity is only as good as the validity of what it contains](#your-datas-validity-is-only-as-good-as-the-validity-of-what-it-contains)
      - [A known and limited set of values in a type can often be hoisted to the type itself](#a-known-and-limited-set-of-values-in-a-type-can-often-be-hoisted-to-the-type-itself)

## The wrong abstraction is much worse than copy-pasting

Resist the urge to immediately get rid of copy-pasted code. It's unlikely that you will know for
sure that an abstraction fits for a given use case without allowing it to live in the code base for
a while.

### Copy-paste your first implementation if you need to

If your first implementation is a copy-pasted solution from elsewhere in the code base you will be
allowed time to change it without also modifying your abstraction in ways that it wasn't meant to
support.

### External testimonials/opinions about abstractions

See for example [this video](https://www.youtube.com/watch?v=sYZanbXKb3s) for an example of why this
principle is worth considering and adopting.

## Data is the ideal

Data is the most likely thing to endure in a code base because it is a shared, unambiguous and
definitive way of describing things, even across languages:

A JavaScript `object` is a Python `dict` which is a Haskell `Map k v` which is a... You get the
point. We can reason much the same about this thing in all of these languages and many more. You
don't generally have to explain this type of thing to someone who has already seen and understood it
in one language. This is also why you should understand your data structures and their
characteristics when you use them.

This extends also to user-defined types: "Structs" are named collections of known fields with
associated values. "(Tagged) Unions" are named collections of cases where each case can carry
different data with it. Most things fall into combinations of these two general concepts and if you
design with that in mind, your ideas are transferrable to most technologies. Some of them make using
these concepts much easier; it's advisable to use that fact.

### Data is the same when it looks the same

Despite the best efforts of language creators, many languages still confuse the place of data with
some supposed identity of that data. Data is equal to data that looks exactly like it. This concept
is transferrable across time and place; if the equality of data is the equality of its parts it is
also equal when it is read from disk, read from a websocket or from a database, be it tomorrow, next
week or next year. This does not hold true when you start involving the place (pointer/instance) of
data.

If you make this mistake in one place, any place that uses that place is also affected. If the
identity of a piece of data is based on a pointer or instance, anything that contains that data will
also abide by the same rules.

It's advisable to reflect on whether or not your favorite technology forces you into this all the
time and what that means for all of your data.

### Data does not have behavior

Avoid mixing data and behavior if you can. The most flexible way to work with data is to allow it
to just be data, and to then create behavior for it as functions in your code base.

#### Shared behavior for a known set of types is just a function

If there is a set of types/data that have some shared behavior and you control all instances of it,
all you need to do in order to have that shared behavior is to create a function that takes all of
those types and decides exactly what to do in each case. This keeps the behavior in one place that
is easy to get an overview of.

The most direct way to see this behavior through is to realize that if we have a known set of things
that have some characteristic and purpose, they are a union that itself is a type. That union is the
input type for our function that defines the behavior. The function looks at which case is active in
the union and executes the desired behavior.

- Elixir => Pattern matching / guards in function heads to determine case
- Haskell => You already have a union for the thing, that is your input
- TypeScript => Make an actual union where each case is distinguishable from the others
- ...

#### Shared behavior for an unknown set of types

Sometimes we do actually need or want extensible behavior. We have written functions that need a
certain concept to be applied to an input argument and we don't necessarily know the entire set of
inputs beforehand. For this, prefer to use constructs that allow you to codify the behavior and
rules as a set of functions:

- Elixir, Clojure => Protocols
- Haskell => Type Classes
- TypeScript => Methods in interfaces
- ...

### Avoid abstracting data when you can

It's tempting to abstract over data as you do with functions. As an example of this, TypeScript
allows you to create new types by merging already existing types as well as even doing type-level
logic to generate new types. This is very easy to misuse. Vastly prefer duplication over abstraction
when it comes to data. If a type does not have a required field/component you will simply not be
able to use it in the intended way (if you have static checks for this kind of thing), so the cost
of an error where you forgot to modify one of your cases simply does not exist in the same way as
for functions.

If you do want to share concepts between two types, the first stop should be a type that itself has
the relevant fields/components. With that in hand, create fields in the types that carry that data.
You don't need to use advanced type-level machinery to have the same kind of data in two places.

When we do want to do type-level logic for types, these should ideally not be exported; they should
be internal types used for additional typesafety in our plumbing.

### External testimonials/opinions about data

["The Value of Values" by Rich Hickey](https://www.youtube.com/watch?v=-6BsiVyC1kM) talks about many
of these points and more.

## Dependencies

Dependencies are important in programming. This section will not refer primarily to package
dependencies, as you might think, but rather data and logic dependencies.

### Data dependencies

#### Constructors are king

If I cannot have a thing of type `A` without also having a thing of type `B`, the most direct way to
accomplish this is to have the value of type `B` in the constructor of `A`. If I can create a `B` I
can then also create an `A`.

Consider a "click event type" event:

```typescript
interface ClickEvent {
  button: string;
}

function handleClick({button}: ClickEvent): void {
  if (clickEvent.button === "mouse1") {
    // What if this function can fail? The click event should never have been
    // valid in the first place.
    const coordinates = getMouseCoordinates();

    console.log(`User left clicked @ ${coordinates}`)
  }
  
  // ...
}
```

It's clear in this case that a click event in most cases depends on coordinates. We want to instead
say that in order to have a valid `ClickEvent` you should also have to have a set of `Coordinates`.

```typescript
interface ClickEvent {
  button: string;
  coordinates: Coordinates;
}

// We know here that we are passed a complete `ClickEvent` so we don't have to
// even consider the case where `getMouseCoordinates` has failed.
function handleClick({button, coordinates}: ClickEvent): void {
  if (button === "mouse1") {
    console.log(`User left clicked @ ${coordinates}`)
  }
  
  // If this function needed to be defined for every possible input value, which
  // values do we actually need to check here?
}
```

#### Your data's validity is only as good as the validity of what it contains

In our examples above our `button` field was of type `string`. This is a set of infinite values and
we would require logic to actually check the data. It can be more useful to limit your type instead
to a known set of values by way of a union type.

```typescript
type ClickButton = "mouse1" | "mouse2" | "mouse3";

interface ClickEvent {
  button: ClickButton;
  coordinates: Coordinates;
}

function handleClick({button, coordinates}: ClickEvent): void {
  // We can be confident that we have handled all possible cases of the click
  // type here because we have made it so that you can only pass one of three
  // different values in the constructor.
  if (button === "mouse1") {
    console.log(`User left clicked @ ${coordinates}`)
  } else if (button === "mouse2") {
    console.log(`User right clicked @ ${coordinates}`)
  } else if (button === "mouse3") {
    console.log(`User middle clicked @ ${coordinates}`)
  } else {
    // This is TypeScript specific and makes it so that we are always forced to
    // handle all cases. This is very useful when one adds a case to
    // `ClickButton`, because this code would then not compile and we would be
    // reminded that we are expected to handle all cases of this union here.
    return assertUnreachable(button);
  }
}
```

This concept applies in any language that allows you to statically determine that a value is of a
given type, without having to deal with implicit type conversions that may give erroneous values. In
Haskell this end result would look as follows:

```haskell
data ClickButton = Mouse1 | Mouse2 | Mouse3

data ClickEvent = ClickEvent
  { button :: ClickButton,
    coordinates :: Coordinates
  }

handleClick :: ClickEvent -> IO ()
handleClick ClickEvent {button = Mouse1, coordinates} =
  putStrLn ("User left clicked @ " <> show coordinates)
handleClick ClickEvent {button = Mouse2, coordinates} =
  putStrLn ("User right clicked @ " <> show coordinates)
handleClick ClickEvent {button = Mouse3, coordinates} =
  putStrLn ("User middle clicked @ " <> show coordinates)
```

#### A known and limited set of values in a type can often be hoisted to the type itself

Our `ClickEvent` contains the union of our different click buttons. This information could be put in
the constructors of the click event by turning `ClickEvent` itself into a union:

```haskell
data ClickEvent
  = Mouse1Click Coordinates
  | Mouse2Click Coordinates
  | Mouse3Click Coordinates

handleClick :: ClickEvent -> IO ()
handleClick (Mouse1Click coordinates) =
  putStrLn ("User left clicked @ " <> show coordinates)
handleClick (Mouse2Click coordinates) =
  putStrLn ("User right clicked @ " <> show coordinates)
handleClick (Mouse3Click coordinates) =
  putStrLn ("User middle clicked @ " <> show coordinates)
```

We've now made it so that each event can carry different data, as they are not merely part of the
same structure where all pieces of the data have to be present. Our data dependencies for the
different click types can potentially change and differ from each other. Let's say, for example,
that I wanted left clicks to also be able to take a modifier key into consideration:

```haskell
data ClickEvent
  = Mouse1Click Coordinates ModifierKey
  | Mouse2Click Coordinates
  | Mouse3Click Coordinates

handleClick :: ClickEvent -> IO ()
handleClick (Mouse1Click coordinates modifierKey) =
  putStrLn
    ( "User left clicked @ " <> show coordinates
        <> " with modifier key "
        <> show modifierKey
    )
handleClick (Mouse2Click coordinates) =
  putStrLn ("User right clicked @ " <> show coordinates)
handleClick (Mouse3Click coordinates) =
  putStrLn ("User middle clicked @ " <> show coordinates)
```

It also opens up for expressing more things without having to ignore parts of our data in many
cases:

```haskell
data ClickEvent
  = Mouse1Click Coordinates ModifierKey
  | Mouse2Click Coordinates
  | Mouse3Click Coordinates
  | Mouse1Drag StartCoordinates StopCoordinates

handleClick :: ClickEvent -> IO ()
handleClick (Mouse1Click coordinates modifierKey) =
  putStrLn
    ( "User left clicked @ " <> show coordinates
        <> " with modifier key "
        <> show modifierKey
    )
handleClick (Mouse2Click coordinates) =
  putStrLn ("User right clicked @ " <> show coordinates)
handleClick (Mouse3Click coordinates) =
  putStrLn ("User middle clicked @ " <> show coordinates)
handleClick (Mouse1Drag (StartCoordinates start) (StopCoordinates stop)) =
  putStrLn
    ( "User dragged the left button from " <> show start
        <> " to "
        <> show stop
    )
```

None of these additions need to affect the other, distinct cases and making this a union we could
extend freely allowed us to add events that have more or different data than our previous
definition.
