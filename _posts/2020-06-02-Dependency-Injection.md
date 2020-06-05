---
layout: post
title: Dependency Injection in Java
date: 2020-06-02
categories:
- Java
tags:
- configuration
published: false
---

Before diving in lets take a look 2 definitions of dependency injection (according to Wikipedia). 

> **Dependency Injection** is a technique in which an object receives other objects that it depends on. These other objects are called dependencies. 
> -- <cite><a href="https://en.wikipedia.org/wiki/Dependency_injection">Wikipedia</a></cite>

> **Dependency injection** is one form of the broader technique of [inversion of control](https://en.wikipedia.org/wiki/Inversion_of_control). A client who wants to call some services should not have to know how to construct those services.
> -- <cite><a href="https://en.wikipedia.org/wiki/Dependency_injection">Wikipedia</a></cite>

In short when an object needs another object it should be handed to the object it needs.

Why would you need dependency injection? Lets take a look at a code for a transfer money system:

There is the `MoneyTransferService` (see [Listing 1](#listing1)) which has a single method to transfer money from one account to another. The business logic is implemented in an abstract base class `AbstractMoneyTransferService` (see [Listing 2](#listing2)). 

<a id="listing1"></a>
```java
public interface MoneyTransferService {

	Transaction transfer(String source, String target, BigDecimal amount);
}
```
##### Listing 1: MoneyTransferService interface

<a id="listing2"></a>
```java
public abstract class AbstractMoneyTransferService implements MoneyTransferService {

	@Override
	public Transaction transfer(String source, String target, BigDecimal amount) {
		var src = getAccountRepository().find(source);
		var dst = getAccountRepository().find(target);

		src.credit(amount);
		dst.debit(amount);

		var transaction = new MoneyTransferTransaction(src, dst, amount);
		return getTransactionRepository().store(transaction);
	}

	protected abstract AccountRepository getAccountRepository();

	protected abstract TransactionRepository getTransactionRepository();
}
```
##### Listing 2: Abstract class implementing the business logic

**NOTE:** This base class is actually an implementation of a different inversion of control mechanism called [Template Method Pattern](https://en.wikipedia.org/wiki/Template_method_pattern)!

Now we need a class that provides the needed `AccountRepository` and `TransactionRepository` so the class can fullfil the `transfer` operation. The `SimpleMoneyTransferService` (see [Listing 3](#listing3)) is an implementation that doesn't use dependency injection but rather instantiates the needed classes by itself. 

<a id="listing3"></a>
```java
public class SimpleMoneyTransferService extends AbstractMoneyTransferService {

	private final AccountRepository accountRepository;
	private final TransactionRepository transactionRepository;

	public SimpleMoneyTransferServiceImpl() {
		super();
		this.accountRepository = new MapBasedAccountRepository() {{initialize();}};
		this.transactionRepository = new MapBasedTransactionRepository();

	}

    @Override
	protected AccountRepository getAccountRepository() {
		return this.accountRepository;
	}

	@Override
	protected TransactionRepository getTransactionRepository() {
		return this.transactionRepository;
	}
}
```
##### Listing 3: Simple no dependency injection based implementation

The problem with the code above is that the `SimpleMoneyTransferService` is now tied to our specific `Map` based implementations of the 2 repositories. It also needs to know that the `MapBasedAccountRepository` needs to be initialized by calling the `initialize` method. The fact that it creates these default implementations makes it also quite hard to test, you could take a stab at it with something like `PowerMock` to replace the constructor but that would be quite a hack for a test.

To make a money transfer the following class can be used

```java
public class TransferMoney {

	private static final Logger logger = LoggerFactory.getLogger(TransferMoney.class);

	public static void main(String[] args) {

		var service = new SimpleMoneyTransferService();
		var transaction = service.transfer("123456", "654321", new BigDecimal("250.00"));
		logger.info("Money Transfered: {}", transaction);
	}
}
```
##### Listing 4: No dependency injection based runner

Running this class will transfer money from one account to another, this fact is stored as a `Transaction` more precise a `MoneyTransferTransaction`. Running it is simple enough, but trying to create a test can be cumbersome. 

```java
```
##### Listing 5: No dependency injection based runner






