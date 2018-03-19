### Introduction of Dependent Type with Idris

Yohei Oda

---

### whoami

- Scala engineer
- ex-troubleshooter of Java technology
- Coq??😪

---

### What I want to talk...

- Introduce Idris and dependent type in some simple cases
- Show difference from usual programming languages
- type is interesting

---

#### What's idris?

- Functional Programming language
  - Haskell based syntax
  - a little bit different...
    - default eager evaluation
    - Type annotation `:` , Cons `::`
    - etc
- latest version 1.2.0
  - Version 1 doesn't mean "for production"

---

#### What's idris?

- This slide is based on the book "Type Driven Development with Idris"
- You can buy it in manning or amazon.
- https://www.manning.com/books/type-driven-development-with-idris

![](https://images.manning.com/720/960/resize/book/1/453215a-afa1-443f-9f2d-3b6bf24c34db/Brady-TDDI-HI.png)

---

#### What's idris?

- Idris can use "dependent type" natively.

![](https://upload.wikimedia.org/wikipedia/commons/1/19/Lambda_cube.png)

from [wikipedia](https://upload.wikimedia.org/wikipedia/commons/1/19/Lambda_cube.png)

---

#### What's dependent type?

- Very strong type containing value
  - Type depends on value
- `types are first class`
  - Type like value and function
- Proof with dependent type
  - but Idris focus on `general purpose`

---

#### Let's start dependent type

- Vect
  - list which contains number of elements in its type
  - `Vect n a`
    - n: number of elements
    - a: type of elements
---

#### Vect

- Type

```haskell
Vect : Nat -> Type -> Type
```

- Constructor

```haskell
Nil : Vect 0 elem
    Empty vector

(::) : (x : elem) -> (xs : Vect len elem) -> Vect (S len) elem
    A non-empty vector of length S len, consisting of a head element and the rest of the list, of length len.
    infixr 7
```

- Nat is a type represents `natural number`
   - `Z` or `S a`
---

Lets's write some Vect's functions

---

#### map for Vect

```haskell
my_vect_map: (a->b) -> Vect n a -> Vect n b
my_vect_map f [] = []
my_vect_map f (x :: xs) = f x :: my_vect_map f xs
```

```
> my_vect_map length ["apple", "banana"]
[5, 6] : Vect 2 Nat
```
---

#### append for Vect

```haskell
my_vect_append : Vect n a -> Vect m a -> Vect (n + m) a
my_vect_append [] ys = ys
my_vect_append (x :: xs) ys = x :: my_vect_append xs ys
```

```
> my_vect_append [1,2,3] [6,7]
[1, 2, 3, 6, 7] : Vect 5 Integer
```

---

##### tail for Vect

```haskell
my_vect_tail: Vect (S n) a -> Vect n a
my_vect_tail (x :: xs) = xs
```

- use `Vect (S n) a`
  - cannot call this function with empty vector
  - this function never fails

---

##### zip for Vect

```haskell
my_vect_zip : Vect n a -> Vect n b -> Vect n (a, b)
my_vect_zip [] [] = []
my_vect_zip (x :: xs) (y :: ys) = (x, y) :: my_vect_zip xs ys
```

- can call only when the numbers of elements of two Vector are same

---

##### index for Vect

```haskell
my_index : Fin len -> Vect len elem -> elem
my_index FZ     (x::xs) = x
my_index (FS k) (x::xs) = index k xs
```

- `Fin len` means a natural number from 0 to len.
- this function never fails

---

#### Merit of dependent type so far

- Express strong contracts in signature
  - It is clear what the function wants
- Contracts in signature is forced by compiler
- Prevent runtime error
- More testable😊

---

#### Let's call my_zip

1. We have 2 Vectors, `Vect n String` and `Vect m Int`
1. `n==m` returns `True`
1. Is it possible to call `my_zip`?

---

Compile error 😭

---

#### Why?

- Compiler cannot gurantee the behavior of `n==m`
- We must show the equality of these values for compiler
- (We can use build-in data types and functions for it but first we create them by ourselves for study)

---

#### Define a data type for equality

```haskell
data EqNat : (num1 : Nat) -> (num2 : Nat) -> Type where
     Same : (num : Nat) -> EqNat num num
```

- Type `EqNat` has only one data constructor `Same`
- Constructor shows the following statement
  - If a value with `EqNat a b` type exists, a and b are same.

---

#### Write function for EqNat

```haskell
sameS : (k: Nat) -> (j: Nat) -> (eq : EqNat k j) -> EqNat (S k) (S j)
sameS j j (Same j) = Same(S j)
```

- This function shows the following statement
  - If `k` and `j` are same, `S k` and `S j` are same. 

---

#### create `EqNat` instance

```haskell
checkEqNat : (num1 : Nat) -> (num2 : Nat) -> Maybe (EqNat num1 num2)
checkEqNat Z Z = Just (Same 0)
checkEqNat Z (S k) = Nothing
checkEqNat (S k) Z = Nothing
checkEqNat (S k) (S j) = case checkEqNat k j of
                              Nothing => Nothing
                              (Just eq) => Just (sameS _ _ eq)
```

---

#### convert `Vect m a` to `Vect len a`

```haskell
exactLength : (len : Nat) -> (input : Vect m a) -> Maybe (Vect len a)
exactLength {m} len input = case checkEqNat m len of
                                 Nothing => Nothing
                                 Just (Same len) => Just input
```

- Compiler can deal `input` as `Vect len a` type's value

---

#### call `my_zip`

```haskell
exactLength : (len : Nat) -> (input : Vect m a) -> Maybe (Vect len a)
exactLength {m} len input = case checkEqNat m len of
                                 Nothing => Nothing
                                 Just (Same len) => Just input

my_zip: Vect n a -> Vect n b -> Vect n (a, b)
my_zip [] [] = []
my_zip (x :: z) (y :: w) = (x, y) :: my_zip z w

maybe_zip: Vect n a -> Vect m b -> Maybe(Vect n (a,b))
maybe_zip {n} x y = case exactLength n y of
                      Nothing => Nothing
                      Just z => Just (my_zip x z)
```

---

#### generalize `checkEqNat`

```haskell
checkEqNat' : (num1 : Nat) -> (num2 : Nat) -> Maybe (num1 = num2)
checkEqNat' Z Z = Just Refl
checkEqNat' Z (S k) = Nothing
checkEqNat' (S k) Z = Nothing
checkEqNat' (S k) (S j) = case checkEqNat' k j of
                             Nothing => Nothing
                             (Just proof) => Just (cong proof)
```

- rewrite checkEqNat with built-in data types and functions of Idris
- `num1 = num2` is general version of `EqNat num1 num2`
- cong is general version of `sameS`
- Actually Idris can express stronger decidability than this(but no time...) 

---

#### auto-implicit argument

```haskell
Prelude.List.tail : (l : List a) -> {auto ok : NonEmpty l} -> List a
```

- search `NonEmpty l` type's value in compile time
  - `NonEmpty`'s contructor

  ```haskell
  IsNonEmpty : NonEmpty (x :: xs)
  ```
---

#### Summary

- With dependent type, you can express many kinds of contracts in signature
  - contracts are guranteed by type system
- Proofs are often necessary for compiler to understand types precicely
  - type constructor is very important

---

#### Other interesting topics...
- Decidability and `Void`
- Dependent pattern match with view
- Dependent pairs
- Show which vector contains the value or not
- `rewrite` to use proof
- IDE support with dependent type
- etc

---

#### References

- [Type Driven Development with Idris](https://www.manning.com/books/type-driven-development-with-idris)
  - （almost） official guidebook for Idris

- [Idris Tutorial](http://docs.idris-lang.org/en/latest/tutorial/index.html)

- [Idris for Haskellers](https://github.com/idris-lang/Idris-dev/wiki/Idris-for-Haskellers)

- [10 things Idris improved over Haskell](https://deque.blog/2017/06/14/10-things-idris-improved-over-haskell/)

- [Idris for Scala developers](https://slides.com/cb372/idris-for-scala-devs/embed)
