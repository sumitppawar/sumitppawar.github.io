---
title: Composing scalaz disjunction
date: 2016-10-23T09:51:20+01:00
layout: single
---
Scalaz provides disjunction `\/` which is kind of scala's `Either`. 
It provides ways to convert different type to disjunction and it compose.

Let's start with example 
Assume that we have different functions as below 
```scala
 def findCustomerById(id: Long): Option[Customer]
 def getCustomerAccount(customer: Customer): Option[Account]
 def credit(account: Account, amount: Double): Try[Unit]
 def debit(account: Account, amount: Double): Try[Unit]

type StringError = String
def transfer(customerId: Long, to: Account, amount: Double): StringError \/ Unit
```
In above we want to `compose` `def transfer` using other mentioned functions.
The `def transfer` is returning  `scalaz` disjunction 

Let's do it by using scalaz disjunction

```scala
import scalaz._
import Scalaz._
import scalaz.std.option.optionSyntax._

def transfer(customerId: id, toAccount: Account, amount: Double): StringError \/ Unit = 
 for {
	  customer <- findCustomerById(customerId).\/>(s"Customer with id $customerId not found")
	  fromAcc  <- getCustomerAccount(customer) .\/>(s"Customer account not found")
	  _        <- debit(fromAcc, amount).toDisjunction(th => s"Error while debit ${th.getCause}")
	  _.       <- credit(toAccount, amount).toDisjunction(th => s"Error while credit ${th.getCause}")
  } yield ()
  ```
  
  
  Awesome!  so we have composed ```transfer``` function and we were able to convert other type construct `F[]` to `\/`



