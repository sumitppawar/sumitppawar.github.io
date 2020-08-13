---
title: Tagless Final Vs Free monads
date: 2020-08-13T09:51:20+01:00
layout: single
---

World of functional programing is more about writing code which focus on **What (Algebra)**? rather than **how** ?.
This article is about how do we write description of a program (**Algebra/What** ) and segregated **How** (**Interpreter of an Algebra**).

So far I have found two functional pattern which will help us to define **How and What** of our code base.

Let's consider a small requirement of authenticate user token and fetch all the resources that user has access:

```scala
case class User(id: Id, name: String)
case class Resources(code: ResourceCode, name: String)
```

Steps

1.  Validate user `token`
2.  Find user using `token`
3.  Fetch user `Resource`s using user `Id`

Let's solve using two techniques (functional patterns)

1.  Tagless Final
2.  Free Monads

#### Using Tagless Final

Let's define algebra for `UserRespositoryAlg`

```scala
trait UserRespositoryAlg[F[_]] {
	def validateToken(token: Token):F[Unit]    //<- expression for token validation
	def findUser(token: Token): F[User] //<- expression for find user
	def getUserResources(userId: Id): F[Seq[Resource]] //<- expression for find Resource
}
class UserService[F[_]:Monad](repo: UserRespositoryAlg[F]) {.
	// Monad <- has map and flatMap which can usefull in for comprehension
	// No implementation yet, but we are still able  define requirement
	def getResources(token: Token): F[Seq[Resource]] = for {
			_          <- validateToken(token)
			user       <- findUser(token)
			resources  <- getUserResources(user.id)
		} yield resources
}
```

1.  We compose our business logic using different expression (Function composition)
2.  We are able to define our business logic **`What`** without knowing **`how`** and our code base is more descriptive

Let's define interpreter for above `UserRepositoryAlg`

```scala
import scalaz._ //Using scalaz for monad instance of Future
import Scalaz._
import scala.concurrent.ExecutionContext.Implicits.global
object UserRespositoryInterpreter[F[_]:Applicative] extends UserRespositoryAlg[F] {
		def validateToken(token: Token):Future[Unit] = implicitly[Applicative[F]].point(())

		def findUser(token: Token): Future[User] = implicitly[Applicative[F]].point((
			User(Id("qsiuwfgweifvkwjev"), "Tagless")
		)

		def getUserResources(userId: Id): Future[Seq[Resource]] = implicitly[Applicative[F]].point {
			Seq(
				Resource(ResourceCode("alert-dashboard"), "Alert dashboard" ),
				Resource(ResourceCode("transaction-dashboard"), "Transactions dashboard")
			)
		}
}
```

1.  While writing interpreter (`UserRespositoryInterpreter`) for our algebra (`UserRespositoryAlg`)
    - We can choose any `F[_]`, let's use `Future` type construct (Need a Monad instance defined for `Future`)
    - We can easy replace type construct for testing , maybe be `F[Option]`

```scala
// All together
import scalaz._
import Scalaz._
import scala.concurrent.Await
import scala.concurrent.duration._
import scala.concurrent.ExecutionContext.Implicits.global
 val repo = UserRespositoryInterpreter[Future]
 val service = new UserService[Future](repo)
 val rF = service.getResources(Token("qsiuwfgweifvkwjev"))
 val resources =  Await.result(rF, 2 second)
```

#### Using Free Monad (Using scalaz)

1. Idea behind using free monad is define `expressions` as data (`ADT`)

   - In our business logic we four expression
     1. validateToken
     2. findUser
     3. getUserResources
   - Let's define above expressions as `ADT` (Algebraic Data Types)

   ```scala
   //Operations as data
   trait UserRespositoryAlg[S]
   case class ValidateToken(token: Token) extends UserRespositoryAlg[Unit]
   case class FindUser(token: Token) extends UserRespositoryAlg[User]
   case class GetUserResources(id: Id) extends UserRespositoryAlg[Seq[Resource]]

   //Service class
   object UserService {
   		def getResources(token: Token):Free[UserRespositoryAlg, Seq[Resource]]  =
   		for {
   		_          <- Free.liftF(ValidateToken(token))
   		user       <- Free.liftF(FindUser(token))
   		resources  <- Free.liftF(GetUserResources(user.id))
   		} yield resources
   }
   ```

2. As per above we have defined our business logic in terms of Free monads using ADT

   Let's define **interpreter** for above

```scala

val userRepoInterpreter = new (UserRespositoryAlg ~> Future) {  // scalaz way to define free monad interpreter
  override def apply[A](fa: UserRespositoryAlg[A]): Future[A] = fa  match {
    case ValidateToken(token) => Future.success(())
    case FindUser(token) => Future.success(User(Id("qsiuwfgweifvkwjev"), "FreeMonad"))
    case GetUserResources(id) => Future.success {
			Seq(
				Resource(ResourceCode("alert-dashboard"), "Alert dashboard" ),
				Resource(ResourceCode("transaction-dashboard"), "Transactions dashboard")
			)
		}
  }
}
//All together
val rF: Future[Seq[Resource]] = UserService
.getResources(Token("qsiuwfgweifvkwjev"))
.foldMap(userRepoInterpreter)

 val resources =  Await.result(rF, 2 second)
```

That's it !
