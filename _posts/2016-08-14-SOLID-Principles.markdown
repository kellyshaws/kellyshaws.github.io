---
layout: post
title:  "S.O.L.I.D. Principles"
date:   2016-08-14 09:00:15
description: "S.O.L.I.D. Principles"
permalink: post/SOLID-Principles
disqus:
  id: SOLID-Principles
categories:
- blog
- Design pattern
---

##Definition##

S.O.L.I.D. - it is prinpliples of class design, which facilitate the work of the programmer.All programmers must know how to use them. Using of this principles makes it easier to support and expand project.<br>

##The single responsibility principle##

Definition: should not be more than one reason to change the class <br>

What is the reason for the change logic of the class? As example, it can be changing the relationship between the classes, the introduction of the new requirements, or the abolition of the old. If the object has a lot of responsibility, it will change very often. Thus, if a class has more than one responsibility, it leads to brittleness and design errors in unexpected places when the code changes.<br>

Scenarios where you can find a violation of this principle very much. One of the most popular is “God object”. In object-oriented programming, a god object is an object that knows too much or does too much. The god object is an example of an anti-pattern. Following class is an example of such object:<br>

```
class Helper {
    
    public void sendMessageToUser(Message message, User user) {
        // sends email to user
    }
    
    public void changeUserPassword(User user, String newPassword) {
        // saves new password
    }
    
    public void addNewOrder(User user, Order order) {
        // adds new order to the user
    }
    
    public List<Order> getCurrentOrders() {
        // gets current user orders
    }
    
}
```

This class is really monster object! This class can do a lot of operations which is not related to each other. It seems that the boundaries of responsibility he has no. It turns out that this class will often change their behavior, making it difficult to test it and test components that use it. This approach will reduce the efficiency of the system and increase the cost of its maintenance.<br>

Desision<br>

The solution is to split the class on the basis of the uniqueness of responsibility: one class per responsibility.<br>

```
class MessageService {
    public void sendMessageToUser(Message message, User user) {
        // sends email to user
    }
}

class UserService {
    public void changeUserPassword(User user, String newPassword) {
        // saves new password
    }
}

class OrderManagementService {
    public void addNewOrder(User user, Order order) {
        // adds new order to the user
    }

    public List<Order> getCurrentOrders() {
        // gets current user orders
    }
}
```

##The open/closed principle##

Definition: software entities (classes, modules, functions, etc.) should be open for extension, but closed for modifications.<br>

As we know software projects are constantly changing within your life. Changes may occur, for example, due to new customer requirements or revising old ones. In the end, you need to change the code in accordance with the current situation. The open/closed principle just gives an understanding of how to be sufficiently flexible in an ever-changing requirements.<br>

The simplest example of a violation of the open/closed principle - the use of specific objects without abstractions. Suppose that we have MailService object. For logging the actions it uses Logger. It records information in a text file.<br>

```
class Logger {
    public void log(String logText, LogLevel level) {
        
    }
}
class MailService {
    private Logger logger;

    public MailService() {
        logger = new Logger();
    }

    public void sendMessage(String message) {
        logger.log("Message sended", LogLevel.DEBUG);
    }
}
```

This design is quite viable until we decide to record MailService log into the database. To do this we need to create a class that will record all the logs in the database:<br>

```
class DatabaseLogger {
    public void log(String logText, LogLevel level) {

    }
}
```

And now the most interesting. We must change MailService class due to changed business requirements:<br>

```
class MailService {
    private DatabaseLogger logger;

    public MailService() {
        logger = new DatabaseLogger();
    }

    public void sendMessage(String message) {
        logger.log("Message sended", LogLevel.DEBUG);
    }
}
```

But according to the principle of single responsibility MailService not responsible for logging, why change came to this class? Because it violated our open/closed principle. MailService not closed for modification.<br>

Decision<br>

To solve this problem we can use abstraction. We can user common interface for any logger.<br>

```
interface Logger {
    void log(String logText, LogLevel level);
}

class FileLogger implements Logger {

    @Override
    public void log(String logText, LogLevel level) {

    }

}

class DatabaseLogger implements Logger {

    @Override
    public void log(String logText, LogLevel level) {

    }

}

class MailService {
    private Logger logger;

    public MailService(Logger logger) {
        this.logger = logger;
    }

    public void sendMessage(String message) {
        logger.log("Message sended", LogLevel.DEBUG);
    }
}
```

