# Classes, Objects & Data Containers

When it comes to working with classes and objects and writing clean classes and objects, there are a couple of rules and concepts you should be aware of.

1. We can differentiate between objects and data containers / data structures
2. Consider using Polymorphism
3. Classes should be small
4. Classes should have a high cohesion
5. Respect the "Law of Demeter"
6. Write SOLID classes

---

## Objects vs Data Containers / Data Structures

Classes are blueprints for objects.

There also are programming languages where you can create objects without (actively) using classes - for example JavaScript. And there are languages where you MUST use classes for pretty much everything (e.g. Java).

Either way, objects allow you to group related data (properties) and functionalities (methods) together. And objects typically expose a public API of methods which can be used anywhere in your code to interact with these objects.

For example:

```javascript
customer.message(someMessage);
```

Even though we always work with objects, we can think of them as either real objects, which provide behavior through a public API, or simple data holders, which mainly store data.

A data container is really just that - an object which holds a bunch of data.

Here's an example:

```javascript
class UserData {
  public name: string;
  public age: number;
}

const userData = new UserData();
userData.name = 'Sepehr';
userData.age = 22;
```

This class has no methods and both properties are exposed publicly.

An object on the other hand hides its data from the public and instead exposes a public API in the form of methods:

```typescript
class User {
  private name: string;
  private age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  greet() {
    console.log(`Hi! I'm ${this.name} and I'm ${this.age} years old.`);
  }
}

const userData = new UserData('Sepehr', 22);
userData.greet();
```

Both are absolutely valid types of objects - it's just important to use the right kind of object for the right job. And you should avoid mixing the types.

Why does this matter?

When writing clean code, we shouldn't try to access the internals of an object. This can cause problems if the object changes its internal structure and our code stops working.

In addition, it makes code harder to read if we have to figure out what the properties of other classes mean and how their values should be used.

Calling something like `greet()` on the other hand is straightforward.

Of course, when we use a data container where a data container is needed, that is completely fine:

```javascript
function validateInput(credentials) {
  if (!isEmail(credentials.email) || !isPassword(credentials.password)) {
    // ...
  }
}
```

In this example, it's clear that `credentials` just holds some data which we can extract and work with.

---

## Polymorphism

Polymorphism is a fancy term but in the end it just means that you re-use code (e.g. call the same method) multiple times but that the code will do different things, depending on the object type.

It’s a powerful concept that can help us avoid duplicating code.

Here's an example that does not use polymorphism:

```typescript
class Delivery {
  private purchase: Purchase;

  constructor(purchase: Purchase) {
    this.purchase = purchase;
  }

  deliverProduct() {
    if (this.purchase.deliveryType == 'express') {
      Logistics.issueExpressDelivery(this.purchase.product);
    } else if (this.purchase.deliveryType == 'insured') {
      Logistics.issueInsuredDelivery(this.purchase.product);
    } else {
      Logistics.issueStandardDelivery(this.purchase.product);
    }
  }

  trackProduct() {
    if (this.purchase.deliveryType == 'express') {
      Logistics.trackExpressDelivery(this.purchase.product);
    } else if (this.purchase.deliveryType == 'insured') {
      Logistics.trackInsuredDelivery(this.purchase.product);
    } else {
      Logistics.trackStandardDelivery(this.purchase.product);
    }
  }
}
```

The problem with this code is that we have the same `if` checks with different logic inside of the `if` statements. So it's some code duplication but not everything is duplicated.

And we need these different cases because a product can of course be sent and tracked (and maybe we could even come up with additional methods).

Here's how this issue could be solved in a polymorphic way:

```typescript
abstract class Delivery {
  protected purchase: Purchase;

  constructor(purchase: Purchase) {
    this.purchase = purchase;
  }

  abstract deliverProduct(): void; 
  abstract trackProduct(): void;
}

class ExpressDelivery extends Delivery {
  deliverProduct() {
    Logistics.issueExpressDelivery(this.purchase.product);
  }

  trackProduct() {
    Logistics.trackExpressDelivery(this.purchase.product);
  }
}

class InsuredDelivery extends Delivery {
  deliverProduct() {
    Logistics.issueInsuredDelivery(this.purchase.product);
  }

  trackProduct() {
    Logistics.trackInsuredDelivery(this.purchase.product);
  }
}

class StandardDelivery extends Delivery {
  deliverProduct() {
    Logistics.issueStandardDelivery(this.purchase.product);
  }

  trackProduct() {
    Logistics.trackStandardDelivery(this.purchase.product);
  }
}
```

