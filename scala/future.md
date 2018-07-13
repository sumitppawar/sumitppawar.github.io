Future 
-----
```Future[T]``` the scala's way to handle concurrency. </br>
```Future[T]``` is write once container, once it completed we can not write it again.</br>
```Future[T]``` is read only interface, for composing future we ```Promise[T]``` interface.</br>
It provides great way to compose, chain operation on future, but by ```java Future``` we can mostly add callback or wait for future to Complete.

```
scala> import scala.concurrent._
scala> import scala.concurrent.ExecutionContext.Implicits.global
scala> val fInt = Future {4}
```
To complete future, it need to have ```ExecutionContext``` to complete its task.</br>
We can't complet future without promise, ```Future[T]``` object provide apply method which will return ```DefaultPromise```. </br>
As ```Future[T]``` is container, we have map, flatMap etc containers method available on future. </br>
For-comprehension for future run sequential, so make sure we extract future outside of for-comprehension.</br>

###### Composing Future using Promise[T]
```
def getAllUser: Future[List[String]] = {
  val promise = Promise[List[String]]
  try {
    val listuser = getUserFromDb
    promise.success(listuser)
  } catch {
    case t: Throwable => promise.failure(t)
  }
  promise.future
}
```
