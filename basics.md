## Scala Basics 
### Functor





Functors are objects that can implement the map function. Examples for objects are collections, container or single values.
Overall you can say if you can map the object, it is a functor.

A map is a higher order function. This means that it's a function that either takes a function as an argument or returns a function. For example you define a List in Scala:

```Scala
val l1 = List(1,2,3)
//l1: List[Int] = List(1,2,3)
```

in the next step you create a new list by adding one to each element of the list by using the map function:

```Scala
val l2 = l1.map(x=>x+1)
// l2: List[Int] = List(2, 3, 4)
```

this makes it clearer that map is a higher order function on lists. Therefore is List[] a functor because you can map it.

In Scala you can think of a functor as a trait. In the following definition represents F the functor:

```Scala
trait F[A] {
  def map(f: A => B): F[B]
}
```

What functors do is to transform the context of a value and you will get back the same type of structure. 

You have a value which is wrapped into a context ``` F[A]```. 

After you apply map to it, the value will be unwrapped and the function will be applied ```(f: A => B)```. 

At least the value will be rewrapped back into the context ```F[B]```. 

If you look at the first example, you have List l1 and after you map it you get l2 which is also a List and again a functor.

