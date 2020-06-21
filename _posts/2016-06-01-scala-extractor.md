---
title: Scala Extractors
date: 2016-06-01T09:51:20+01:00
layout: single
---

 
Scala extractors are used for pattern matching.
1. If we define ```case class``` , compiler will automatically generate ```unapply``` for you.
  When used for pattern matching scala compiler call unapply method of provided type.
2. Extractor not need to be in same object of that type.
3. Extractor is breaking encapsulation if we looks from point of OOPs, that's why scala encourage to separate data and behavior .

 
Types of extractor

- Type that we want decompose has only one parameter.

```scala
class Person(val name: String)  
// If we define Pesron as a case class compile will generate unapply
object Person {
   def unapply(p: Person): Option[String] = Option(p.name)
}
val person = new Person("Sumit")
person match {
    case Person(name) => println(name)
	case _ => println("Extractor not found")
}
```
- Type that we want decompose has multiple parameters. 

```scala
class Person(val name: String, val age: Int)

object Person {
	def unapply(p: Person): Option[(String, Int)] = Option((p.name, p.age))
}

val person = new Person("Sumit", 27)
person match {
	case Person(name, age) => println(s"$name,$age")
	case _ => println("Extractor not found")
}
```
-  Boolean Extractor 
	- `@` helps to bind appropriate type to `p`

```scala
class Person(val name: String, val age: Int)
object PersonAgeValidator {
	def unapply(p: Person): Boolean = if(p.age>27) true else false
}

val person = new Person("Sumit", 27)
person match {
	case p @ PersonAgeValidator() => println(s"${p.name},${p.age}")
	case _ => println("Invalid age")
}

```
- Extractor as infix operator
	- In scala we can also use  extractor as infix operator.
	-  If we have more than two parameter to decompose,  we can't use infix extractor.

```scala
new Person("name", 27) match {
	case name Person age => println("Awesome")
}
```

- Extractor on Seq/Stream
	- While doing pattern matching on Seq/Steam use ```:: or #::``` extractor.

```scala

object #:: {
	def unapply[A](xs: Stream[A]): Option[(A, Stream[A])] =
		if (xs.isEmpty) None
		else Some((xs.head, xs.tail))
}
val stream = 1 #:: 2 #:: 2 #:: Stream.empty
stream match {
	case head #:: tail => println(s"$head")
	case _ => "Empty"
}
```
- Extracting multiple element

```scala
val stream = 1 #:: 2 #:: 2 #:: Stream.empty
stream match {
	case #::(head, #:: (second,tail)) => println(s"$head")
	case _ => "Empty"
}
```