Above, there are three specific classes (`ExpressDelivery` etc.) which all inherit from the `Delivery` base class and share the same functionalities.

All these specific classes implement the same `deliverProduct()` and `trackProduct()` methods - though the exact code in these methods differs.

### That's the idea behind polymorphism!

We can now write a factory function or class method that creates instances of these classes when needed. It's the factory function which holds the `if` check from above.

Hence we only need the check once instead of once per method.

```javascript
function createDelivery(purchase) {
  if (purchase.deliveryType === 'express') {
    return new ExpressDelivery(purchase);
  } else if (purchase.deliveryType === 'insured') {
    return new InsuredDelivery(purchase);
  } else {
    return new StandardDelivery(purchase);
  }
}
```

You can now always run code like this and the exact result will depend on the `deliveryType` used for creating the objects:

```javascript
const expressDelivery = createDelivery({ deliveryType: 'express', ... });
expressDelivery.deliverProduct();
```

---

## Classes Should Be Small

Just like functions, classes should be small. They should also focus on one thing. For classes, this means that all their tasks should be related to the same object.

A `User` class might for example have a `login()` method. It might have a couple of other methods that do user-typical things but a `refund()` method would be strange.

`refund()` sounds more like a payment-related method so it would make more sense in a `Payment` class for example.

Therefore, a `User` class that also handles payments would be too big, even if it only contains a few lines of code.

The size of a class is therefore defined by its number of responsibilities. And clean classes should only have one responsibility (also see **"Single Responsibility Principle"**, further down below).

The responsibility of a `User` class should be to handle user-related tasks, for example `login()`, `logout()` etc.

---

## Classes & Cohesion

Clean classes have high cohesion.

Cohesion means: how much do a class's methods actually use that class's own properties?

Say a class has 2 properties and 3 methods. If every method uses both properties, that's the highest cohesion possible. If none of the methods touch the properties, the class has no cohesion at all — the properties are just sitting there, unused.

Low cohesion is a warning sign. It usually means the class should just be a plain data container instead of an object with methods, since its methods aren't really working with its own data.

You won't always hit perfect cohesion, and that's fine. Aim for high, not perfect.

When you notice cohesion dropping in a class, that's your signal to split it into smaller, more focused classes. So caring about cohesion naturally leads you to smaller classes, and smaller classes are easier to work with.

---

## The Law Of Demeter

When working with objects, the following code is considered to be bad / not clean:

```javascript
this.customer.lastPurchase.date;
```

This code breaks the Law of Demeter, which says you shouldn't access an object's internals through another object.

This is also called the "principle of least knowledge".

A method is only allowed to directly touch:
- itself (`this`)
- objects stored directly in its own properties
- objects passed in as parameters
- objects it creates itself

In this case, `lastPurchase` is a property of the `customer` object. And `customer` itself is a property of the class the method belongs to. So `lastPurchase` is a separate object, maybe an instance of some `Purchase` class — and that class has its own `date` property.

**Side-note:** Here I'm directly accessing properties, but it works the same way with getters (`customer.getLastPurchase().getDate()`).
Accessing this `date` property through `lastPurchase` **violates** the Law of Demeter.

The problem with code like this is that you can end up with very long statements, full of "chaining" (`.something.somethingElse.more()` is called "chaining").

This creates strong dependencies and whenever the `Purchase` class would change (e.g. `date` is renamed to `paymentDate`) all code snippets that look like above needs to be updated.

The code above would be better implemented like this:

```javascript
this.customer.getLastPurchaseDate();
```

Now the extra dependency on `lastPurchase.date` is removed.

Even this solution isn't ideal — it's better to "tell, don't ask". In other words, it depends on what you actually needed to do with that date.

Let's say you needed the date to send the purchased products to the customer.

