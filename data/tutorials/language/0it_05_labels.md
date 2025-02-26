---
id : labels
title: Labelled & Optional Arguments
description: >
  Provide labels to your functions arguments
category: "Introduction"
---

# Labelled and Optional Arguments to Functions

In this tutorial, we learn how to use labels in OCaml.

As a reminder: throughout this tutorial the code is written in the `ocaml` toplevel
and the command prompt appears as a `#`.

Also remember that an expression must end with `;;` for OCaml to evaluate it. 
Unless these examples start with a `#` toplevel prompt and end with `;;`, it isn't an 
expression to evaluate but rather an example of code structure.

## Labelled Arguments

Python has a nice syntax for writing arguments to functions. Here's
an example (from the Python tutorial, since I'm not a Python
programmer):

```python
def ask_ok(prompt, retries=4, complaint='Yes or no, please!'):
  # function definition omitted
```
Here are the ways we can call this Python function:

```python
ask_ok ('Do you really want to quit?')
ask_ok ('Overwrite the file?', 2)
ask_ok (prompt='Are you sure?')
ask_ok (complaint='Please answer yes or no!', prompt='Are you sure?')
```

Notice that in Python we are allowed to name arguments when we call
them, or use the usual function call syntax, and we can have optional
arguments with default values.

OCaml also has a way to label arguments and have optional arguments with
default values.

The basic syntax is:

```ocaml
# let rec range ~first:a ~last:b =
  if a > b then []
  else a :: range ~first:(a + 1) ~last:b;;
val range : first:int -> last:int -> int list = <fun>
```

(Notice that both `to` and `end` are reserved words in OCaml, so they
cannot be used as labels. This means you cannot have `~to` or
`~end`.)

The type of our previous `range` function was:

<!-- $MDX skip -->
```ocaml
range : int -> int -> int list
```

And the type of our new `range` function with labelled arguments is:

```ocaml
# range;;
- : first:int -> last:int -> int list = <fun>
```

Confusingly, the `~` (tilde) is *not* shown in the type definition, but
you need to use it everywhere else.

With labelled arguments, it doesn't matter which order you give the
arguments anymore:

```ocaml
# range ~first:1 ~last:10;;
- : int list = [1; 2; 3; 4; 5; 6; 7; 8; 9; 10]
# range ~last:10 ~first:1;;
- : int list = [1; 2; 3; 4; 5; 6; 7; 8; 9; 10]
```

There is also a shorthand way to name the arguments so that the label
matches the variable in the function definition:

```ocaml
# let may ~f x =
  match x with
  | None -> ()
  | Some x -> ignore (f x);;
val may : f:('a -> 'b) -> 'a option -> unit = <fun>
```

It's worth spending some time working out exactly what this function
does, and also working out its type signature by hand. There's a lot
going on. First of all, the parameter `~f` is just shorthand for `~f:f`
(i.e., the label is `~f` and the variable used in the function is `f`).
Secondly, notice that the function takes two parameters. The second
parameter (`x`) is unlabelled. It is permitted for a function to take a
mixture of labelled and unlabelled arguments.

What is the type of the labelled `f` parameter? Obviously it's a
function of some sort.

What is the type of the unlabelled `x` parameter? The `match` clause
gives us a clue. It's an `'a option`.

This tells us that `f` takes an `'a` parameter, and the return value of
`f` is ignored, so it could be anything. The type of `f` is therefore
`'a -> 'b`.

The `may` function as a whole returns `unit`. Notice in each case of the
`match` the result is `()`.

Thus the type of the `may` function is (you can verify this in the
OCaml interactive toplevel if you want):

```ocaml
# may;;
- : f:('a -> 'b) -> 'a option -> unit = <fun>
```
What does this function do? Running the function in the OCaml toplevel
gives us some clues:

```ocaml
# may ~f:print_endline None;;
- : unit = ()
# may ~f:print_endline (Some "hello");;
hello
- : unit = ()
```

If the unlabelled argument is a “null pointer” then `may` does nothing.
Otherwise, `may` calls the `f` function on the argument. 

Why is this
useful? We're just about to find out...

## Optional Arguments
Optional arguments are like labelled arguments, but we use `?` instead
of `~` in front of them. Here is an example:

```ocaml
# let rec range ?(step=1) a b =
  if a > b then []
  else a :: range ~step (a + step) b;;
val range : ?step:int -> int -> int -> int list = <fun>
```

Note the somewhat confusing syntax switching between `?` and `~`. We'll
talk about that in the next section, but here is how you call this function:

```ocaml
# range 1 10;;
- : int list = [1; 2; 3; 4; 5; 6; 7; 8; 9; 10]
# range 1 10 ~step:2;;
- : int list = [1; 3; 5; 7; 9]
```

In this case, `?(step=1)` means that `~step` is an
optional argument which defaults to 1. We can also omit the default
value and just have an optional argument. This example is modified from
LablGTK:

