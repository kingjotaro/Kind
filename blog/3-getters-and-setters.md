Getters and setters in Kind
===========================

One of the most annoying parts of pure functional programming is getting,
setting and mutating deeply nested fields. In impure languages like JavaScript,
this was never a problem. For example, consider the following object:

```javascript
let obj = {
  name: "sample"
  data: {
    "a": [1.0, 2.0, 3.0, 4.0, 5.0, 6.0]
    "b": [7.0, 7.0, 7.0, 7.0, 7.0, 7.0]
  }
}
```

Setting nested fields is easy:

```javascript
obj.data["a"][0] = 42.0
```

In Haskell, the equivalent code is verbose. Lenses greatly improve the
situation, but they 1. have considerable runtime cost, 2. require big external
libraries, 3. can be overkill, 4. are still not as succinct as the JS code.

To be fair, the JavaScript version, while terse, is problematic. Not only
because it mutates the original object, but because, if any of these keys don't
exist, the program above will crash. To make this program safe, one must make
several checks that end up making the code verbose too:

```javascript
var data = obj.data
if ( obj.data !== undefined
  && obj.data["a"] !== undefined
  && obj.data["a"][0] !== undefined) {
  obj.data["a"][0] = 42.0
}
```

In Kind, the earlier versions of the language suffered from a similar problem.
The equivalent object could be defined as:

```javascript
type Object {
  new(
    name: String
    data: Map<List<F64>>
  )
}

obj: Object
  Object.new("sample", {
    "a": [1.0, 2.0, 3.0, 4.0, 5.0, 6.0] 
    "b": [7.0, 7.0, 7.0, 7.0, 7.0, 7.0] 
  })
```

And, like on most pure languages, setting nested fields was verbose:

```javascript
obj2: Object
  case obj {
    new: case Map.get!("a", obj.data) as got_list {
      none: obj
      some: case List.get!(0, got_list.value) as got_number {
        none: obj
        some: Object.new(obj.name, Map.set!("a", List.set!(0, 42.0, got_list.value), obj.data))
      }
    }
  }
```

Since the last version, Kind features a built-in getter and setter syntax that
makes these operations very succinct. The program above can be written as just:

```javascript
obj2: Object
  obj@data{"a"}[0] <- 42.0
```

Notice how `x@field` accesses a field, `x{key}` accesses a Map entry, and
`x[index]` accesses a List element. These accessors can be chained to access
deep fields:

```javascript
data: Map<List<F64>>
  obj@data

nums: Maybe<List<F64>>
  obj@data{"a"}

number: Maybe<F64>
  obj@data{"a"}[0]
```

Notice a `Maybe` shows up only when needed (such as when getting an element from
a list or map). If you and the access with a `<-`, instead of getting the field,
you'll set it. You can also use `<=` to apply a function instead of setting:

```javascript
obj3: Object
  obj@data{"a"}[0] <= Nat.mul(2)
```

This desugars to an efficient, linear core program that doesn't involve heavy
lenses and avoids re-getting nested fields. 

So, in short, dealing with nested fields in JavaScript looks nice but is
terrible; in Haskell, it looks terrible and is; in Kind, is the a joyful
experience that makes you proud about your career choice.

I'm making this post because this is such a huge, needed quality of life
improvement that I really think every pure language should come with something
similar out of the box, and I don't understand why this is so hard. You
shouldn't need huge third party libs to do something as simple.

Finally, note this is *not* a built-in lens implementation. Lenses are
first-class objects. Instead, it is just a baseline syntax for immutably
setting, getting and modifying nested values in records, lists and maps. And
that completely changes how the language feels.