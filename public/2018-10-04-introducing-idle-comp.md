----
-layout: post
-title: Introducing IdleComp
----

Few weeks ago, Philip Walton from google, posted an article named
"[Idle Until Urgent](https://philipwalton.com/articles/idle-until-urgent/)" showing how
First Input Delay (FID) impacts user's perception on web perfomance. On this article Philip introduces us
to `requestIdleCallback`, a simple WebAPI made of just two functions (`requestIdleCallback` and `cancelIdleCallback`).

But, how does it works?
`requestIdleCallback` takes one argument, a function, and returns a numeric id. It behaves much like `setTimeout`,
but instead of calling your function after a given time it will call your function when the browser is doing no work.
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

> Hi, the modals aren't working anymore, everytime I reload the page and click at the "show modal" botton nothing happens.

So you start to debug the site and realise that a exception is thrown when you try to show a modal before `registerModalComponent` is called.
Now you should figure out a way to register the modal component sinchronously if it hasn't being registred yet.

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

Now, imagine that you need to optimize more components, imagine having one `idleCallbackId` and one `registredYet` like variables for each component. It simplily won't scale.

### Entering the IdleComp

Now, what if instead of having all this state to manage you could simply have a data type that look like a promise,
something you're familiar with, to help you deal with all this asynchronism?

```javascript
import IdleComp from 'idle-comp'

const idleModal = IdleComp
  .of(undefined) // initial value, in this example anything will do, so we pass nothing
  .map(registerModalComponent)
```

So every time you want to show a modal the only thing you need to do is call `idleModal.returns()`. And if `registerModalComponent` wasn't called yet it will be called sinchronously.