```javascript
const date = this.customer.lastPurchase.date;
this.warehouse.deliverPurchasesByDate(this.customer, date);
```

Instead of getting the date and then checking for the products, the following code would be an even better way of avoiding the "Law of Demeter" violation:

```javascript
this.warehouse.deliverPurchase(customer.lastPurchase);
```

As you can see, it's not just about moving code around — sometimes a different solution (working with different objects entirely) is the cleanest way of getting something done.

It's also important to note that the "Law of Demeter" really only applies to property / attribute chaining.

**What about nested objects from an API response?**

The rule above is meant for objects that have behavior, not for plain data. An API response is just data (a DTO), so accessing something like `response.customer.lastPurchase.date` isn't really a "Law of Demeter" violation the same way `this.customer.lastPurchase.date` is on a domain object.

The real problem with API responses is different: your code becomes tightly coupled to the exact shape the backend sends. If the backend renames `date` to `paymentDate`, or restructures the nesting, every place that reads `response.customer.lastPurchase.date` breaks.

The fix is to map the raw response into your own model right where you receive it, so the rest of your code never has to touch the raw nested shape:

```javascript
function getCustomer(response) {
  return {
    lastPurchaseDate: response.data?.customer?.lastPurchase?.date ?? null,
  };
}
```

Now the rest of the app just uses `customer.lastPurchaseDate`, and only this one function needs to change if the API shape changes.
The following code would be fine (as long as these methods are real methods and not just getters wrapped around properties):

```javascript
this.customer.sendMessage(message).retry(2);
```

Why would this be okay?

Because now we're using the public interfaces of various classes instead of reaching into their internal data.

If you're only working with a couple of data containers, chaining their properties together does not violate the "Law of Demeter". A data container's only job is to hold and expose data — it has no logic to protect, so there's nothing being "reached through" in the way LoD warns against.

```javascript
// order and address are just data containers, nothing more
order.address.city;
```

---

## SOLID Classes

When it comes to writing clean classes, there are a couple of helpful rules, laws, and concepts (e.g. the "Law of Demeter", see above). But it's the SOLID principles that are most often referenced.

Below, all five principles are explained and put into the context of clean code.

---

### S: Single-Responsibility Principle (SRP)

> A class should only have a single responsibility - it should only change for this one responsibility.

The SRP simply states that a class should focus on one core responsibility. Changes to the class should only happen when that responsibility itself needs to change.

If a class needs to change because of different responsibilities, it's too big and should be split into multiple smaller classes.

The SRP is an important principle when it comes to writing clean code, since it enforces smaller, and therefore more readable, classes.

Here's an example for a class that violates the SRP:

```javascript
class ReportDocument {
  generateReport(data) {
    // ...
  }

  createPDF(report) {
    // ...
  }
}
```

The actual code in the methods is missing, because it's not needed - it's the structure and API of the class which matters here.

This `ReportDocument` class violates the SRP because it needs to change because of at least two reasons:
- If anything related to how a report is generated changes
- If anything related to how a report is printed as a PDF changes

It's fair to assume that in most applications, these two features aren't directly connected. The people who decide how the PDF should look aren't necessarily the same people who decide how the report is generated.

This class would therefore better be split - e.g. into `Report` and `PDFPrinter` classes.

The SRP is an important principle for writing clean code, because it typically leads to smaller, more focused classes — which is exactly what we want from a clean code perspective!

---

### O: Open-Closed Principle (OCP)

> A class should be open for extension but closed for modification.

While the sentence above might sound confusing, this is actually a straightforward principle.

Once you've decided how a class should look (i.e. its public API / methods), you should "close" it for modification — meaning the code no longer changes.

Of course, that's not entirely true. You should still keep working on the class, especially to fix errors.

But you shouldn't be editing the class all the time — whether that's to add new features, or just to handle slightly different variations of a feature it already supports.

Here's an example of a class that violates the OCP:

```javascript
class Printer {
  printPDF(data) {
    // ...
  }

  printWebDocument(data) {
    // ...
  }

  printPage(data) {
    // ...
  }

  verifyData(data) {
    // ...
  }
}
```

This `Printer` class has different methods for printing a web document (e.g. generating a HTML file), a "normal" page and a PDF.

