---
title: Better Scala - Prefer ad-hoc Polymorphism over sub-type Polymorphism
date: 2017-01-01T09:51:20+01:00
layout: single
---

Let's assume that we want add a `def ++[A](a: A, b:A):A` on type  `A`

1. Using **Sub-Type** Polymorphism 

```scala
trait Plus[A] {  
  def plus(b:A): A  
}  
  
def ++[A<:Plus[A]](a: A, b:A):A = a.plus(b)  
  
case class Money(n: Int) extends Plus[Money] {  
  def plus(b:Money): Money = Money(n+b.n)  
}  
++(Money(1), Money(2))
```
 - In above example , we are saying that `def + will work for any type A which is sub-type of Plus`
 - Limitations 
	 - How to add `+` operation for type which are coming for other Lib or do not have access 
		 - Example `String, Int `

2. Using **Ad-hoc**  Polymorphism (TypeClass)

```scala
trait Plus[A] {  
  def plus(a:A, b:A): A  
}  
def ++[A:Plus](a: A, b:A):A = implicitly[Plus[A]].plus(a,b)  
case class Money(n: Int)  
  
implicit val moneyPlusOps = new Plus[Money] {  
  override def plus(a: Money, b: Money): Money = Money(a.n + b.n)  
}  
++(Money(1), Money(2))
```
- No sub-typing  (No issues with we get using inheritance)
- We can add behaviour to any Type without know anything about type or source code
- The function definitions can be enabled or disabled in different scopes