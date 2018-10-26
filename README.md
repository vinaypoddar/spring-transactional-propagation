# spring-transactional-propagation
In this tutorial you will learn about the transaction propagation behaviors provided by the Spring framework.

## Introduction
While dealing with Spring managed transactions the developer is able to specify how the transactions should behave in terms of propagation. In other words the developer has the ability to decide how the business methods should be encapsulated in both logical or physical transactions. Methods from distinct Spring beans may be executed in the same transaction scope or actually being spanned across multiple nested transactions. This may lead to details like how does the inner transaction outcome result affects the outer transactions. We will see the different propagation behaviors provided by Spring in the next sections.

This tutorial will only focus in transaction propagation behavior. For other details about Spring transactions you may refer to other tutorials on this website or the official Spring documentation at SpringSource website.

## REQUIRED behavior
Spring ***REQUIRED*** behavior means that the same transaction will be used if there is an already opened transaction in the current bean method execution context. If there is no existing transaction the Spring container will create a new one. If multiple methods configured as ***REQUIRED*** behavior are called in a nested way they will be assigned ***distinct logical transactions*** but they will all share the ***same physical transaction***. In short this means that if an inner method causes a transaction to rollback, the outer method will fail to commit and will also rollback the transaction. Let's see an example:


```java
// Outer Bean
@Autowired
private TestDAO testDAO;

@Autowired
private InnerBean innerBean;

@Override
@Transactional(propagation=Propagation.REQUIRED)
public void testRequired(User user) {
  testDAO.insertUser(user);
  try{
    innerBean.testRequired();
  } catch(RuntimeException e){
    // handle exception
  }
}
```

```java
// Inner bean
@Override
@Transactional(propagation=Propagation.REQUIRED)
public void testRequired() {
  throw new RuntimeException("Rollback this transaction!");
}
```
Note that the inner method throws a ***RuntimeException*** and is annotated with ***REQUIRED*** behavior. This means that it will use the ***same*** transaction as the outer bean, so the outer transaction will fail to commit and will also rollback.


## REQUIRES_NEW behavior
***REQUIRES_NEW*** behavior means that a new physical transaction will always be created by the container. In other words the inner transaction may commit or rollback independently of the outer transaction, i.e. the outer transaction will not be affected by the inner transaction result: they will run in ***distinct physical transactions***.

```java
// Outer bean
@Autowired
private TestDAO testDAO;

@Autowired
private InnerBean innerBean;

@Override
@Transactional(propagation=Propagation.REQUIRED)
public void testRequiresNew(User user) {
  testDAO.insertUser(user);
  try{
    innerBean.testRequiresNew();
  } catch(RuntimeException e){
    // handle exception
  }
}
```
```java
// Inner bean
@Override
@Transactional(propagation=Propagation.REQUIRES_NEW)
public void testRequiresNew() {
  throw new RuntimeException("Rollback this transaction!");
}
```
The inner method is annotated with ***REQUIRES_NEW*** and throws a ***RuntimeException*** so it will set its transaction to rollback but will not affect the outer transaction. The outer transaction is paused when the inner transaction starts and then resumes after the inner transaction is concluded. They run independently of each other so the outer transaction may commit successfully.


## NESTED behavior
The ***NESTED*** behavior makes nested Spring transactions to use the same physical transaction but sets savepoints between nested invocations so inner transactions may also rollback independently of outer transactions. This may be familiar to JDBC aware developers as the savepoints are achieved with JDBC savepoints, so this behavior should only be used with Spring JDBC managed transactions (Spring JDBC transactions example).


## MANDATORY behavior
The ***MANDATORY*** behavior states that an existing opened transaction must already exist. If not an exception will be thrown by the container.


## NEVER behavior
The ***NEVER*** behavior states that an existing opened transaction must ***not*** already exist. If a transaction exists an exception will be thrown by the container.


## NOT_SUPPORTED behavior
The ***NOT_SUPPORTED*** behavior will execute outside of the scope of any transaction. If an opened transaction already exists it will be paused.


## SUPPORTS behavior
The ***SUPPORTS*** behavior will execute in the scope of a transaction if an opened transaction already exists. If there isn't an already opened transaction the method will execute anyway but in a non-transactional way.
