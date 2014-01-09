method-combinators
==================

tl;dr
---

This library gives you some handy function combinators you can use to make [Method Decorators] in CoffeeScript (click [here] for examples in JavaScript):

[here]: https://github.com/raganwald/method-combinators/blob/master/README-JS.md

[Method Decorators]: https://github.com/raganwald/homoiconic/blob/master/2012/08/method-decorators-and-combinators-in-coffeescript.md#method-combinators-in-coffeescript "Method Decorators in CoffeeScript"

```coffeescript
this.before =
  (decoration) ->
    (base) ->
      ->
        decoration.apply(this, arguments)
        base.apply(this, arguments)

this.after =
  (decoration) ->
    (base) ->
      ->
        decoration.call(this, __value__ = base.apply(this, arguments))
        __value__

this.around =
  (decoration) ->
    (base) ->
      (argv...) ->
        __value__ = undefined
        callback = =>
          __value__ = base.apply(this, argv)
        decoration.apply(this, [callback].concat(argv))
        __value__

this.provided =
  (condition) ->
    (base) ->
      ->
        if condition.apply(this, arguments)
          base.apply(this, arguments)

this.excepting =
  (condition) ->
    (base) ->
      ->
        unless condition.apply(this, arguments)
          base.apply(this, arguments)
```

The library is called "Method Combinators" because these functions are isomorphic to the combinators from Combinatorial Logic.

Back up the truck, Chuck. What's a Method Decorator?
---

A method decorator is a function that takes a function as its argument and returns a new function that is to be used as a method body. For example, this is a method decorator:

```coffeescript
mustBeLoggedIn = (methodBody) ->
                   ->
                     if currentUser?.isValid()
                       methodBody.apply(this, arguments)
```

You use it like this:

```coffeescript
class SomeControllerLikeThing

  showUserPreferences:
    mustBeLoggedIn ->
      #
      # ... show user preferences
      #
```

And now, whenever `showUserPreferences` is called, nothing happens unless `currentUser?.isValid()` is truthy. And you can reuse `mustBeLoggedIn` wherever you like. Since method decorators are based on function combinators, they compose very nicely, you can write:

```coffeescript
triggersMenuRedraw = (methodBody) ->
                       ->
                         __rval__ = methodBody.apply(this, arguments)
                        @trigger('menu:redraww')
                        __rval__

class AnotherControllerLikeThing

  updateUserPreferences:
    mustBeLoggedIn \
    triggersMenuRedraw \
    ->
      #
      # ... save updated user preferences
      #
```

Fine. Method Decorators look cool. So what's a Method Combinator?
---

Method combinators are convenient function combinators for making method decorators. When writing decorators, the same few patterns tend to crop up regularly:

1. You want to do something *before* the method's base logic is executed.
2. You want to do something *after* the method's base logic is executed.
3. You want to wrap some logic *around* the method's base logic.
4. You only want to execute the method's base logic *provided* some condition is truthy.

Method *combinators* make these common kinds of method decorators extremely easy to write. Instead of:

```coffeescript
mustBeLoggedIn = (methodBody) ->
                   ->
                     if currentUser?.isValid()
                       methodBody.apply(this, arguments)

triggersMenuRedraw = (methodBody) ->
                       ->
                         __rval__ = methodBody.apply(this, arguments)
                        @trigger('menu:redraww')
                        __rval__
```

We write:

```coffeescript
mustBeLoggedIn = provided -> currentUser?.isValid()

triggersMenuRedraw = after -> @trigger('menu:redraww')
```

And they work exactly as we expect:

```coffeescript
class AnotherControllerLikeThing

  updateUserPreferences:
    mustBeLoggedIn \
    triggersMenuRedraw \
    ->
      #
      # ... save updated user preferences
      #
```

The combinators do the rest!

Can I use this with Node's callback-oriented programming?
---

This library also provides [method combinators that work in an asynchronous world](https://github.com/raganwald/method-combinators/blob/master/doc/async.md#method-combinators-in-an-asynchronous-world).

So these are like RubyOnRails controller filters?
---

There are some differences. These are much simpler, which is in keeping with JavaScript's elegant style. For example, in Rails all of the filters can abort the filter chain by returning something falsy. The `before` and `after` decorators don't act as filters. Use `provided` if that's what you want.

More specifically:

* None of the decorators you build with the method combinators change the arguments passed to the method. The `before` and `around` callbacks can execute code before the method body is executed, but only for side-effects.
* The `provided` decorator will return `void 0` if it evaluates to falsy or return whatever the method body returns. There's no other way to change the return value with `provided`
* The `around` decorator will return `void 0` if you don't call the passed callback. Otherwise, it returns whatever the method body would return. You can't change its arguments or the return value. You don't need to pass arguments to the callback. If you do, they will be ignored.

Is it any good?
---

[Yes][y].

[y]: http://news.ycombinator.com/item?id=3067434

Can I install it with npm?
---

Yes: `npm install method-combinators`

Anything you left out?
---

Yes, there are some extra combinators that are useful for things like error handling and design-by-contract. Read [the source][c] for yourself. Or have a gander at these blog posts:

* [Method Combinators in CoffeeScript ](https://github.com/raganwald/homoiconic/blob/master/2012/08/method-decorators-and-combinators-in-coffeescript.md#method-combinators-in-coffeescript)
* [Using Method Decorators to Decouple Code](https://github.com/raganwald/homoiconic/blob/master/2012/08/decoupling_with_method_decorators.md#using-method-decorators-to-decouple-code)
* [Memoized, the *practical* method decorator](https://github.com/raganwald/homoiconic/blob/master/2012/09/memoize-the-practical-method-decorator.md#memoized-the-practical-method-decorator)
* [More Practical Method Combinators: Pre- and Post-conditions](https://github.com/raganwald/homoiconic/blob/master/2012/09/precondition-and-postcondition.md#more-practical-method-combinators-pre--and-post-conditions)

I'm writing a book called [CoffeeScript Ristretto](http://leanpub.com/coffeescript-ristretto). Check it out!

Et cetera
---

Method Combinators was created by [Reg "raganwald" Braithwaite][raganwald]. It is available under the terms of the [MIT License][lic]. The `retry` and condition combinators were inspired by Michael Fairley's Ruby [method_decorators](https://github.com/michaelfairley/method_decorators).

[raganwald]: http://braythwayt.com
[lic]: https://github.com/raganwald/method-combinators/blob/master/license.md

[js]: https://github.com/raganwald/method-combinators/blob/master/lib/method-combinators.js
[c]: https://github.com/raganwald/method-combinators/blob/master/lib/method-combinators.coffee
