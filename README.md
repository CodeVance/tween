# tween.core
Tween library for [arcadia-unity](github.com/arcadia-unity/Arcadia)

![tween](https://cloud.githubusercontent.com/assets/2467644/19105679/52ed9bbe-8ab1-11e6-9a4c-3087e5f43e09.gif)


## timelines


`timeline` starts a coroutine that iterates a `seq?` collection. Items are invoked every frame and lazily iterate when falsey. 

```clj
(timeline [
 #(log "hello")
  (wait 1.0)
 #(log "goodbye")])
```


`timeline-1` uses another fn layer to provide an initialization closure. This closure is needed to repeat `wait` and `tween` fns.

```clj
(timeline-1 
  (cycle [
    (fn [] (wait (rand)))
    (fn [] #(log 'tick)])))
```

`timeline*` macro wraps forms with `(fn [])`, and uses an Array for performance. Keywords are removed for options. `:loop` cycles the iterator.

```clj
(timeline* :loop
  (wait 0.5)
  (tween {:local {
    :position (v3 0 2 0)
    :scale (v3 (rand))}} (object/named "ball") 0.9))
```

### wait

`(wait 0.5)`

returns fn that returns true for the duration after it's first invokation

### timeline control macros: `AND`, `OR`, and `NOT`

returns a fn returning the boolean result of invoking forms.  Uses let bindings for each form to "evaporate" any closures. Pretty experimental, but does allows temporal control flow.

```clj
(timeline* 
  (AND (wait 2.0) 
       (OR (AND (NOT (wait 1.0))
                (tween {:position (v3 0 10 0)} foo 5.0))
           (OR (wait 3) 
              #(log "cancel")))))
```

## tweens

## `deftag` macro

Registers a type for tweening.  `:pair` should be a type with mutable `a` and `b` fields. `:lerp` fn takes two values and a System.Single ratio, which will be between `0` and `1`. 

```clj
(deftag       UnityEngine.Vector3 
  {:pair      Vector3Pair
   :lerp      UnityEngine.Vector3/Lerp           
   :identity  (UnityEngine.Vector3.)})
```

## `deftween` macro

```clj
(deftween [:material :color] [this]
  {:get (.color this)
   :tag UnityEngine.Color
   :base (.material (.GetComponent this UnityEngine.Renderer))
   :base-tag UnityEngine.Material})
```

register an edn path as a tween. The path is arbitrary and the symbol binding will be to a user provided object.
*  `arg-1` edn path
*  `arg-2` vector with symbol used in getter code 
*  `arg-3` map with:
  *  `:tag` qualified type of property (should be registered with deftag)
  *  `:get` code to get tweenable property
  *  `:base` optional getter for derived object 
  *  `:base-tag` optional type-hint for base object





## `tween` macro

Expands into a fn that returns true for the duration after it's first invokation. The target map is exploded into paths, a path registered with `deftween` will expand into code.

```clj
(tween 
 {:local 
   {:scale (Vector3. 2.0 2.0 2.0)
    :position (Vector3. 0.0 2.0 0.0)}}  
  (GameObject.) 
  2.5 {:in :pow5})
```

*  `arg-1`  map of target values
*  `arg-2`  object reference
*  `arg-3`  duration
*  `& more` compiles into opt map, naked easing keys compile to both `:in` and `:out` opts




### easing fns

`{:in :pow3}`

*  keys `:in` `:out`
*  vals `:pow2` `:pow3` `:pow4` `:pow5` `nil`
