# Simple examples for Elixir macro usage

Via `quote` the abstract syntax tree (AST) for an Elixir expression can be preserved instead of evaluating the expression.

An AST is either a a leaf node being one of `nil`, `false`, `true`, `:atom`, `"string"`, Number (eg. `42`), List (`[...]`) or two-element tuple `{a, b}` or an inner node being encoded as three-element `{fun, context, args}` representing a function application.

```ex
quote do
  nil
end #=> nil

quote do
  false
end #=> false

quote do
  true
end #=> true

quote do
  :myAtom
end #=> :myAtom

quote do
  String
end #=> {:__aliases__, [alias: false], [:String]}

quote do
  Kernel
end #=> {:__aliases__, [alias: false], [:Kernel]}

quote do
  "Hello"
end #=> "Hello"

quote do
  5.5
end #=> 5.5

quote do
  5
end #=> 5

quote do
  []
end #=> []

quote do
  [1, 2, 3]
end #=> [1, 2, 3]

quote do
  [foo: 5, bar: 6]
end #=> [foo: 5, bar: 6]

quote do
  [foo: 5, bar: 6]
end #=> [foo: 5, bar: 6]

quote do
  %{}
end #=> {:%{}, [], []}

quote do
  %{x: 5}
end #=> {:%{}, [], [x: 5]}

quote do
  ~c"Hello"
end #=> {:sigil_c, [delimiter: "\"", context: Elixir, imports: [{2, Kernel}]],
 [{:<<>>, [], ["Hello"]}, []]}

quote do
  !true
end #=> {:!, [context: Elixir, imports: [{1, Kernel}]], [true]}

quote do
  if x do
    y
  end
end #=> {:if, [context: Elixir, imports: [{2, Kernel}]],
 [{:x, [], Elixir}, [do: {:y, [], Elixir}]]}

quote do
  if x do
    y
  else
    z
  end
end #=> {:if, [context: Elixir, imports: [{2, Kernel}]],
 [{:x, [], Elixir}, [do: {:y, [], Elixir}, else: {:z, [], Elixir}]]}

quote do
  for x <- y, if: x > 5 do
    x
  end
end #=> {:for, [],
 [
   {:<-, [], [{:x, [], Elixir}, {:y, [], Elixir}]},
   [if: {:>, [context: Elixir, imports: [{2, Kernel}]], [{:x, [], Elixir}, 5]}],
   [do: {:x, [], Elixir}]
 ]}

quote do
  for x <- y, reduce: 0 do
    acc -> acc + x
  end
end #=> {:for, [],
 [
   {:<-, [], [{:x, [], Elixir}, {:y, [], Elixir}]},
   [reduce: 0],
   [
     do: [
       {:->, [],
        [
          [{:acc, [], Elixir}],
          {:+, [context: Elixir, imports: [{1, Kernel}, {2, Hello}]],
           [{:acc, [], Elixir}, {:x, [], Elixir}]}
        ]}
     ]
   ]
 ]}

quote do
  def a(b) do
    b
  end
end #=> {:def, [context: Elixir, imports: [{1, Kernel}, {2, Kernel}]],
 [{:a, [context: Elixir], [{:b, [], Elixir}]}, [do: {:b, [], Elixir}]]}

quote do
  fn x -> x end
end #=> {:fn, [], [{:->, [], [[{:x, [], Elixir}], {:x, [], Elixir}]}]}

quote do
  fn
    0 -> 0
    1 -> 2
    x -> x
    _ -> y
  end
end #=> {:fn, [],
 [
   {:->, [], [[0], 0]},
   {:->, [], [[1], 2]},
   {:->, [], [[{:x, [], Elixir}], {:x, [], Elixir}]},
   {:->, [], [[{:_, [], Elixir}], {:y, [], Elixir}]}
 ]}

quote do
  quote do
    x
  end
end #=> {:quote, [context: Elixir], [[do: {:x, [], Elixir}]]}

quote do
  defmodule Hello do
    @foo
    @bar 1
  end
end #=> {:defmodule, [context: Elixir, imports: [{2, Kernel}]],
 [
   {:__aliases__, [alias: false], [:Hello]},
   [
     do: {:__block__, [],
      [
        {:@, [context: Elixir, imports: [{1, Kernel}]],
         [{:foo, [context: Elixir], Elixir}]},
        {:@, [context: Elixir, imports: [{1, Kernel}]],
         [{:bar, [context: Elixir], [1]}]}
      ]}
   ]
 ]}

quote do
  1
  2
  3
end #=> {:__block__, [], [1, 2, 3]}

quote unquote: false do
  1
  unquote(2)
  3
end #=> {:__block__, [], [1, {:unquote, [], [2]}, 3]}

quote unquote: false do
  {:foo, 5}
  {:bar, 5}
  {:baz, 5}
end #=> {:__block__, [], [foo: 5, bar: 5, baz: 5]}

quote line: 10, generated: true do
  x
end #=> {:x, [generated: true, line: 10], Elixir}

quote line: 10, file: "custom.ex", generated: true do
  x
end #=> {:x, [generated: true, keep: {"custom.ex", 10}], Elixir}
```
