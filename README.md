# CI646: Lambda Calculus

In an earlier lecture we looked at the lambda calculus (λC), a simple formalism
that provides a model for functions and their application. In fact, all that the λC
consists of is functions and a way to apply arguments to functions. You may
want to look back over the slides for Week 1 before attempting this lab exercise,
and there is also a short refresher below.

This is an expression in the λC: `(λx.y x) y`. 

The part within brackets, `(λx.y x)`,
is a lambda binding (we know that because of the λ). In this expression, `x` is a *bound* variable because 
it appears in a lambda binding (`λx`). Because `y` does not appear in a lambda binding,
it is a *free* variable. The lambda expression requires a single argument, the `x`
referred to in the binding, and it will take whatever is to the right of it as that
argument; in this case, the outer `y`.

Supplying the argument, or reducing the expression, means stripping off the
binding, `λx`, and replacing all occurrences of `x` with the argument that was
supplied. But if we supply `y` as the argument to `(λx.y x)` we can see that there
might be some confusion, since there is already something called `y` inside it,
and these `y`s refer to different things. So we can rename variables before doing
anything else, in a process called α-conversion: `(λx.y x) z`.

This doesn’t make any difference since identifiers like `x`, `y` and `z` don’t have
any particular meaning; we just need to make sure we don’t get them mixed up
with each other.

Now, we can safely apply the argument. This is called β-reduction, and we
end up with: `z y`. This can’t be reduced any further, because there are no lambda bindings left.
The other type of reduction we can do is called η-reduction (η is pronounced
"eta") and means throwing away redundant arguments.

## Church encoding

We can use the λC to produce a working programming language, with numbers,
Boolean values, if statements, loops, lists and other data structures, and so on.
Given that λC is so simple, this has always struck me as pretty incredible!
A programming language built up from first principles in this way is called a
*Church encoding*. Our two ways of reducing expressions represent computation
("running" the "program") and a term that can’t be reduced any more is the
"result" of the "program". (An expression that can’t be reduced any more is said
to be in *normal form*.) 

The idea is to express the *behaviour* of PL constructs,
such as numbers or conditional statements etc, in the *structure* of λ-terms.

### Church Booleans

We can implement Boolean (true/false) logic by modelling *true* as the expression 
that takes two arguments and returns its first, and *false* as the expression
that takes two arguments and returns its second:

```
true ≡ λ x y . x
false ≡ λ x y . y
```

This might seem like an arbitrary setup -- why not the other way round? But Boolean logic is simply
a "world" in which there are exactly two possible values, and these expressions provide a
model of that.

We can use these Church Booleans to create an if statement. In any programming language
an "if" statement such as **if p then x else y** has three parts: `p`, an expression that can be 
evaluated to `true` or `false` (I've named it `p` because this is called the *predicate*), `x`, 
the "thing to do" (statement to evaluate) when `p` is `true`, and `y`, the statement to evaluate 
when `p` is false. 

So, to express these semantics, we need an expression that returns `x` if `p` is `true`, and `y` otherwise. 
The first argument, `p` is a λ expression which can be reduced to a Church Boolean. That is, its normal
form is one of `true` or `false` as defined above. The other expressions, `x`
and `y`, could be anything. Here’s our (surprisingly simple) definition:

```
if ≡ λ p x y . p x y
```

Make sure you understand this! With a pencil and paper, try substituting `true` and `false`
for `p` and some other λ expressions for `x` and `y` (or just the atoms `x` and `y`, as these are 
legitimate λ expressions) then reduce the expression as far as you can.

After you solve the problems below switch to the `solution` branch of this repository to see my
solutions.

1. Write a λ expression equivalent to logical negation, or `not`. That
is, `not true` returns `false` and `not false` returns `true`. The argument
to `not`, say `p`, can be reduced to a Church Boolean, so `not` is a λ function that takes one
argument (more properly, all λ functions take one argument and to represent functions that take more
than one argument we create functions
that return other functions expecting more arguments). The result of calling `not p` should be a Church Boolean. Make use of the fact that 
when `p` is equivalent to `true` (as defined above), it’s going to return its first argument,
otherwise it will return its second.

```
not ≡ λ b . b (λ x y . y) (λ x y . x)
```

So, if `b` ("boolean") is equivalent to `true` it will return its first argument, which is the expression for `false`, and vice versa.


2. Write a λ expression equivalent to logical conjunction, `and`. That
is, a λ expression that takes two expressions that can be reduced to Church Booleans and 
returns the Church Boolean `true` if and only if they are both equivalent to `true`. Otherwise, it 
returns `false`.

```
and ≡ λ x y . x y x
```

If `x` is equivalent to `true` it will return its first argument, `y`. So in this case the result of the whole thing is the same as `y`.
If `x` is `false`, the whole thing is `false` so we return `x`. 

3. Write a λ expression equivalent to logical disjunction, `or`. That
is, a λ expression that takes two expressions that can be reduced to Church Booleans and returns the
Church Boolean `true` if either of them is equivalent to `true`. Otherwise, it returns `false`.

```
or ≡ λ x y . x x y
```

If `x` is equivalent to `true` the whole thing is `true` so we return `x`. Otherwise the whole thing is `true` if and only if `y` is true, 
so we return `y`.


## A REPL for λC

*REPL* stands for *Read-Evaluate-Print-Loop* and is another name for an interpreter – something which allows 
us to type in commands and see the result. There is a fun little REPL available for working with λC at
https://repl.it/talk/share/Lambda-Calculus-Interpreter/15943, credit to the user of that site called ekzhang. 
Click on the *Run* button to start the interpreter then type in some λ
expressions to see them being reduced automatically. 

3. Type the following expression into the REPL and press enter: `(\ p. \x. \y. p x y ) (\ x. \y. x) z w`.
Note that the leftmost expression is the `if` statement from above and
its first argument is `true`. Was the result what you expected?

4. Enter your definitions of `not`, `and` and `or` from above as "macros" by entering
"NOT = ..." and so on. Now you can call your macros and reduce them
with suitable arguments to check that they do what you expect.

5. Finally, note that this REPL comes with many useful macros predefined.
Click on `program.txt` in the left sidebar to see the list of these. Look at the definition of
numbers, called *Church numerals*. A Church numeral is a λ expression that takes two arguments, say
`f` and `x`. The Church numeral `n` applies `f` to `x` `n` times. So 'zero' applies `f` to `x`
zero times, "one" does it once, and so on: `zero ≡ λf.λx.x`, `one ≡ λf.λx.f x`, `two ≡ λf.λx.f (f x)`, 
and so on. Church numerals are "natural" numbers, so there are no negative values -- one less than zero is just zero, as is `n-m`
where `m` is greater than `n`.

    Use the definitions of Church numerals and arithmetic here: https://en.wikipedia.org/wiki/Church_encoding to define
    the PRED (the predecessor of a numeral) and SUB (subtraction) macros.
