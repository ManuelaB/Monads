## Scala Basics 

In this blog we get to know more about monads. 

Monads are derived from category theory in mathematics. They are commonly used in programming to mean higher order functions. 
For this post it isn't necessary to have knowledge in category theory.

Having a programming language like Scala where everything has to be immutable is often hard to intercat. Because in the real world everything changes the whole time. 
They found something in monad which allows to explain it in the function contract. Everytime you call a function with the same argument you have to get same result in cause no side effects. But if you pass a function as parameter to a function you can get different results.

Before we get to know more about monads we have to define a functor because monad is a type of it.

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

In Scala you can think of a functor as a trait. In the following pseudo-code represents F the functor:

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


### Monads

Monads are like wrappers where you can wrap objects. It's like wrapping a present to make it look prettier and that's the same with monads. They are also allow us to handle asynchronous code which is very powerful.

Recalling that a functor is a type which has a map method, a monad is a type of functor which implements the flatMap method. This can be written in a pseudo-code:

```Scala
trait M[A] {
  def map(f: A => B): M[B]
  def flatMap(f: A => M[B]): M[B]
}
```

You have a class M with the type A, this would be written in M[A]. If you want to flatten it you need to provide a function A -> M[B]. FlatMap will transform A into M[B] and the result is M[M[B]]. After mapping, flatMap will flatten it and you get M[B].


The following example shows how to map a function returning a list over the elements of a List:

```Scala
val l3 = l1.map { x => List(x-1, x+1) }
//l3: List[List[Int]] = List(List(0, 2), List(1, 3), List(2, 4))
```

This example returns a list of lists but this result looks a bit confusing. If you want to flatten it down to have only the list with the values in it you can use the operation flatMap:

```Scala
val l4 = l1.flatMap { x => List(x-1, x+1) }
//l4: List[Int] = List(0, 2, 1, 3, 2, 4)
```

Notice that not all functors can have a flattening operation, so not all functors are monads, but all monads are functors. Monads are therefore more powerful than functors.

The flatMap function doesn't require a function with A -> M[A]. If M is a List and A is a Int, Int -> List[String], this would also work, so it doesn’t have to be Int -> List[Int].

```Scala

val l5 = l1.flatMap(x=>List(x + "a"))
//l5: List[String] = List(1a, 2a, 3a)
```

Therefore flatMap is more powerful then map because it will chain operations together. The map function is just a subset of flatMap.

The synthetic sugar of a flatMap method is to write it as a for-comprehension with the symbol ```<-```. Notice that it is not comparable with a for-loop.

```Scala
l1.flatMap(x => l2.map(y => x + y ) )
//res0: List[Int] = List(3, 4, 5, 4, 5, 6, 5, 6, 7)

for {
    first <- l1
    second <- l2
} yield first + second
//res1: List[Int] = List(3, 4, 5, 4, 5, 6, 5, 6, 7)
```

As you can see, the for-comprehension is much more readable then using the flatMap method.


### When do we need Monads?

To get a better understanding why Moands are helpful, here is an example.

Lets say you want to get the address from a user. These two methods will return a user and an address with the type Future. 

```scala
def getUser(name: String): Future[User] = ...

def getAddress(user: User): Future[Address] = ...
```

As we mentioned before, Future is an asynchron computation and will return a value in some time later. It has also the flatMap method which you can use to transform the content and also write it as a for-comprehension.

```scala
val getCity: Future[String] =
      getUser("Andreas").flatMap(user => getAddress(user).map(address => address.city))
```


```scala
val getCity: Future[String] = 
    for {
        user        <- getUser("Andreas")
        address     <- getAddress(user)
    } yield address.city
```

The code is now much more easier to read. First you get the user and pass it to the getAddress method. Once you have the address you will get the city of it.

There are several problems which we didn't mention yet. It is possible that not every name corresponds to an existing user or the user has no address. For this reason you can use Option which serves this purpose. You just have to change the returntype of the two methods and the value ```getCity```.

```scala
def getUser(name: String): Future[Option[User]] = ... 

def getAddress(user: User): Future[Option[Address]] = ...
```

