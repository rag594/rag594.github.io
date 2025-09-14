+++
title = "Dissecting Springs @Transactional Annotation"
date = "2025-09-13"
description = "How does Springs @Transactional annotation works and gotchas"
[taxonomies]
tags = ["spring", "annotations", "transactional", "proxy", "aop"]
+++

In Spring we widely use **@Transactional** annotation for executing bunch of stuff wrapped up in a DB transaction.
Spring makes it easier for us to leverage AOP(Aspect Oriented programming) and does not pollute our business logic with DB Transaction, security, caching, logging etc. Before diving into the nitty-gritties of **@Transactional**, lets start with what are proxy objects and how does Spring makes use of it for **@Transactional**

## What is a proxy and how does Spring uses it under the hood?

Proxy acts like a substitute for the given object. It acts as a middleman and when any client calls a method on an object,
proxy object appears and acts on behalf of that object and performs any kind of action before/after method invocation.

In other programming lanuages like golang(we have hooks in GORM - [https://gorm.io/docs/hooks.html](https://gorm.io/docs/hooks.html)) which behaves or act in a similar way. Below is a simple example.

Lets say I have a Service interface which is implemented by a `DummyService`. `DummyService` simply just does a simple work and logs `Work is in progress`. What if there are actions I want to take before doing a work and after doing a work which does not have any relation with the actual work or the business logic??.

```java
public interface Service {
    void doWork();
}
```

```java
public class DummyService implements Service {
    @Override
    public void doWork() {
        System.out.println("Work is in progress");
    }
}
```

Here comes the proxy for the rescue which helps in performing the before and after work.

```java
public static void main(String[] args) {
	// 1. Create a dummy Service
	Service realDummyService = new DummyService();

	// 2. create a proxy and wrap the business logic/method invocation inside it.
	// Here before invoking the actual business logic we perform the before work
	// and after method invocation, perform the after work
	Service proxyDummyService = (Service) Proxy.newProxyInstance(
			realDummyService.getClass().getClassLoader(),
			new Class[]{Service.class},
			(Object proxyObj, Method method, Object[] methodArgs) -> {
				// Wrapping up the business logic inside proxy
				System.out.println("Before " + method.getName());
				Object result = method.invoke(realDummyService, methodArgs);
				System.out.println("After " + method.getName());
				return result;
			}
	);

	// 3. invoke the method via proxy and not the actual object
	proxyDummyService.doWork();
}
```

Below is the output of how proxy works

```shell
Before doWork
Work is in progress
After doWork
```

## How does Springs **@Transactional** Works ?

Springs **@Transactional** simply creates a database transaction for bunch of stuff inside the method annotated with it. **@Transactional** uses proxy objects to wrap up the business logic inside transactions.

Lets consider an example to understand.We have `UserService` which in turn uses `UserRepository` to persist.

```java
// UserService.java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public void doSomething(String name, String email) {
        createUser(name, email);
    }

    @Transactional
    public void createUser(String name, String email) {
        User user = new User(name, email);
        System.out.println("Transaction active? " +
                org.springframework.transaction.support.TransactionSynchronizationManager.isActualTransactionActive());
        this.userRepository.save(user);
    }
}
```

```java
// UserRepository.java
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Integer> {
}
```


### When createUser is invoked(Directing invoking method annotated with **@Transactional**)

When **createUser** is invoked directly, proxy object which was created for `UserService` is used and wraps the actual business logic with **TransactionInterceptor** (an advice which gets invoked by proxy and starts the transaction).

![createUser](/spring-direct-invoking-transaction.png)

### When doSomething is invoked(Indirect invoking method annotated with **@Transactional** within same bean)

When **doSomething** is invoked, proxy does not get involved here as we invoke `createUser` it gets invoked from the same class and target object uses `this` to invoke `createUser`. In this case proxy object of `UserService` does not wrap the business logic with **TransactionInterceptor** and it does not start a transaction.

```java
 public void doSomething(String name, String email) {
        createUser(name, email);
    }
```

Here only `UserRepository` proxy object which was created is used and wraps the actual business logic(save) with **TransactionInterceptor** (an advice which gets invoked by proxy and starts the transaction).

![doSomething](/spring-indirect-invoking-transaction.png)