```ocaml
# type window =
  {mutable title: string;
   mutable width: int;
   mutable height: int};;
type window = {
  mutable title : string;
  mutable width : int;
  mutable height : int;
}
# let create_window () =
  {title = "none"; width = 640; height = 480;};;
val create_window : unit -> window = <fun>
# let set_title window title =
  window.title <- title;;
val set_title : window -> string -> unit = <fun>
# let set_width window width =
  window.width <- width;;
val set_width : window -> int -> unit = <fun>
# let set_height window height =
  window.height <- height;;
val set_height : window -> int -> unit = <fun>
# let open_window ?title ?width ?height () =
  let window = create_window () in
  may ~f:(set_title window) title;
  may ~f:(set_width window) width;
  may ~f:(set_height window) height;
  window;;
val open_window :
  ?title:string -> ?width:int -> ?height:int -> unit -> window = <fun>
```

This example is significantly complex and quite subtle, but the pattern
used is very common in the LablGTK source code. Let's concentrate on the
simple `create_window` function first. This function takes a `unit` and
returns a `window` initialised with default settings for title, width,
and height:

```ocaml
# create_window ();;
- : window = {title = "none"; width = 640; height = 480}
```

The `set_title`, `set_width`, and `set_height` functions are impure
functions which modify the `window` structure. For
example:

```ocaml
# let w = create_window () in
  set_title w "My Application";
  w;;
- : window = {title = "My Application"; width = 640; height = 480}
```

