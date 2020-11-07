---
title: Trampoline - A better way for stack safety in Scala
date: 2020-08-13T09:51:20+01:00
layout: single
---

This read is about being stack safety in scala and what are the ways to be stack safe.

Before getting into `Trampoline` first let's understand  what is tail recursion ? And how  `scala compiler` optimize it ? 

Suppose we want to write a function which will return `sum of all number in given List`
```scala
def sum(seq: List[Int]): Long = seq match {
	case Nil => 0
	case head::tail => 
		head + sum(tail)
}
```

 1.  Above solution is `not a tail recursive`
   - Every new method call need previous call value, so need to keep previous stack 
   - If we have `List` with large number of elements then, there will  `List.lnegth`  stack we need keep to get result.
   -  If process does not have enough memory to keep all these stack we will get eventually a `StackOverflowException` 

Let's rewrite above to remove dependency of previous stack 
```scala
def sum(seq: List[Int], sum: Int = 0): Long = seq match {
	case Nil => sum
	case head::tail => 
		sum(tail, sum + head)
}
```
1. Above solution is `tail recursive` 
2. As runtime don't to have to remember value from previous stack, it can the use same stack for new method call or Get rid of previous call to make some space  

But  there is problem with  scala runtime does not do any optimization that we expected to do by `default`.
So even with above solution with `tail recursive` call, we can get `StackOverflowException` for large size `List`.

As runtime doesn't do anything about the tail call optimization `Scala compiler` provide way to convert `tail recursive` into `loop`  

```scala
@scala.annotation.tailrec
def sum(seq: List[Int], sum: Int = 0): Long = seq match {
	case Nil => sum
	case head::tail => 
		sum(tail, sum + head)
}
```

As per documentation the annotation `@scala.annotation.tailrec`  on `def` compiler do following things
1. It verifies that the method will be compiled with tail call optimization.
2. If it is present, the compiler will issue an error if the method cannot  be optimized into a loop.

So far ....

 1. We got one solution using  which is using scala compiler with `@scala.annotation.tailrec` which will convert tail call into loop.

Is there is any better way to be stack safe?  
Yes,  and  **`Trampoline`** it is .

What is **`Trampoline`** ?
1. It is way to define recursive call as ADT (Algebraic Data Types).
2. Way to describe recursive call  as a data using  ADT (Algebraic Data Types ).

Lets define `Trampoline`  ADT 
```scala
sealed trait Trampoline[T]
final case class Done[T](value: T) extends  Trampoline[T]
final case class More[T](next: () => Trampoline[T])
```
Now we have ADT for  `Trampoline` let's define recursive method using above
```scala
def sum(seq: List[Int], sum: Long = 0): Trampoline[Long] = seq match {
	case Nil => Done(sum)
	case head::tail => 
		More(() => sum(tail, sum + head))
}
```

As per above by calling , will get `Trampoline` ADT
```scala 
val trampoline: Trampoline = sum(List(1,2,3))
```
Let's write interpreter for `Trampoline` algebra 
```scala
def run[T](trampoline: Trampoline[T]): T = trampoline match {
	case Done(value) => value 
	case More(f) => run(f())
}
``` 

Let's get value 
```scala 
val trampoline: Trampoline = sum(List(1,2,3))
run(trampoline) //6
```

**Summary** 

Two ways to be stack safe 
 1. Using tail call optimization 
 2. Using `Trampoline` 