Now we can change logger logic without changin of MailService. There are many design patterns that help us to extend code without changing it. For instance the Decorator pattern help us to follow Open Close principle. Also the Factory Method or the Observer pattern might be used to design an application easy to change with minimum changes in the existing code.<br>

##The Liskov Substitution Principle##

Definition: Derived classes must be substitutable for their base classes. Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it.<br>

The example of violation of LSP is incorrect inheritance. For instance, we want to make DoubleList which inserts two times.<br>

```
class DoubleList<T> extends ArrayList<T> {
    
    @Override
    public boolean add(T t) {
        return super.add(t) && super.add(t);
    }
    
}
```

But if client of this class wants to work with this class using interface List, he will be surprised:<br>

```
@Test
public void testArrayList() {
    List<Integer> list = new ArrayList<>();
    list.add(2);
    assertEquals(1, list.size());
}

@Test
public void testDoubleList() {
    List<Integer> list = new DoubleList<>();
    list.add(2);
    assertEquals(1, list.size()); // test fails
}
```

Behaviour of DoubleList is different from any implementation of List interface. It is a violation of LSP. So, to avoid unexpected behaviour, we need to make our own interface, as example DoubleList. This interface will be implemented by any class with behaviour in which elements will be doubled.<br>

##The Interface Segregation Principle##

Definition: clients should not be forced to depend upon interfaces that they don’t use.<br>

Sometimes, there are the libraries which are very useful for us. We can use such libraries with its built in classes. Such systems have some level of abstraction. So, we can use our own implementations of this abstraction. But if we want to extend our application adding own class that contains only some of the methods of the original system, we are forced to implement the full interface and to write some dummy methods. Such an interface is named fat interface or polluted interface. Having an interface pollution is not a good solution and might induce inappropriate behavior in the system. Example of fat interface:<br>

```
interface Worker {
    void work();
    void eat();
}

class SimpleWorker implements Worker {
    public void work() {
	// working
    }
    public void eat() {
	// eating in launch break
    }
}

class SuperWorker implements Worker {
    public void work() {
	// working much more
    }

    public void eat() {
	// eating in launch break
    }
}

class Robot implements Worker {
    public void work() {
       	// working
    }
    
    public void eat() {
        throw new UnsupportedOperationException();
    }
}

class Manager {
    private List<Worker> workers;
    
    // ...
    
    public void manage() {
	workers.foreach(w -> w.work());
    }
	
    public void launch() {
	workers.foreach(w -> w.eat()); // throws exception if workers list contains Robot object
    }
}
```

According to the Interface Segregation Principle, a flexible design will not have polluted interfaces. In our case the IWorker interface should be split in different interfaces.<br>

```
interface Workable {
    void work();
}

interface Feedable {
    void eat();
}

class Worker implements Workable, Feedable {
    public void work() {
	// working
    }

    public void eat() {
	// eating in launch break
    }
}

class Robot implements Workable {
    public void work() {
	// working
    }
}

class SuperWorker implements Workable, Feedable {
    public void work() {
	// working much more
    }

    public void eat() {
	// eating in launch break
    }
}

class Manager {
    private List<Workable> workers;
    private List<Feedable> feedables;

    // ...
    
    public void manage() {
	workers.foreach(w -> w.work());
    }
    
    public void launch() {
	feedables.foreach(f -> f.eat());
    }
}
```

Adapter pattern can be used to segrate already fat interfaces.<br>

##The Dependency Inversion Principle##

Definitions: A. High-level modules should not depend on low-level modules. Both should depend on abstractions. B. Abstractions should not depend on details. Details should depend on abstractions.<br>

Example of this principle is MailService from code above (Single Responsiblity):<br>

```
class MailService {
    private Logger logger;

    public MailService(Logger logger) {
        this.logger = logger;
    }

    public void sendMessage(String message) {
        logger.log("Message sended", LogLevel.DEBUG);
    }
}
```

The only difference is that we did class extension earlier, but now we are doing an inversion of control. As you remember, class MailService had depended on Logger class. But now it depends on interface Logger and we can use different implementations of this interface. It called Dependency Inversion.<br>

When this principle is applied it means the high level classes are not working directly with low level classes, they are using interfaces as an abstract layer. In this case instantiation of new low level objects inside the high level classes(if necessary) can not be done using the operator new. Instead, some of the Creational design patterns can be used, such as Factory Method, Abstract Factory, Prototype.<br>