Whenever we add a new type of document (e.g. a Word or Excel document), we need to add a new method — and that usually means duplicating a lot of similar code across methods.

This problem might sound familiar - we solved it with "Polymorphism" earlier!

And indeed, that's how we can fix the issue here as well — by following the OCP.

This `Printer` class is currently not closed because we need to come back to it and edit it whenever a new type of document should be printed.

To follow the OCP, we should close it and instead extend it whenever we want to add a new type of document.

```typescript
interface Printer {
  print(data: any);
}

class PrinterImplementation {
  verifyData(data: any) {
    // ...
  }
}

class WebPrinter extends PrinterImplementation implements Printer {
  print(data: any) {
    // print web document
  }
}

class PDFPrinter extends PrinterImplementation implements Printer {
  print(data: any) {
    // print PDF document
  }
}

class PagePrinter extends PrinterImplementation implements Printer {
  print(data: any) {
    // print real page
  }
}
```

This snippet shows how we could fix the issue. Now, the base class `PrinterImplementation` (named like this because we have an interface named `Printer`) is closed. It doesn't need to change just because we want to support a new type of document.

Instead, we extend it - we add new subclasses which are based on `PrinterImplementation` (which adds common functionality like verification) and implement `Printer` (which forces all implementing classes to have a `print()` method).

The OCP is an important principle when it comes to writing clean code. Because it typically leads to smaller and more focused classes - which is in line with what we want for our classes from a clean code perspective!

---

### L: Liskov Substitution Principle (LSP)

> Objects in a program should be replaceable with instances of their subtypes.

The above sentence should be quite clear. Here's an example of a base class being replaced with a subclass of it:

```javascript
class Car {
  drive() {
    console.log('Driving...');
  }
}

class SportsCar extends Car {
  boost() {
    console.log('Boosting...');
  }
}

const car = new Car();
car.drive();

// We can also replace Car() with SportsCar()
const sportsCar = new SportsCar();
sportsCar.drive();
```

In this example, `Car` is the superclass and `SportsCar` is the subclass. We can replace a `Car` with a `SportsCar` without changing our code.

Now, let's add a new subclass which actually violates the principle:

```javascript
class ToyCar extends Car {
  // Problem: Can't actually be driven on the road!
}
```

We can't use `ToyCar` in place of `Car`, because a toy car can't actually drive! As soon as we have a case like this, we've violated the LSP.

A better way of modelling our data then would be to do it like this:

```javascript
class Car { }

class DrivableCar extends Car {
  drive() {
    console.log('Driving...');
  }
}

class SportsCar extends DrivableCar {
  boost() {
    console.log('Boosting...');
  }
}

const sportsCar = new SportsCar();
sportsCar.drive();
sportsCar.boost();

class ToyCar extends Car {
  // Good! Not every car needs to be drivable!
}
```

Now we added a new `DrivableCar` class which is based on `Car`. And we can now choose whether we have other cars which are "just cars" (like `ToyCar`) or "drivable cars" (like `SportsCar`).

Now we follow the LSP.

So it's fair to say that the goal of the LSP is to force us to model our data correctly and think carefully about the entities we work with in our code.

The LSP is definitely an important and popular principle, but it doesn't have that big of an effect on our code if we look at it from a clean code perspective.

Yes, it could lead to smaller classes but it's really mostly focused on forcing us to model our data correctly.

---
### I: Interface Segregation Principle (ISP)

> Many client-specific interfaces are better than one general-purpose interface.

Reading sentences like the one above can always be confusing, but the ISP is actually also quite straightforward.

When working with classes in an OOP way, you frequently encounter "interfaces". Not all programming languages support interfaces, but many do.

Interfaces are basically contracts which force implementing classes to implement certain behaviors (methods and properties).

Here's an example of code that violates the ISP:

```typescript
interface Database {
  storeData(data: any);
  connect(uri: string);
}

class SQLDatabase implements Database {
  connect(uri: string) {
    // connecting...
  }

  storeData(data: any) {
    // Storing data...
  }
}

class InMemoryDatabase implements Database {
  connect(uri: string) {
    // Needs a connect method (because of interface)
    // but hasn't anything to connect to :(
  }

  storeData(data: any) {
    // Storing data...
  }
}
```

