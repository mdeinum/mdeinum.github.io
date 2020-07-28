---
layout: post
title: Dependency Injection in Java
date: 2020-07-28
categories:
- Java
tags:
- configuration
- dependency injection
published: true
---

Before diving in lets take a look at 2 definitions of dependency injection (according to Wikipedia). 

> **Dependency Injection** is a technique in which an object receives other objects that it depends on. These other objects are called dependencies. 
> -- <cite><a href="https://en.wikipedia.org/wiki/Dependency_injection">Wikipedia</a></cite>

> **Dependency injection** is one form of the broader technique of [inversion of control](https://en.wikipedia.org/wiki/Inversion_of_control). A client who wants to call some services should not have to know how to construct those services.
> -- <cite><a href="https://en.wikipedia.org/wiki/Dependency_injection">Wikipedia</a></cite>

In short. when an object needs another object it should be handed to the object it needs.

Why would you need dependency injection? Lets take a look at the code for a transfer money system:

There is the `MoneyTransferService` (see [Listing 1](#listing1)) which has a single method to transfer money from one account to another. The business logic is implemented in a base class `AbstractMoneyTransferService` (see [Listing 2](#listing2)). 

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

**NOTE:** This base class is an implementation of a different inversion of control mechanism called [Template Method Pattern](https://en.wikipedia.org/wiki/Template_method_pattern)!

Now we need a class that provides the needed `AccountRepository` and `TransactionRepository` so the class can fulfill the `transfer` operation. The `SimpleMoneyTransferService` (see [Listing 3](#listing3)) is an implementation that doesn't use dependency injection but rather instantiates the needed classes by itself. 

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

To make a money transfer the class as shown in [Listing 4](#listing4) can be used. It will construct a `SimpleMoneyTransferService` and call the `transfer` method on it. This class is also going to be used, with some modifications, in the later samples. 

<a id="listing4"></a>

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

Running this class will transfer money from one account to another, this fact is stored as a `Transaction` more precise a `MoneyTransferTransaction`.

The problem with the code above is that the `SimpleMoneyTransferService` is now tied to our specific `Map` based implementations of the 2 repositories. It also needs to know that the `MapBasedAccountRepository` needs to be initialized by calling the `initialize` method. The class using the repository now needs to have intimate knowledge of how to construct a class and bootstrap it. Another issue is what if the implementation used a quite expensive resource like a `DataSource` or remote file source, you probably wanted to reuse a single shared instance in that case. 

Finally, these default implementations make it also quite hard to test, you could take a stab at it with something like [PowerMock](https://github.com/powermock/powermock) to replace the constructor or `final` fields but that would be quite a hack for a test.

## Dependency Injection
As mentioned in the introduction _dependency injection_ is a way of handling the dependencies of an object to the object in question. In the case of the sample above, we would hand the dependencies to both repositories to the `MoneyTransferService` instead of it being responsible for it. Injecting the dependencies can be done in 5 ways (or a combination thereof):

1. [Field inject injection](#field-injection)
2. [Constructor injection](#constructor-injection)
3. [Setter/Property injection](#setter-injection)
4. [Method injection](#method-injection)
5. [Interface based](#interface-injection)

<a id="field-injection"></a>
### Field Injection

Field injection is the practice of defining a field and with reflection setting the value of those fields. The `MoneyTransferService` as shown in [Listing 5](#listing5), has 2 fields to represent the needed dependencies. 

<a id="listing5"></a>

```java
class MoneyTransferService extends AbstractMoneyTransferService {

	private AccountRepository accountRepository;
	private TransactionRepository transactionRepository;

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
##### Listing 5: Field injection based `MoneyTransferService`

Now as the `MoneyTransferService` itself doesn't control those dependencies they need to be handed to the service by an external class. The dependency injection can be done by `name` or by `type`. When using type-based injection, the fields of the class would need to be retrieved, and see if there is an instance that matches the type or name. [Listing 6](#listing6) shows a helper class to do name or type-based field injection. 

<a id="listing6"></a>

```java
public class ReflectionUtil {

    private static final Logger log = LoggerFactory.getLogger(ReflectionUtil.class);

    public static void setFieldsByName(Object target, Map<String, Object> context) {
        Field[] fields = target.getClass().getDeclaredFields();
        Stream.of(fields)
                .filter(it -> Objects.isNull(getField(it, target)))
                .forEach(it -> {
                    String name = it.getName();
                    Object value = context.get(name);
                    if (value == null || !it.getType().isAssignableFrom(value.getClass())) {
                        String msg = String.format("Cannot inject '%s' into field '%s' of '%s'", value, name, target.getClass());
                        throw new IllegalStateException(msg);
                    } else {
                        setField(it, target, value);
                    }
                });
    }

    public static void setFieldsByType(Object target, Map<String, Object> context) {
        Field[] fields = target.getClass().getDeclaredFields();
        for (Object value : context.values()) {
            Stream.of(fields)
                    .filter(it -> it.getType().isAssignableFrom(value.getClass()))
                    .filter(it -> Objects.isNull(getField(it, target)))
                    .forEach(it -> setField(it, target, value));
        }
    }

    private static Object getField(Field field, Object target) {
        field.setAccessible(true);
        try {
            log.info("Getting field '{}' of '{}' '", field.getName(), target);
            return field.get(target);
        } catch (IllegalAccessException e) {
            var msg = String.format("Cannot get field: %s of: %s", field, target.getClass());
            throw new IllegalStateException(msg, e);
        }
    }

    private static void setField(Field field, Object target, Object value) {
        field.setAccessible(true);
        try {
            log.info("Setting field '{}' of '{}' to '{}'", field.getName(), target, value);
            field.set(target, value);
        } catch (IllegalAccessException e) {
            var msg = String.format("Cannot set field: %s with value: %s", field, value);
            throw new IllegalStateException(msg, e);
        }
    }
}
```
##### Listing 6: Field Injection Helper Class

When doing injection by name, the list of declared fields for a class is retrieved, and when the field on the target object, here the `MoneyTransferService` is being looked up in the context map and set as a value. Before the field can be set it needs to be made accessible (generally, those fields have the `private` access modifier). If the type doesn't match one cannot set the value, this is something you either need to check or let it blow up when setting an illegal type. 

When injecting by type, the values are injected by inspecting the type compatibility of the fields in the class and the values in the context map. 

The runner, as shown in [Listing 7](#listing7) can be used to do a money transfer. First, the needed dependencies are constructed and properly initialized, before injecting them (using reflection) into the `MoneyTransferService`. 

<a id="listing7"></a>

```java
public class TransferMoney {

    private static final Logger logger = LoggerFactory.getLogger(TransferMoney.class);

    public static void main(String[] args) {
        var accountRepository = new MapBasedAccountRepository();
        accountRepository.initialize();

        var transactionRepository = new MapBasedTransactionRepository();
        var service = new MoneyTransferService();


        var context = Map.of(
                "accountRepository", accountRepository,
                "transactionRepository", transactionRepository);
        setFieldsByName(service, context);
//        setFieldsByType(service, context);

        var transaction = service.transfer("123456", "654321", new BigDecimal("250.00"));

        logger.info("Money Transfered: {}", transaction);

    }
}
```
##### Listing 7: Field injection based runner

Although it works, it requires working with the Java Reflection API (which is not necessarily a bad thing!) but it kind of hides the needed dependencies from external uses. You need to intimately know the class to now its dependencies, instead of looking at a constructor or Javadoc. 

The advantage is that the `MoneyTransferService` now doesn't need to know what type of dependencies it needs and that it needs initializing. For a unit test you can now easily use a mocking library like [Mockito](https://mockito.org) to inject a mock (See [Listing 8](#listing8).

<a id="listing8"></a>

```java
class MoneyTransferServiceTest {

    private final MoneyTransferService service = new MoneyTransferService();
    private final AccountRepository mockAccountRepository = Mockito.mock(AccountRepository.class);
    private final TransactionRepository mockTransactionRepository = Mockito.mock(TransactionRepository.class);

    @BeforeEach
    public void setup() {
        Map<String, Object> context = Map.of(
                "accountRepository", mockAccountRepository,
                "transactionRepository", mockTransactionRepository);

        ReflectionUtil.setFieldsByName(service, context);
    }

    @Test
    public void transferMoney() {
        // Given
        var act1 = new Account("123456");
        act1.debit(BigDecimal.valueOf(1000L));
        when(mockAccountRepository.find("123456")).thenReturn(act1);
        when(mockAccountRepository.find("654321")).thenReturn(new Account("654321"));
        when(mockTransactionRepository.store(isA(MoneyTransferTransaction.class))).thenAnswer(AdditionalAnswers.returnsFirstArg());
        //When
        var transaction = service.transfer("123456", "654321", new BigDecimal("250.00"));
        //Then
        assertNotNull(transaction);
        verify(mockTransactionRepository, times(1)).store(isA(MoneyTransferTransaction.class));
    }
}
```
##### Listing 8: Field injection based test case with mock.

<a id="constructor-injection"></a>
### Constructor Injection

Constructor based injection is creating a constructor that accepts the dependencies. The main benefit of this is that it is clear upon construction which dependencies are needed by a class to operate (see [Listing 9](#listing9)).

<a id="listing9"></a>

```java
class MoneyTransferService extends AbstractMoneyTransferService {

	private final AccountRepository accountRepository;
	private final TransactionRepository transactionRepository;

	public MoneyTransferService(AccountRepository accountRepository, TransactionRepository transactionRepository) {
		super();
		this.accountRepository = accountRepository;
		this.transactionRepository = transactionRepository;
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
##### Listing 9: Construction injection based `MoneyTransferService`

To make a money transfer, the repositories need to be constructed before the service can be constructed with those dependencies (See [Listing 10](#listing10)).

<a id="listing10"></a>

```java
public class TransferMoney {

	private static final Logger logger = LoggerFactory.getLogger(TransferMoney.class);

	public static void main(String[] args) {
		var transactionRepository = new MapBasedTransactionRepository();

		var accountRepository = new MapBasedAccountRepository();
		accountRepository.initialize();

		var service = new MoneyTransferService(accountRepository, transactionRepository);

		var transaction = service.transfer("123456", "654321", new BigDecimal("250.00"));

		logger.info("Money Transfered: {}", transaction);
	}
}
```
##### Listing 10: Construction injection based runner

The usage of the code is nice and clean, no reflection is involved, just regular Java constructs. It is also not possible to construct an object in an invalid state because all the fields are initialized before use. You could add a check in the constructor that a dependency isn't allowed to be `null` to even prevent illegal use further. 

The same modification can be done for the test case (see [Listing 11](#listing11)).

<a id="listing11"></a>

```java
class MoneyTransferServiceTest {

    private final AccountRepository mockAccountRepository = Mockito.mock(AccountRepository.class);
    private final TransactionRepository mockTransactionRepository = Mockito.mock(TransactionRepository.class);
    private final MoneyTransferService service = new MoneyTransferService(mockAccountRepository, mockTransactionRepository);

    @Test
    public void transferMoney() {
        // Given
        Account act1 = new Account("123456");
        act1.debit(BigDecimal.valueOf(1000L));
        when(mockAccountRepository.find("123456")).thenReturn(act1);
        when(mockAccountRepository.find("654321")).thenReturn(new Account("654321"));
        when(mockTransactionRepository.store(isA(MoneyTransferTransaction.class))).thenAnswer(AdditionalAnswers.returnsFirstArg());
        //When
        var transaction = service.transfer("123456", "654321", new BigDecimal("250.00"));
        //Then
        assertNotNull(transaction);
        verify(mockTransactionRepository, times(1)).store(isA(MoneyTransferTransaction.class));
    }
}
```
##### Listing 11: Construction injection based test case

<a id="setter-injection"></a>
### Setter / Property Injection

A class has so-called setter methods to set the dependencies needed for an object instance. Generally, those setters follow the [Java Beans Specification](https://download.oracle.com/otndocs/jcp/7224-javabeans-1.01-fr-spec-oth-JSpec/). For the `MoneyTransferService` that would mean adding a `setAccountRepository` and `setTransactionRepository` method (see [Listing 12](#listing12)). Before the `MoneyTransferService` can be used the setters need to be called before invoking any logic on the (see [Listing 13](#listing13)).

<a id="listing12"></a>

```java
class MoneyTransferService extends AbstractMoneyTransferService {

	private AccountRepository accountRepository;
	private TransactionRepository transactionRepository;

	@Override
	protected AccountRepository getAccountRepository() {
		return this.accountRepository;
	}

	@Override
	protected TransactionRepository getTransactionRepository() {
		return this.transactionRepository;
	}

	public void setAccountRepository(AccountRepository accountRepository) {
		this.accountRepository = accountRepository;
	}

	public void setTransactionRepository(TransactionRepository transactionRepository) {
		this.transactionRepository = transactionRepository;
	}

}

```
##### Listing 12: Setter injection based `MoneyTransferService`

<a id="listing13"></a>

```java
public class TransferMoney {

	private static final Logger logger = LoggerFactory.getLogger(TransferMoney.class);

	public static void main(String[] args) {
		var transactionRepository = new MapBasedTransactionRepository();

		var accountRepository = new MapBasedAccountRepository();
		accountRepository.initialize();

		var service = new MoneyTransferService();
		service.setAccountRepository(accountRepository); // Inject dependency
		service.setTransactionRepository(transactionRepository); // Inject dependency

		var transaction = service.transfer("123456", "654321", new BigDecimal("250.00"));

		logger.info("Money Transfered: {}", transaction);
	}
}
```
##### Listing 13: Setter injection based runner

The test case can be modified as well (see [Listing 14](#listing14))

<a id="listing14"></a>

```java
class MoneyTransferServiceTest {

    private final AccountRepository mockAccountRepository = Mockito.mock(AccountRepository.class);
    private final TransactionRepository mockTransactionRepository = Mockito.mock(TransactionRepository.class);
    private final MoneyTransferService service = new MoneyTransferService();

    @BeforeEach
    public void setup() {
        service.setAccountRepository(this.mockAccountRepository);
        service.setTransactionRepository(this.mockTransactionRepository); 
    }

    @Test
    public void transferMoney() {
        // Given
        Account act1 = new Account("123456");
        act1.debit(BigDecimal.valueOf(1000L));
        when(mockAccountRepository.find("123456")).thenReturn(act1);
        when(mockAccountRepository.find("654321")).thenReturn(new Account("654321"));
        when(mockTransactionRepository.store(isA(MoneyTransferTransaction.class))).thenAnswer(AdditionalAnswers.returnsFirstArg());
        //When
        var transaction = service.transfer("123456", "654321", new BigDecimal("250.00"));
        //Then
        assertNotNull(transaction);
        verify(mockTransactionRepository, times(1)).store(isA(MoneyTransferTransaction.class));
    }
}
```
##### Listing 14: Setter injection based test case

The setup for the service is done in the `setup` method, the setters are called before a test method is invoked. This additional setup is now required because it isn't possible to set the mocks directly.

<a id="method-injection"></a>
### Method Injection
Method injection is similar to setter injection in that a method is being used to inject the dependencies. Where a setter accepts a single argument with method injection you can provide multiple arguments. For the `MoneyTransferService` instead of 2 setter methods we could provide 1 method used to inject the dependencies into the service (see [Listing 15](#listing15))

<a id="listing15"></a>

```java
class MoneyTransferService extends AbstractMoneyTransferService {

	private AccountRepository accountRepository;
	private TransactionRepository transactionRepository;

	@Override
	protected AccountRepository getAccountRepository() {
		return this.accountRepository;
	}

	@Override
	protected TransactionRepository getTransactionRepository() {
		return this.transactionRepository;
	}

	public void initialize(AccountRepository accountRepository, TransactionRepository transactionRepository) {
		this.accountRepository=accountRepository;
		this.transactionRepository=transactionRepository;
	}
}
```
##### Listing 15: Method injection based `MoneyTransferService`

<a id="listing16"></a>

```java
public class TransferMoney {

	private static final Logger logger = LoggerFactory.getLogger(TransferMoney.class);

	public static void main(String[] args) {
		var transactionRepository = new MapBasedTransactionRepository();

		var accountRepository = new MapBasedAccountRepository();
		accountRepository.initialize();

		var service = new MoneyTransferService();
		service.initialize(accountRepository,transactionRepository); // Inject dependencies

		var transaction = service.transfer("123456", "654321", new BigDecimal("250.00"));

		logger.info("Money Transfered: {}", transaction);
	}
}
```
##### Listing 16: Method injection based runner

To be able to run the test case the `initialize` method would need to be called as well before each test, see [Listing 17](#listing17)

<a id="listing17"></a>

```java
class MoneyTransferServiceTest {

    private final AccountRepository mockAccountRepository = Mockito.mock(AccountRepository.class);
    private final TransactionRepository mockTransactionRepository = Mockito.mock(TransactionRepository.class);
    private final MoneyTransferService service = new MoneyTransferService();

    @BeforeEach
    public void setup() {
        service.initialize(this.mockAccountRepository, this.mockTransactionRepository);
    }

    @Test
    public void transferMoney() {
        // Given
        Account act1 = new Account("123456");
        act1.debit(BigDecimal.valueOf(1000L));
        when(mockAccountRepository.find("123456")).thenReturn(act1);
        when(mockAccountRepository.find("654321")).thenReturn(new Account("654321"));
        when(mockTransactionRepository.store(isA(MoneyTransferTransaction.class))).thenAnswer(AdditionalAnswers.returnsFirstArg());
        //When
        var transaction = service.transfer("123456", "654321", new BigDecimal("250.00"));
        //Then
        assertNotNull(transaction);
        verify(mockTransactionRepository, times(1)).store(isA(MoneyTransferTransaction.class));
    }
}
```
##### Listing 17: Method injection based test case

<a id="interface-injection"></a>
### Interface-based Injection

Interface-based injection is based on interface and that the program knows what to do with it. In the money transfer sample 2 interfaces could be added to support injection of the 2 repositories (see [Listing 18](#listing18) )

<a id="listing18"></a>

```java
public interface AccountRepositoryAware {

    void setAccountRepository(AccountRepository repo);
}

public interface TransactionRepositoryAware {

    void setTransactionRepository(TransactionRepository repo);
}
```

##### Listing 18: Interfaces for injection

As can be seen in Listing 18, 2 interfaces define how to inject a certain dependency. To get the dependencies into the `MoneyTransferService` it would need to implement those 2 interfaces and we would need to check (in the injector) which dependencies it needs (see [Listing 19](#listing19) and [Listing 20](#listing20)). This style of dependency injection is often used when using IoC containers like [Spring](https://spring.io/projects/spring-framework) or [Guice](https://github.com/google/guice) when needing some special infrastructure components (like the Spring `ApplicationContext`).

<a id="listing19"></a>

```java
class MoneyTransferService extends AbstractMoneyTransferService implements AccountRepositoryAware, TransactionRepositoryAware  {

	private AccountRepository accountRepository;
	private TransactionRepository transactionRepository;

	@Override
	protected AccountRepository getAccountRepository() {
		return this.accountRepository;
	}

	@Override
	protected TransactionRepository getTransactionRepository() {
		return this.transactionRepository;
	}

	@Override
	public void setAccountRepository(AccountRepository accountRepository) {
		this.accountRepository = accountRepository;
	}

	@Override
	public void setTransactionRepository(TransactionRepository transactionRepository) {
		this.transactionRepository = transactionRepository;
	}
}
```
##### Listing 19: Interface injection based `MoneyTransferService`

The `MoneyTransferService` now implements the 2 interfaces and when we want to inject we could call the setters but there would probably be an injector or helper class checking the interfaces and act accordingly. 
<a id="listing20"></a>

```java
public class TransferMoney {

	private static final Logger logger = LoggerFactory.getLogger(TransferMoney.class);

	public static void main(String[] args) {
		var transactionRepository = new MapBasedTransactionRepository();

		var accountRepository = new MapBasedAccountRepository();
		accountRepository.initialize();

		var service = new MoneyTransferService();
        Injections.injeect(service, accountRepository, transactionRepository);

		var transaction = service.transfer("123456", "654321", new BigDecimal("250.00"));

		logger.info("Money Transfered: {}", transaction);
	}
}
```
##### Listing 20: Interface injection based runner


The `Injections` class ([Listing 21](#listing21) is a simple helper that will check if a bean implements a certain interface and inject the desired beans accordingly. Any class implementing one of these interfaces can use this method to get the dependencies injected.

<a id="listing21"></a>

```java
public abstract class Injections {

    private Injections() {}
    
    public static void inject(Object bean, AccountRepository accountRepository, TransactionRepository transactionRepository) {
        if (bean instanceof AccountRepositoryAware) {
            ((AccountRepositoryAware) bean).setAccountRepository(accountRepository);
        }

        if (bean instanceof TransactionRepositoryAware) {
            ((TransactionRepositoryAware) bean).setTransactionRepository(transactionRepository);
        }
    }
}
```
##### Listing 21: Helper class to do dependency injection

The test case can also use the `Injections` class to inject the mocks into the `MoneyTransferService` (see [Listing 22](#listing22)). 

<a id="listing22"></a>

```java
class MoneyTransferServiceTest {

    private final AccountRepository mockAccountRepository = Mockito.mock(AccountRepository.class);
    private final TransactionRepository mockTransactionRepository = Mockito.mock(TransactionRepository.class);
    private final MoneyTransferService service = new MoneyTransferService();

    @BeforeEach
    public void setup() {
        Injections.inject(service, mockAccountRepository, mockTransactionRepository);
    }

    @Test
    @DisplayName("Money Transfer - Setter Injection")
    public void transferMoney() {
        // Given
        var act1 = new Account("123456");
        act1.debit(BigDecimal.valueOf(1000L));
        when(mockAccountRepository.find("123456")).thenReturn(act1);
        when(mockAccountRepository.find("654321")).thenReturn(new Account("654321"));
        when(mockTransactionRepository.store(isA(MoneyTransferTransaction.class))).thenAnswer(AdditionalAnswers.returnsFirstArg());
        //When
        var transaction = service.transfer("123456", "654321", new BigDecimal("250.00"));
        //Then
        assertNotNull(transaction);
        verify(mockTransactionRepository, times(1)).store(isA(MoneyTransferTransaction.class));
    }
}
```
##### Listing 17: Interface injection based test case

# Summary

Dependency Injection can be done in different forms in your application and it doesn't have to rely on frameworks or features like Spring, Guice or CDI. You can start using it in your own applications right now. Different forms of dependency injection exists and have different usages. 

1. [Field inject injection](#field-injection)
2. [Constructor injection](#constructor-injection)
3. [Setter/Property injection](#setter-injection)
4. [Method injection](#method-injection)
5. [Interface based](#interface-injection)

Which one to use depends on the use case and if you are using a framework or not. However it it probably best to use [constructor-based](#constructor-injection) dependency injection. The others you might come across when using frameworks or features of JEE. 
