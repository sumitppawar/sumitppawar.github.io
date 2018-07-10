Extractor

Scala extractors are used for pattren matching.</br>
If we define ```case class``` compiler will automatically generate ```unapply``` for you. </br>
When used for pattern matching scala compiler call unapply method of provided type. 
Extractor not need to be in same object of that type.</br>
Extractor is breaking encapsulation if we looks from point of OOPs, thats why scala encourage to separate data and behavior </br>
using case class as ADT.



##### Forms of extractor
###### When type that we want decompose has only one parameter. </br>
```
class Person(val name: String)

object Person {
  def unapply(p: Person): Option[String] = Option(p.name)
}

val person = new Person("Sumit")

person match {
  case Person(name) => println(name)
  case _ => println("Not found extractor")
}

```

###### When type that we want decompose has only multiple paramters. </br>
```
class Person(val name: String, val  age: Int)

object Person {
  def unapply(p: Person): Option[(String, Int)] = Option((p.name, p.age))
}

val person = new Person("Sumit", 27)

person match {
  case Person(name, age) => println(s"$name,$age")
  case _ => println("Not found extractor")
}

```

###### Boolean extractor </br>
```@```  helps to bind appropriate type to ```p```. 
```
class Person(val name: String, val  age: Int)

object PersonAgeValidator {
  def unapply(p: Person): Boolean = if(p.age>27) true else false
}

val person = new Person("Sumit", 27)

person match {
  case p @ PersonAgeValidator() => println(s"${p.name},${p.age}")
  case _ => println("Not found extractor")
}
```
</br>

###### Extractor on seq/stream
In scala we use extractor as infix operator.
```
new Person("name", 27) match {
  case name Person age => println("Awesome")
} 

```
If we have more than two parameter to decompose we can't use infix extractor. </br>
While doing pattern matching on Seq/Steam use ```:: or #::``` extractor. </br>

```
object #:: {
  def unapply[A](xs: Stream[A]): Option[(A, Stream[A])] =
    if (xs.isEmpty) None
    else Some((xs.head, xs.tail))
}

val stream = 1 #:: 2 #:: 2 #:: Stream.empty
stream match {
  case head #:: tail =>  println(s"$head")
  case _ => "Empty"
}
 ```
 While extracting multiple element works likes below 
 ```
 val stream = 1 #:: 2 #:: 2 #:: Stream.empty
stream match {
  case #::(head, #:: (second,tail)) =>  println(s"$head")
  case _ => "Empty"
}
 ```
 