So far, this is just the imperative "mutable records" that we talked
about in ["If Statements, Loops, and Recursions"](https://ocaml.org/docs/if-statements-and-loops). Now the complex part is the `open_window`
function. This function takes *4* arguments, three of them optional,
followed by a required, unlabelled `unit`. Let's first see this function
in action:

```ocaml
# open_window ~title:"My Application" ();;
- : window = {title = "My Application"; width = 640; height = 480}
# let window = open_window ~title:"Clock" ~width:128 ~height:128 ();;
- : window = {title = "Clock"; width = 128; height = 128}
```

It does what you expect, but how? The secret is in the `may` function
(see above) and the fact that the optional parameters *don't* have
defaults.

When an optional parameter doesn't have a default, then it has type
`'a option`. The `'a` would normally be inferred by type inference, so
in the case of `?title` above, this has type `string option`.

Remember the `may` function? It takes a function and an argument, and it
calls the function on the argument, provided that the argument isn't `None`.
So:

<!-- $MDX skip -->
```ocaml
# may ~f:(set_title window) title;;
```

If the optional title argument is not specified by the caller, then
`title` = `None`, so `may` does nothing. But if we call the function
with, for example:

```ocaml
# open_window ~title:"My Application" ();;
- : window = {title = "My Application"; width = 640; height = 480}
```

then `title` = `Some "My Application"`, and `may` therefore calls
`set_title window "My Application"`.

You should make sure you fully understand this example before proceeding
to the next section.

## `Warning: This optional argument cannot be erased`
We've just touched upon labels and optional arguments, but even this
brief explanation might have raised several questions. The first may be: 
"Why use the extra `()` (unit) argument to `open_window`?" Let's try defining this
function without the extra `unit`:

```ocaml
# let open_window ?title ?width ?height =
  let window = create_window () in
  may ~f:(set_title window) title;
  may ~f:(set_width window) width;
  may ~f:(set_height window) height;
  window;;
Warning 16 [unerasable-optional-argument]: this optional argument cannot be erased.
Warning 16 [unerasable-optional-argument]: this optional argument cannot be erased.
Warning 16 [unerasable-optional-argument]: this optional argument cannot be erased.
val open_window : ?title:string -> ?width:int -> ?height:int -> window =
  <fun>
```

Although OCaml has compiled the function, it has generated a somewhat
infamous warning: "This optional argument cannot be erased," referring
to the final `?height` argument. To try to show what's going on here,
let's call our modified `open_window` function:

```ocaml
# open_window;;
- : ?title:string -> ?width:int -> ?height:int -> window = <fun>
# open_window ~title:"My Application";;
- : ?width:int -> ?height:int -> window = <fun>
```

That didn't work. In fact, it didn't even run the
`open_window` function at all. Instead it printed some strange type
information. 

Let's examine why:

Recall currying, uncurrying, and partial application of functions. Let's say
we have a function `plus` defined as:

```ocaml
# let plus x y =
  x + y;;
val plus : int -> int -> int = <fun>
```
We can partially apply this as `plus 2`, for example, which is "the
function that adds 2 to things":

```ocaml
# let f = plus 2;;
val f : int -> int = <fun>
# f 5;;
- : int = 7
# f 100;;
- : int = 102
```

In the `plus` example, the OCaml compiler can easily work out that
`plus 2` doesn't have enough arguments supplied yet. It needs another
argument before the `plus` function itself can be executed. Therefore
`plus 2` is a function which is waiting for its extra argument to come
along.

Things are not so clear when we add optional arguments into the mix. The
call to `open_window;;` above is a case in point. Does the user mean
"execute `open_window` now, or does the user mean to supply some or all
of the optional arguments later? Is `open_window;;` waiting for extra
arguments to come along like `plus 2`?

OCaml plays it safe and doesn't execute `open_window`. Instead, it treats
it as a partial function application. The expression `open_window`
literally evaluates to a function value.

Let's go back to the original working definition of `open_window`,
where we had the extra unlabelled `unit` argument at the end:

```ocaml
# let open_window ?title ?width ?height () =
  let window = create_window () in
  may ~f:(set_title window) title;
  may ~f:(set_width window) width;
  may ~f:(set_height window) height;
  window;;
val open_window :
  ?title:string -> ?width:int -> ?height:int -> unit -> window = <fun>
```

If you want to pass optional arguments to `open_window` you must do so
before the final `unit`, so if you type:

```ocaml
# open_window ();;
- : window = {title = "none"; width = 640; height = 480}
```
you must mean "execute `open_window` now with all optional arguments
unspecified". Whereas if you type:

```ocaml
# open_window;;
- : ?title:string -> ?width:int -> ?height:int -> unit -> window = <fun>
```
you mean "give me the functional value" or (more usually in the
toplevel) "print out the type of `open_window`".

## More `~`shorthand
Let's rewrite the `range` function yet again, this time using as much
shorthand as possible for the labels:

```ocaml
# let rec range ~first ~last =
  if first > last then []
  else first :: range ~first:(first + 1) ~last;;
val range : first:int -> last:int -> int list = <fun>
```

Recall that `~foo` on its own is short for `~foo:foo`. This applies also
when calling functions as well as declaring the arguments to functions.
In the above the `~last` is short for
`~last:last`.

## Using `?foo` in a Function Call
There's another little wrinkle concerning optional arguments. Suppose we
write a function around `open_window` to open up an application:

```ocaml
# let open_application ?width ?height () =
  open_window ~title:"My Application" ~width ~height;;
Error: This expression has type 'a option
       but an expression was expected of type int
```

Recall that `~width` is shorthand for `~width:width`. The type of
`width` is `'a option`, but `open_window ~width:` expects an `int`.

OCaml provides more syntactic sugar. Writing `?width` in the function
call is shorthand for writing `~width:(unwrap width)`, where `unwrap`
would be a function which removes the "`option` wrapper" around
`width` (it's not actually possible to write an `unwrap` function like
this, but conceptually that's the idea). So the correct way to write
this function is:

```ocaml
# let open_application ?width ?height () =
  open_window ~title:"My Application" ?width ?height;;
val open_application : ?width:int -> ?height:int -> unit -> unit -> window =
  <fun>
```

## When and When Not to Use `~` and `?`
The syntax for labels and optional arguments is confusing, and you may
often wonder when to use `~foo`, when to use `?foo`, and when to use
plain `foo`. It's something of a black art that takes practice to get
right.

`?foo` is only used when declaring the arguments of a function, ie:

<!-- $MDX skip -->
```ocaml
let f ?arg1 ... =
```

or when using the specialised "unwrap `option` wrapper" form for
function calls:

```ocaml
# let open_application ?width ?height () =
  open_window ~title:"My Application" ?width ?height;;
val open_application : ?width:int -> ?height:int -> unit -> unit -> window =
  <fun>
```
The declaration `?foo` creates a variable called `foo`, so if you need
the value of `?foo`, use just `foo`.

The same applies to labels. Only use the `~foo` form when declaring
arguments of a function, i.e.:

<!-- $MDX skip -->
```ocaml
let f ~foo:foo ... =
```

The declaration `~foo:foo` creates a variable called simply `foo`, so if
you need the value just use plain `foo`.

However, things get complicated for two reasons: first, the shorthand
form `~foo` (equivalent to `~foo:foo`), and second, when you call a
function that takes a labelled or optional argument using the
shorthand form.

Here is some apparently obscure code from LablGTK to demonstrate all of
this:

<!-- $MDX skip -->
```ocaml
# let html ?border_width ?width ?height ?packing ?show () =  (* line 1 *)
  let w = create () in
  load_empty w;
  Container.set w ?border_width ?width ?height;            (* line 4 *)
  pack_return (new html w) ~packing ~show                  (* line 5 *);;
```
On line 1, we have the function definition. Notice there are 5 optional
arguments and the mandatory `unit` 6<sup>th</sup> argument. Each of the
optional arguments is going to define a variable, e.g., `border_width` of
type `'a option`.

On line 4, we use the special `?foo` form for passing optional arguments
to functions that take optional arguments. `Container.set` has the
following type:

<!-- $MDX skip -->
```ocaml
module Container = struct
  let set ?border_width ?(width = -2) ?(height = -2) w =
    (* ... *)
```
Line 5 uses the `~`shorthand. Let's write this in long form:

```ocaml
# pack_return (new html w) ~packing:packing ~show:show;;
Line 1, characters 1-12:
Error: Unbound value pack_return
```

The `pack_return` function actually takes mandatory labelled arguments
called `~packing` and `~show`, each of type `'a option`. In other words,
`pack_return` explicitly unwraps the `option` wrapper.
