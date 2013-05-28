> "Those who are unaware they are walking in darkness
will never seek the light."

~ Bruce Lee

-------------------------------------

# Classical Inheritance is Obsolete:

How to Think in Prototypal OO

_Eric Elliott_

-------------------------------------

![Adobe Logo](adobe.jpeg)

![Tout](tout.jpg) ![BandPage](bandpage.jpg) ![Zumba Fitness](zumba.jpg)

-------------------------------------

!["Programming JavaScript Applications"](pja.jpg)

-------------------------------------

## "In the old days..."

-------------------------------------

## "Java Will Rule The World"

-------------------------------------

## "JavaScript is Just a Toy"

-------------------------------------

## "You Specialize in JavaScript?"

-------------------------------------

![Facepalm](bear-facepalm.jpg)

-------------------------------------

## "JavaScript isn't Object Oriented!"

-------------------------------------

## It's easy to mimic classes in JS

... but no class imitation has ever become a defacto standard.

Why is that?

-------------------------------------

## Inheritance Hierarchies are Trouble

-------------------------------------

## Inflexible hierarchies

* Eventually, all single-parent inheritance hierarchies are wrong.

-------------------------------------

## Items

* Tools
* Weapons

-------------------------------------

## No clue!

![Colonel Mustard in the billiard room with a wrench.](clue.jpg)

-------------------------------------

## Tight coupling

-------------------------------------

## Brittle architecture

* Tight coupling and single-parent hierarchies make structures rigid and changes hard.

-------------------------------------

## Duplication by necessity

* Many slightly different versions of the same class.

-------------------------------------

## The Gorilla / Banana Problem

> "The problem with object-oriented languages is they've got all this implicit environment that they carry around with them. You wanted a banana, but what you got was a gorilla holding the banana and the entire jungle."

~ Joe Armstrong, "Coders at Work"

-------------------------------------

## JavaScript naturally handles OO with none of these problems

-------------------------------------

## There's no Class in jQuery

* But there is `$.extend()`

-------------------------------------

## JavaScript's Broken Backbone

* Wonderful separation of concerns
* Elegant and minimal
* Painful to detangle when Backbone inheritance is abused

-------------------------------------

# The Light

> "Favor object composition over class inheritance."

~ The Gang of Four, "Design Patterns"

-------------------------------------

## There is no class

-------------------------------------

## What is a prototype?

A working sample.

It's that simple.

-------------------------------------

## Three kinds of prototypes

-------------------------------------

## Delegate prototype

* Shared methods

-------------------------------------

## Flyweights for free

-------------------------------------

## Old n busted

``` !{language=javascript}
function Greeter(name) {
  this.name = name || 'John Doe';
}

Greeter.prototype.hello = function hello() {
  return 'Hello, my name is ' + this.name;
}

var george = new Greeter('George');
```
-------------------------------------

## New hotness

``` !{language=javascript}
var proto = {
  hello: function hello() {
    return 'Hello, my name is ' + this.name;
  }
};

var george = Object.create(proto);
george.name = 'George';
```

-------------------------------------

## Cloning / Concatenation

* Great for default state
* Mixins

-------------------------------------

## Mixin style

``` !{language=javascript}
var proto = {
  hello: function hello() {
    return 'Hello, my name is ' + this.name;
  }
};

var george = _.extend({}, proto, {name: 'George'});
```
-------------------------------------

## Anything can emit Backbone events

``` !{language=javascript}
var foo = _.extend({
  attrs: {},
  set: function (name, value) {
    this.attrs[name] = value;
    this.trigger('change', {
      name: name,
      value: value
    });
  },
  get: function (name, value) {
    return this.attrs[name];
  }
}, Backbone.Events);
```
-------------------------------------

## Functional inheritance

* Great for encapsulation / privacy
* Functional mixins

-------------------------------------

## Function prototypes
## not Object prototypes

Can replace:

* Constructors
* Init functions
* super()

-------------------------------------

## A simple model implementation

