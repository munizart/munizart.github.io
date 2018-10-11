----
-layout: post
-title: Introducing IdleComp. Composing even when urgent
----

### What this post is about?
Few weeks ago, Philip Walton from google, posted an article named
"[Idle Until Urgent](https://philipwalton.com/articles/idle-until-urgent/)" showing how
First Input Delay (FID) impacts user's perception on web perfomance.

On this blog post I'll cover:
* What is `requestIdleCallback`
* Idle Until Urgent philosophy
* Non-blocking code
* and finally, Composition

### `requestIdleCallback`

On his article Walton introduces us to `requestIdleCallback`,
a simple WebAPI made of just two functions (`requestIdleCallback` and `cancelIdleCallback`).

But, how does it works?
`requestIdleCallback` takes one argument, a function, and returns a numeric id.
It behaves much like `setTimeout`, but instead of calling your function after a
given time it will call your function when the browser is doing no work.
That means that browser's takes can take priority over your's.

```javascript
requestIdleCallback(function () {
  alert ('Browser is idle now')
})
```
<small>`requestIdleCallback` example</small>

But dealing with this kind of asynchronism is a little bit tricky.

Let me show you a example.
Imagine that you decided to optimize your site to only execute
your `registerModalComponent` task when it has nothing else to do.

```javascript
function registerComponents () {
  registerOtherComponents()
  requestIdleCallback(registerModalComponent)
}
```

You run the firsts tests and everything is working like a charm.
Few months later, your client calls you and say something like

> Hi, the modals aren't working anymore,
everytime I reload the page and click at the "show modal" botton nothing happens.

So you start to debug the site and realise that a exception is thrown
when you try to show a modal before `registerModalComponent` is called.
Now you should figure out a way to register the modal component
sinchronously if it hasn't being registred yet. This is what Walton called "Idle Until Urgent"
as `registerModalComponent` is proccessed on idle time, unless you need it urgently.

```javascript
let idleCallbackId
let registredYet

function registerComponents () {
  registerOtherComponents()
  idleCallbackId = requestIdleCallback(() => {
    registerModalComponent()
    registredYet = true
  })
}

function enforceModalComponent () {
  if (!registredYet) {
    cancelIdleCallback(idleCallbackId)
    registerModalComponent()
    registredYet = true
  }
}
```


Ok, so now you have a function named `enforceModalComponent`,
and your `registerComponents` is now impure.
Worst yet, you need to call `enforceModalComponent` every time
you want to use a modal component =[

Now, imagine that you need to optimize more components,
imagine having one `idleCallbackId` and one `registredYet` like
variables for each component. It simplily won't scale.

### Entering the IdleComp

Now, what if instead of having all this state to manage you could simply have
a data type that look like a promise,
something you're familiar with, to help you deal with all this asynchronism?

This exactly what [IdleComp](https://github.com/munizart/idle-comp) does,
allowing you to write that reads like sinchronous, but computes on browser's idle time until you need it.

```javascript
import IdleComp from 'idle-comp'

const idleModal = IdleComp
  .of(undefined) // initial value, in this example anything will do, so we pass nothing
  .map(registerModalComponent)
```

So every time you want to show a modal the only thing you need to do is call `idleModal.returns()`
and if `registerModalComponent` wasn't called yet it will be called sinchronously.
This means that `idleModal.returns()` will make our task urgent.

### Just like a Promise

Remember I said that `IdleComp` is something you're already familiar with? Let me show it to you.

```
Promise.resolve(10)
  .then(ten => add2(ten))
  .then(twelve => divedBy3(twelve))
  .then(four => square(four))
  .then(console.log) // logs 16
```

```
IdleComp.of(10)
  .map(ten => add2(ten))
  .map(twelve => divideBy3(twlve))
  .map(four => square(four))
  .map(console.log) // logs 16
```

As you see, `IdleComp#map` is just like `Promise#then` and both `of` and `resolves`
will put a value inside a data-type structure. This may seems like a considence,
but actually occours as both `IdleComp` and `Promises` are Functors,
a algebric data type that allows you while wrapping a value abstract a behaviour
(future for `Promise`, idle computation for `IdleComp`) and transform the value step by step (composition).

### Non-blocking code

In regular javascript there is one thread where all code are executed and if
your functions takes to long to execute it will block this thread.
Actually, the problem gets even worst as not only user land code is ran in this thread!
Some browser code will be competing with yours.

Generally, the long a funciton is the long a functions takes to execute.
So if break your code into small functions and compose they togheter and take advantage of some
asynchronism you won't be blocking the main thread for long.

### Composition
A nice thing about `IdleComp` is that `IdleComp.of(10).map(add2).map(divideBy3)`
is equivalent to `IdleComp.of(10).map(x => divedBy3(add2))`.
This is composition, an operation over two functions that gives you a new one that
takes the results of the first functions and passes it as a argument to next one.

You can learn more about composition and functional programming on this [wonderfull
article from Eric Elliot](https://medium.com/javascript-scene/composing-software-an-introduction-27b72500d6ea).

### Wrapping up
We learned about `requestIdleCallback`, a WebAPI that gives you access to browsers
idle time using two simple functions.

Next, we needed a way to cancel our idle callback and compute things sinchronously
on user demand and we shaw how bad it can goes if we don't abstract `requestIdleCallback` and
`cancelIdleCallback`.

We also learned about `IdleComp`, a promise-like abstraction for idle computation
that permits us to write idle computations that doesn't block the browser's main
thread by composing small functions togheter.

### Next time
On the next post we will see another `IdleComp`'s method: `flatMap`.

Stay tuned,
Tank you for reading.
=]