In this example, we have a `Database` interface and then we got a couple of database classes for different kinds of databases.

The problem with that code is, that the `Database` interface is too generic - "too general purpose".

It forces the `InMemoryDatabase` to add a `connect()` method, but this type of database has nothing to connect to — there's no separate database server or anything like that.

So it would be better to work with multiple, more focused interfaces instead of the general-purpose one. And that's exactly what the ISP says!

```typescript
interface Database {
  storeData(data: any);
}

interface RemoteDatabase {
  connect(uri: string);
}

class SQLDatabase implements Database, RemoteDatabase {
  connect(uri: string) {
    // connecting...
  }

  storeData(data: any) {
    // Storing data...
  }
}

class InMemoryDatabase implements Database {
  storeData(data: any) {
    // Storing data...
  }
}
```

Now, with this code, we've got two interfaces instead of one, and every class can pick the interfaces that make sense for it. So our `InMemoryDatabase` can just implement `Database`, which forces it to have a `storeData()` method — and ignore the `RemoteDatabase` interface, which would force it to add a `connect()` method.

The ISP is definitely an important and popular principle, but it doesn't have that big of an effect on our code if we look at it from a clean code perspective.

---

### D: Dependency Inversion Principle (DIP)

> Depend on abstractions, not on concrete implementations.

This last principle is a principle which you will often already follow, if you follow the other SOLID principles.

It basically is all about not being too specific in your code.

Sounds strange?

Here's an example of code which we could improve (this code assumes that we have database classes as shown for the ISP available):

```typescript
class App {
  private database: Database | RemoteDatabase;

  constructor(database: Database | RemoteDatabase) {
    if (database.connect) {
      database.connect('my-url');
    }
    this.database = database;
  }

  saveSettings() {
    this.database.storeData('Some data');
  }
}

const sqlDatabase = new SQLDatabase();
const app = new App(sqlDatabase);
```

In this example, we use the databases we created for the ISP. In the `constructor()` of our `App` class, we check whether we've received a `RemoteDatabase` (i.e. whether it has a `connect()` method), and if so, we connect to it.

The problem with this code is, that we're very specific in our `constructor()` method. The code we execute depends on which kind of database we're getting.

So in that example, we "depend on a concretion".

This isn't ideal,because it means we have to repeat this check everywhere we use a database, and we have to update our code whenever the database implementation changes.

Here's how we could rewrite the code to follow the DIP:

```javascript
class App {
  private database: Database;

  constructor(database: Database) {
    this.database = database;
  }

  saveSettings() {
    this.database.storeData('Some data');
  }
}

const sqlDatabase = new SQLDatabase();
sqlDatabase.connect('my-url');
const app = new App(sqlDatabase);
```

This example shows us why it's called the "Dependency Inversion Principle".

We ensure that we depend on an abstraction ("we just get some database") instead of a concretion ("we need to check whether we need to connect"). And we do that by inverting the dependency.

Instead of checking the database type inside `constructor()`, we simply require the caller to pass in a database that's already connected (if needed).

---

## Use Common Sense

There's one important thing I really want to point out: use your common sense.

It's easy to treat every tiny step as a single responsibility and single thing and if you do that, you end up with projects with 100s of classes that all contain only one method.

### This is NOT the goal and definitely NOT clean code!

Yes, classes should be small - just like methods and functions.

But "small" does not mean "almost empty".

It's okay to do work in classes and methods. And while methods probably shouldn't be dozens of lines long, the whole idea behind classes is to group related data and concepts together.

### Don't destroy that by breaking up everything!

Instead, keep the concepts and rules explained here in mind. Apply them as best you can, but don't start breaking everything apart because of them.

---

## Summary Checklist

- ✅ Focus on building "real objects" or data containers / structures
- ✅ Build small classes - focus on a single responsibility (which does not mean "single method")
- ✅ Build classes with high cohesion
- ✅ Follow the "Law of Demeter" for "real objects" (avoid `this.customer.lastPurchase.date`)
- ✅ Especially when doing OOP: Follow the SOLID principles
- ✅ Especially SRP and OCP will help a lot with writing clean code (= readable code)