``` !{language=javascript}
var model = function () {
  var attrs = {};

  this.set = function (name, value) {
    attrs[name] = value;
    this.trigger('change', {
      name: name,
      value: value
    });
  };

  this.get = function (name, value) {
    return attrs[name];
  };

  _.extend(this, Backbone.Events);
};
```
-------------------------------------

## The result

``` !{language=javascript}
var george = {};
model.call(george).set('name', 'George');
george.get('name'); // 'George'

george.on('change', function (event) {
  console.log(event.name, event.value);
});

george.set('name', 'Simon'); // name Simon
```

-------------------------------------

## Use them all

-------------------------------------

## Use factory functions

* Like constructors, but you don't need `new` or `this`.
* Mix and match all three types of prototypes
* Use `.call()` and `.apply()` to swap out source prototypes
at instantiation time.

-------------------------------------

## Problem

Lots of hoop-jumping to use all these features together.

-------------------------------------

# Stampit

Create objects from reusable, composable behaviors.

-------------------------------------

Getting started (Node):

```
$ npm install stampit
```
-------------------------------------

Getting started (without Node):

```
$ git clone git://github.com/dilvie/stampit.git
```
-------------------------------------

## Specify which prototypes to use

* `.methods()` for delegation
* `.state()` for cloning / concatenation
* `.enclose()` for functional / data privacy

-------------------------------------

## Returns a factory

* `.compose()` it with other stampit factories
* Invoke it to instantiate new objects

-------------------------------------

# Demos

-------------------------------------

### “how do I inherit privileged methods and private data?”

``` !{language=javascript}
var a = stampit().enclose(function () {
  // Secrets go here:
  var a = 'a';

  // Methods can access secrets.
  // Nothing else can.
  this.getA = function () {
    return a;
  };
});
```
-------------------------------------

### Invoke the factory to get instances:

``` !{language=javascript}
a(); // Object -- so far so good.
a().getA(); // "a"
```
-------------------------------------

### Let's make another

``` !{language=javascript}
var b = stampit().enclose(function () {
  // This var won't collide with the other `a`
  var a = 'b';

  this.getB = function () {
    return a;
  };
});
```
-------------------------------------

### Time to shake it up

``` !{language=javascript}
var c = stampit.compose(a, b);

var foo = c(); // we won't throw this one away...

foo.getA(); // "a"
foo.getB(); // "b"
```
-------------------------------------

### What was that?

* Multiple ancestors
* Inherited private members

-------------------------------------

Can your class library do that?

ES6 `class` can't.

-------------------------------------

## Let's make some mixins...

``` !{language=javascript}
var availability = stampit().enclose(function () {
  var isOpen = false; // private

  return stampit.extend(this, {
    open: function open() {
      isOpen = true;
      return this;
    },
    close: function close() {
      isOpen = false;
      return this;
    },
    isOpen: function isOpenMethod() {
      return isOpen;
    }
  });
});
```
-------------------------------------

## Membership has rewards...

``` !{language=javascript}
var membership = stampit({
    add: function (member) {
      this.members[member.name] = member;
      return this;
    },
    getMember: function (name) {
      return this.members[name];
    }
  },
  {
    members: {}
  });
```
-------------------------------------

## Set some defaults...

``` !{language=javascript}
var defaults = stampit().state({
    name: 'The Saloon',
    specials: 'Whisky, Gin, Tequila'
  });
```
-------------------------------------

> "Favor object composition over class inheritance."

``` !{language=javascript}
var bar = stampit.compose(defaults, availability, membership);

// Note that you can override state on instantiation:
var myBar = bar({name: 'Moe\'s'});

// Silly, but proves that everything is as it should be.
myBar.add({name: 'Homer' }).open().getMember('Homer');
```
-------------------------------------

## Classes deal with the idea of an object
## ~
## Prototypes deal with the objects themselves

-------------------------------------

> “When there is freedom from mechanical conditioning, there is simplicity. The classical man is just a bundle of routine, ideas and tradition. If you follow the classical pattern, you are understanding the routine, the tradition, the shadow – you are not understanding yourself.”

~ Bruce Lee