```scala
val getCity: Future[Option[String]] = 
    for {
        user        <- getUser("Andreas")
        address     <- getAddress(user)
    } yield address.city
    
```

There is another problem because it doesn't compile any more.

```scala
error: type mismatch;
 found   : Option[User]
 required: User
           address <- getAddress(user)
```

The getAddress method needs the type User but getUser hands over an Option[User]. In the follwing example you can see the procedure of unwrapping the values. The first extraction works easily but second where you hand over the value user is failing. 

```scala
user    <- getUser("Andreas")   <=>   Option[User] <- Future[Option[User]] 

address <- getAddress(user)     <=>   address <- getAddress(Option[User]) //ERROR            
```

Now you try to solve the problem by writing something like this:

```scala
val getCity: Future[Option[String]] = 
    for {
        maybeUser  <- getUser("Andreas")
        maybeCity  <- maybeUser match{
            case Some(user) => getAddress(user).map(_.map(_.city))
            case None => Future.successful(Node)
        }
    } yield maybeCity
```

This code doesn't look good so you ideally want to write it more readable:

```scala
def city: Future[Option[String]] =

  for {
    user    <- maybeUser    <- getUser("Andreas")
    address <- maybeAddress <- getAddress(user)
  } yield address.city
```    

You double flatMap the two methods. First extracting from the Future and then from the Option that you get just the user.

```scala
        user        <-       maybeUser       <-       getUser("Andreas")


"Andreas" = User <------ Option["Andreas"] <------ Future[Option["Andreas"]]
```

This code example doesn't work in real live so you want something like a method which works on Future[Option[X]]. For this case you need a wrapper class.


### Wrapper class

Let's go back to Functors and Monads. If you have two Functors A and B and you know how to map over A[X] and B[X] you also know how to map over A[B[X]].
This is not possible for Monads. Knowing how to flatMap over A[X] and over B[X] doesn't grant you to flatMap over A[B[X]].

Therefore you have to write a wrapper class that works for Future[Option[A]] with its own map and flatMap methods.

The first snippet shows again the definition of a Monad class and the second the wrapper class.

```scala
trait M[A] {
    
	def map(f: A => B): M[B] 
	def flatMap(f: A => M[B]): M[B] 
}
```

```scala
case class FutOpt[A](value: Future[Option[A]]) {
 
    def map[B](f: A => B): FutOpt[B] = FutOpt(value.map(optA => optA.map(f)))
  
    def flatMap[B](f: A => FutOpt[B]): FutOpt[B] = 
        FutOpt(value.flatMap(opt => opt match {
            case Some(a) => f(a).value
            case None => Future.successful(None)
        }))
}
```

Now we can try again to get the city from a user and running the code with the wrapper class. You just have to use the FutOpt in the for-comprehension and use .value to get the result.

```scala
val f: Future[Option[String]] =
    for {
        user    <- FutOpt(getUser("Andreas"))
        address <- FutOpt(getAddress(user))
    } yield address.city
    
    
val getCity: Future[Option[String]] = f.value
```

The wrapper class also works for List[Option[A]].

```scala
case class ListOpt[A](value: List[Option[A]]) {
 
    def map[B](f: A => B): ListOpt[B] = ListOpt(value.map(optA => optA.map(f)))
 
    def flatMap[B](f: A => ListOpt[B]): ListOpt[B] =
        ListOpt(value.flatMap(opt => opt match {
            case Some(a) => f(a).value
            case None => List(None)
        }))
}
```

As you can see there is no knowledge about the 'outer' Monad (Future or List) required. The 'outer' Monad can be any type of structure as long as you can map or flatMap over it. You just need a knowledge about the 'inner' Monad, which is Option in this case. In general it's a generic data structure that wraps any Monad around an Option.

The library cats already provides a class OptionT which solves this problem.


### OptionT

OptionT has two type parameters F and A: **OptionT[F, A].**
F is the wrapping Moand and A is the type inside Option. OptionT[F, A] is a flat Version of F[Option[A]] which we had in our examples before. It is also Monad and has its own map and flatMap methods so you can use the for-comprehension.

In general you can say: **Foo[Bar[X]] becomes BarT[Foo, X]**

To implement the OptionT you have to import
