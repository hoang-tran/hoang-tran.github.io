---
layout: post
title:  'How to unit test your Realm database layer'
categories: ios testing
meta_description: Learn how to write unit tests to cover most of Realm features and make sure that your database layer works flawlessly.
description: Learn how to write unit tests to cover most of Realm features and make sure that your database layer works flawlessly.
summary: Learn how to write unit tests to cover most of Realm features and make sure that your database layer works flawlessly.
tags: ios realm unittest database swift
image: how-to-unit-test-your-realm-database-layer/realm-logo.jpg
related: [
  'How to unit test your Network Layer in iOS',
  'How to write unit tests in iOS Part 1: XCTestCase',
  'How to write unit tests in iOS Part 2: Behavior-driven Development (BDD)',
  'Write better unit test assertions with Nimble'
]
---

Database is a crucial part of any softwares nowadays, especially mobile applications. We use database to store various things, such as:

* app settings: to make it consistent between app launches.
* app context: to perserve the last screen user visited. (in case the app was killed, either intentionally by the user or automatically by the system to reclaim memory)
* data from a network reponse: to better support offline mode.
* anything you want to save for later submission when network is available.
* etc and etc...

Those are the things that make an app fast and responsive and eventually useful.

Therefore, we wanna make sure that our database works correctly. No mistakes or errors is tolerant here.

But **how do we achieve that?**

Well, you know, we write unit tests for it.

The database techonology I'm gonna use today is **Realm**. Although there are many other good alternatives out there that can do the job well, they're just not as great as Realm. Here's a couple of reasons why Realm is my database of choice:

* It's fast, like... extremely fast. Look at this chart: (Source: [qiita](http://qiita.com/moriyaman/items/1a2916f4c2b79e934370))

![Realm speed test](/images/how-to-unit-test-your-realm-database-layer/realm-chart.png)

* It's easy to use. The interface is very simple and swift-friendly. You can do all kinds of database operations with minimal efforts.

{% highlight swift %}
// Define a Dog model
class Dog: Object {
  dynamic var name = ""
  dynamic var age = 0
}

// Create a Dog instance
let myDog = Dog()
myDog.name = "Rex"
myDog.age = 1

// Add the dog the database
let realm = try! Realm()
try! realm.write {
  realm.add(myDog)
}

// Retrieve dogs from database
let puppies = realm.objects(Dog.self).filter("age < 2")
puppies.count // => 1
{% endhighlight %}

* It's open source, mostly.
  * If you encounter a bug, you can hop in and take a look at the underlying implementation. That would help you get an understanding of how things work and hopefully fix your bug.
  * If you find something that can be improved, you can work on it and open a Pull-Request to contribute back to Realm. That's powerful.

Realm is late to the game but proves to be one of the best mobile databases in the market, if not the best, apparently.

However, we're **not** gonna learn how to use Realm today. Instead, we will try to write unit tests for it.

We're gonna write a lot and cover most of Realm features. It may take time so make sure you're passionate about it.

Alright, if that gets you excited, let's jump in.

# Prerequisites:

Before we start, make sure you know:

* [how to use Realm Swift](https://realm.io/docs/swift/latest/).
* how to write basic unit tests:
  * [How to write unit tests in iOS Part 1: XCTestCase](/ios/testing/2016/07/31/how-to-write-unit-tests-in-ios-p1-xctestcase).
  * [How to write unit tests in iOS Part 2: Behavior-driven Development (BDD)](/ios/testing/2016/08/07/how-to-write-unit-tests-in-ios-p2-behavior-driven-development-bdd).
  * [Write better unit test assertions with Nimble](/ios/testing/2016/08/09/write-better-unit-test-assertion-with-nimble).

# Setup Realm for testing:

When we're testing, we may create/update or delete some records from the Realm database. We don't want those changes to affect the production database. If we let that happens, it would become a disaster.

What we're gonna do is we direct all Realm operations into a test database instead of the real one. Realm provides a very convenient way to do exactly just that.

{% highlight swift %}
Realm.Configuration.defaultConfiguration.inMemoryIdentifier = "database A"
{% endhighlight %}

This will point Realm to an in-memory database (namely "database A") where we can freely experiment and do whatever we want. Since it's in memory, it won't affect the production database at all.

Now we need to call this Realm configuration line before any tests get executed. The ideal place is in the **beforeSuite** method:

{% highlight swift %}
class PersonSpec: QuickSpec {
  override func spec() {
    super.spec()

    beforeSuite {
      Realm.Configuration.defaultConfiguration.inMemoryIdentifier = self.name
    }
  }
}
{% endhighlight %}

Note: you can set the `inMemoryIdentifier` to anything as long as it's not nil. In this case, I'm setting it to the Spec's name so every Spec will use different in-memory database.

Additionally, we wanna clear out all records so that each test would start with a fresh empty database. Override the **beforeEach** and put in the code to delete everything within Realm:

{% highlight swift %}
class PersonSpec: QuickSpec {
  override func spec() {
    super.spec()

    beforeSuite {
      Realm.Configuration.defaultConfiguration.inMemoryIdentifier = self.name
    }

    beforeEach {
      let realm = try! Realm()
      try! realm.write {
        realm.deleteAll()
      }
    }
  }
}
{% endhighlight %}

We now have a clean and isolated testing environment. Let's move on and write some real tests.

* [1. Testing models](#testing-models)
  * [1.1. Test custom initializer](#test-custom-initializer)
  * [1.2. Test relationships](#test-relationships)
    * [To-One Relationships](#to-one-relationships)
    * [To-Many Relationships](#to-many-relationships)
    * [Inverse Relationships](#inverse-relationships)
  * [1.3. Test ignored properties](#test-ignored-properties)
* [2. Testing **CRUD** operations](#testing-crud-operations)
  * [2.1. **C**reate](#create)
  * [2.2. **R**ead](#read)
    * [Retrieving all objects](#retrieving-all-objects)
    * [Filtering](#filtering)
    * [Sorting](#sorting)
  * [2.3. **U**pdate](#update)
    * [Update properties](#update-properties)
    * [Update with primary key](#update-with-primary-key)
  * [2.4. **D**elete](#delete)

# 1. Testing models:

## 1.1. Test custom initializer:

Let's say we have a **Person** model with 2 properties: name and age.

{% highlight swift %}
class Person: Object {
  dynamic var name = ""
  dynamic var age = 0
}
{% endhighlight %}

We also have a custom initializer to quickly set those properites:

{% highlight swift %}
class Person: Object {
  dynamic var name = ""
  dynamic var age = 0

  convenience init(name: String, age: Int) {
    self.init()
    self.name = name
    self.age = age
  }
}
{% endhighlight %}

Now how do we test this intializer method?

Well, we do it in 2 steps:

{% highlight abc %}
1. Create a person using the custom initializer.
2. Expect all properties of that person are correctly assigned.
{% endhighlight %}

Let's go ahead and add the test to your Spec file: (`PersonSpec.swift` in this case)

{% highlight swift %}
describe("initialize with name and age") {
  it("initializes and assign properties correctly") {

  }
}
{% endhighlight %}

Fill in the test body with 2 steps we mentioned above:

{% highlight swift %}
describe("initialize with name and age") {
  it("initializes and assign properties correctly") {
    // 1. Create a person using the custom initializer
    let person = Person(name: "name A", age: 18)

    // 2. Expect all properties of that person are correctly assigned.
    expect(person.name) == "name A"
    expect(person.age) == 18
  }
}

{% endhighlight %}

Run the test (Cmd + U) and make sure that it passes.

## 1.2. Test relationships:

### To-One Relationships:

What we have:

{% highlight swift %}
class Dog: Object {
  dynamic var name = ""
  dynamic var owner: Person?
}
{% endhighlight %}

Steps to test:

{% highlight abc %}
1. Create a person.
2. Create a dog.
3. Set the dog's owner to the person.
4. Save the dog to database.
5. Expect that the person is also saved to database as the dog's owner.
{% endhighlight %}

Create the test:

{% highlight swift %}
describe("to-one relationship") {
  it("saves the object and its relationship to database correctly") {

  }
}
{% endhighlight %}

Fill in the test body:

{% highlight swift %}
describe("to-one relationship") {
  it("saves the object and its relationship to database correctly") {
    // 1. Create a person
    let person = Person(name: personName, age: personAge)

    // 2. Create a dog
    let dog = Dog(value: ["name": dogName])

    // 3. Set the dog's owner to the person
    dog.owner = person

    // 4. Save the dog to database
    let realm = try! Realm()
    try! realm.write {
      realm.add(dog)
    }

    // 5. Expect that the person is also saved to database as the dog's owner
    let dogFromDatabase = realm.objects(Dog.self).last
    expect(dogFromDatabase?.name) == dogName
    expect(dogFromDatabase?.owner?.name) == personName
    expect(dogFromDatabase?.owner?.age) == personAge
  }
}
{% endhighlight %}

Note that `personName`, `personAge` and `dogName` are just some constants defined elsewhere to make the code less duplicated.

### To-Many Relationships:

What we have:

{% highlight swift %}
class Person: Object {
  ...

  let dogs = List<Dog>()

  ...
}
{% endhighlight %}

Steps to test:

{% highlight abc %}
1. Create a person.
2. Create 2 dogs.
3. Add the 2 dogs as the person's dogs.
4. Save the person to database.
5. Expect that the dogs are also saved to database as the person's dogs.
{% endhighlight %}

Create the test:

{% highlight swift %}
describe("to-many relationship") {
  it("saves the object and its relationship to database correctly") {

  }
}
{% endhighlight %}

Fill in the test body:

{% highlight swift %}
describe("to-many relationship") {
  it("saves the object and its relationship to database correctly") {
    // 1. Create a person
    let person = Person(name: personName, age: personAge)

    // 2. Create 2 dogs
    let dog0 = Dog(value: ["name": "dog 0"])
    let dog1 = Dog(value: ["name": "dog 1"])

    // 3. Add the 2 dogs as the person's dogs
    person.dogs.append(dog0)
    person.dogs.append(dog1)

    // 4. Save the person to database
    let realm = try! Realm()
    try! realm.write {
      realm.add(person)
    }

    // 5. Expect that the dogs are also saved to database as the person's dogs
    let personFromDatabase = realm.objects(Person.self).last
    expect(personFromDatabase?.dogs.count) == 2
    expect(personFromDatabase?.dogs[0].name) == "dog 0"
    expect(personFromDatabase?.dogs[1].name) == "dog 1"
  }
}
{% endhighlight %}

### Inverse Relationships:

What we have:

{% highlight swift %}
class Dog: Object {
  ...

  let owners = LinkingObjects(fromType: Person.self, property: "dogs")

  ...
}
{% endhighlight %}

Steps to test:

{% highlight abc %}
1. Create a dog.
2. Create 2 persons.
3. Add the dog to each person's dogs.
4. Save the 2 persons to database.
5. Expect that the dog is also saved and its owners are correctly assigned.
{% endhighlight %}

Create the test:

{% highlight swift %}
describe("inverse relationship") {
  it("saves the object and its relationship to database correctly") {

  }
}
{% endhighlight %}

Fill in the test body:

{% highlight swift %}
describe("inverse relationship") {
  it("saves the object and its relationship to database correctly") {
    // 1. Create a dog
    let dog = Dog(value: ["name": dogName])

    // 2. Create 2 persons
    let person0 = Person(name: "person 0", age: personAge)
    let person1 = Person(name: "person 1", age: personAge)

    // 3. Add the dog to each person's dogs
    person0.dogs.append(dog)
    person1.dogs.append(dog)

    // 4. Save the 2 persons to database
    let realm = try! Realm()
    try! realm.write {
      realm.add(person0)
      realm.add(person1)
    }

    // 5. Expect that the dog is also saved and its owners are correctly assigned
    let dogFromDatabase = realm.objects(Dog.self).last
    expect(dogFromDatabase?.owners.count) == 2
    expect(dogFromDatabase?.owners[0].name) == person0.name
    expect(dogFromDatabase?.owners[1].name) == person1.name
  }
}
{% endhighlight %}

## 1.3. Test ignored properties:

What we have:

{% highlight swift %}
class Person: Object {
  dynamic var name = ""
  dynamic var age = 0
  dynamic var address = ""
  dynamic var height = 0.0

  override static func ignoredProperties() -> [String] {
    return ["address", "height"]
  }
}
{% endhighlight %}

Steps to test:

{% highlight abc %}
1. Create a person.
2. Set the person's addess and height to something else.
3. Save the person to database.
4. Expect that the address and height properties are not saved to database.
{% endhighlight %}

Create the test:

{% highlight swift %}
describe("ignored properties") {
  it("doesn't save those properties to database") {

  }
}
{% endhighlight %}

Fill in the test body:

{% highlight swift %}
describe("ignored properties") {
  it("doesn't save those properties to database") {
    // 1. Create a person
    let person = Person(name: personName, age: personAge)

    // 2. Set the person's address and height to something else
    person.address = personAddress
    person.height = personHeight

    // 3. Save the person to database
    let realm = try! Realm()
    try! realm.write {
      realm.add(person)
    }

    // 4. Expect that the address and height properties are not saved to database
    let personFromDatabase = realm.objects(Person.self).last
    expect(personFromDatabase?.address) != personAddress
    expect(personFromDatabase?.height) != personHeight
  }
}
{% endhighlight %}

Again, the `personAddress` and `personHeight` are just some constants defined elsewhere.

# 2. Testing CRUD operations:

## 2.1. Create:

What we have:

{% highlight swift %}
class Person: Object {
  ...

  func save() {
    let realm = try! Realm()
    try! realm.write {
      realm.add(self)
    }
  }

  ...
}
{% endhighlight %}

Steps to test:

{% highlight abc %}
1. Create a person.
2. Call the save method.
3. Expect the person to be saved to database.
{% endhighlight %}

Create the test:

{% highlight swift %}
describe("save") {
  it("saves object to database correctly") {

  }
}
{% endhighlight %}

Fill in the test body:

{% highlight swift %}
describe("save") {
  it("saves object to database correctly") {
    // 1. Create a person
    let person = Person(name: personName, age: personAge)

    // 2. Call the save method
    person.save()

    // 3. Expect the person to be saved to database
    let realm = try! Realm()
    let personFromDatabase = realm.objects(Person.self).last
    expect(personFromDatabase?.name) == person.name
    expect(personFromDatabase?.age) == person.age
  }
}
{% endhighlight %}

## 2.2. Read:

To read from database, we must have some data in it first. Let's create a helper method to provide dummy data:

{% highlight swift %}
class PersonSpec: BaseSpec {
  override func spec() {
    ...
  }

  func createPersons(number: Int) {
    let realm = try! Realm()
    try! realm.write {
      for i in 0..<number {
        let person = Person(name: "person \(i)", age: 17 + i)
        realm.add(person)
      }
    }
  }

  ...
}
{% endhighlight %}

### Retrieving all objects:

What we have:

{% highlight swift %}
class Person: Object {
  ...

  class func all() -> Results<Person> {
    let realm = try! Realm()
    return realm.objects(Person.self)
  }

  ...
}
{% endhighlight %}

Steps to test:

{% highlight abc %}
1. Create 3 dummy persons in database.
2. Call the "all" method.
3. Expect the returned results to exactly match the 3 dummy persons.
{% endhighlight %}

Create the test:

{% highlight swift %}
describe("all") {
  it("returns all persons") {

  }
}
{% endhighlight %}

Fill in the test body:

{% highlight swift %}
describe("all") {
  it("returns all persons") {
    // 1. Create 3 dummy persons in database
    self.createPersons(3)

    // 2. Call the "all" method
    let persons = Person.all()

    // 3. Expect the returned results to exactly match the 3 dummy persons
    expect(persons.count) == 3
    expect(persons[0].name) == "person 0"
    expect(persons[1].name) == "person 1"
    expect(persons[2].name) == "person 2"
  }
}
{% endhighlight %}

### Filtering:

What we have:

{% highlight swift %}
class Person: Object {
  ...

  class func adults() -> Results<Person> {
    let realm = try! Realm()
    return realm.objects(Person.self).filter("age >= 18")
  }

  ...
}
{% endhighlight %}

Steps to test:

{% highlight abc %}
1. Create 3 dummy persons in database with age 17, 18 and 19 respectively.
2. Call the "adults" method.
3. Expect the returned results to only contain 2 persons with age 18 and 19.
{% endhighlight %}

Create the test:

{% highlight swift %}
describe("adults") {
  it("returns persons who are 18+ only") {

  }
}
{% endhighlight %}

Fill in the test body:

{% highlight swift %}
describe("adults") {
  it("returns persons who are 18+ only") {
    // 1. Create 3 dummy persons in database with age 17, 18 and 19 respectively
    self.createPersons(3)

    // 2. Call the "adults" method
    let adults = Person.adults()

    // 3. Expect the returned results to only contain 2 persons with age 18 and 19
    expect(adults.count) == 2
    expect(adults[0].age) == 18
    expect(adults[1].age) == 19
  }
}
{% endhighlight %}

### Sorting:

What we have:

{% highlight swift %}
class Person: Object {
  ...

  class func oldestFirst() -> Results<Person> {
    let realm = try! Realm()
    return realm.objects(Person.self).sorted("age", ascending: false)
  }

  ...
}
{% endhighlight %}

Steps to test:

{% highlight abc %}
1. Create 3 dummy persons in database with age 17, 18 and 19 respectively.
2. Call the "oldestFirst" method.
3. Expect the returned results to be in sorted order. (old -> young)
{% endhighlight %}

Create the test:

{% highlight swift %}
describe("oldestFirst") {
  it("returns persons in sorted order, old to young") {

  }
}
{% endhighlight %}

Fill in the test body:

{% highlight swift %}
describe("oldestFirst") {
  it("returns persons in sorted order, old to young") {
    // 1. Create 3 dummy persons in database with age 17, 18 and 19 respectively
    self.createPersons(3)

    // 2. Call the "oldestFirst" method
    let results = Person.oldestFirst()

    // 3. Expect the returned results to be in sorted order (old -> young)
    expect(results.count) == 3
    expect(results[0].age) == 19
    expect(results[1].age) == 18
    expect(results[2].age) == 17
  }
}
{% endhighlight %}

## 2.3. Update:

### Update properties:

What we have:

{% highlight swift %}
class Person: Object {
  dynamic var name = ""
  dynamic var age = 0
  ...

  func updateName(name: String, age: Int) {
    let realm = try! Realm()
    try! realm.write {
      self.name = name
      self.age = age
    }
  }

  ...
}
{% endhighlight %}

Steps to test:

{% highlight abc %}
1. Create a person.
2. Save the person to database.
3. Call the "updateName:age" method
4. Expect the updated properties are saved to database.
{% endhighlight %}

Create the test:

{% highlight swift %}
describe("updateName:age") {
  it("updates properties to database correctly") {

  }
}
{% endhighlight %}

Fill in the test body:

{% highlight swift %}
describe("updateName:age") {
  it("updates properties to database correctly") {
    // 1. Create a person
    let person = Person(name: personName, age: personAge)

    // 2. Save the person to database
    person.save()

    // 3. Call the "updateName:age" method
    person.updateName(newName, age: newAge)

    // 4. Expect the updated properties are saved to database
    let realm = try! Realm()
    let personFromDatabase = realm.objects(Person.self).last
    expect(personFromDatabase?.name) == newName
    expect(personFromDatabase?.age) == newAge
  }
}
{% endhighlight %}

### Update with primary key:

What we have:

{% highlight swift %}
class Person: Object {
  ...
  dynamic var id = 0

  override static func primaryKey() -> String? {
    return "id"
  }

  func updateFrom(person: Person) {
    guard self.id == person.id else { return }

    let realm = try! Realm()
    try! realm.write {
      realm.add(person, update: true)
    }
  }

  ...
}
{% endhighlight %}

Steps to test:

Context 1: different primary key (id)

{% highlight abc %}
1. Create person A.
2. Save person A to database.
3. Create person B with different id than person A.
4. Call "updateFrom" on person A, pass person B in as argument.
5. Expect person A not to update anything.
{% endhighlight %}

Context 2: same primary key (id)

{% highlight abc %}
1. Create person A.
2. Save person A to database.
3. Create person B with the same id as person A.
4. Call "updateFrom" on person A, pass person B in as argument.
5. Expect person A to have its properties updated from person B.
{% endhighlight %}

Create the test:

{% highlight swift %}
describe("update with id") {
  context("different id") {
    it("does not update anything") {

    }
  }

  context("same id") {
    it("updates properties to database correctly") {

    }
  }
}
{% endhighlight %}

Fill in the test body:

{% highlight swift %}
describe("update with id") {
  context("different id") {
    it("does not update anything") {
      // 1. Create a person A
      let personA = Person(id: 1, name: personName, age: personAge)

      // 2. Save person A to database
      personA.save()

      // 3. Create person B with different id than person A
      let personB = Person(id: 2, name:newName, age: newAge)

      // 4. Call "updateFrom" on person A, pass person B in as argument
      personA.updateFrom(personB)

      // 5. Expect person A not to update anything
      let realm = try! Realm()
      let personFromDatabase = realm.objects(Person.self).last
      expect(personFromDatabase?.name) == personName
      expect(personFromDatabase?.age) == personAge
    }
  }

  context("same id") {
    it("updates properties to database correctly") {
      // 1. Create a person A
      let personA = Person(id: 1, name: personName, age: personAge)

      // 2. Save person A to database
      personA.save()

      // 3. Create person B with the same id as person A
      let personB = Person(id: 1, name:newName, age: newAge)

      // 4. Call "updateFrom" on person A, pass person B in as argument
      personA.updateFrom(personB)

      // 5. Expect person A to have its properties updated from person B
      let realm = try! Realm()
      let personFromDatabase = realm.objects(Person.self).last
      expect(personFromDatabase?.name) == newName
      expect(personFromDatabase?.age) == newAge
    }
  }
}
{% endhighlight %}

## 2.4. Delete:

What we have:

{% highlight swift %}
class Person: Object {
  ...

  func delete() {
    let realm = try! Realm()
    try! realm.write {
      realm.delete(self)
    }
  }

  ...
}
{% endhighlight %}

Steps to test:

{% highlight abc %}
1. Create 3 dummy persons.
2. Find the second person in database.
3. Call "delete" method on that person.
4. Expect 2 persons remain in the database: the first and third one.
{% endhighlight %}

Create the test:

{% highlight swift %}
describe("delete") {
  it("deletes the object from database") {
    // 1. Create 3 dummy persons
    self.createPersons(3)

    // 2. Find the second person in database
    let realm = try! Realm()
    let secondPerson = realm.objectForPrimaryKey(Person.self, key: 1)

    // 3. Call "delete" method on that person
    secondPerson?.delete()

    // 4. Expect 2 persons remain in the database: the first and third one
    let persons = Person.all()
    expect(persons.count) == 2
    expect(persons[0].id) == 0
    expect(persons[1].id) == 2
  }
}
{% endhighlight %}

# Wrap up

These are just some simple use cases of Realm. They serve as the building blocks for writing more complex ones.

In a real-world scenario, it might require a combination of multiple Realm operations at a time. However, the testing approach is basically the same. You can extend it very easily.

Alright, that's it for today. The sample project can be found at: <https://github.com/hoang-tran/UnitTestRealm>

I wish to hear more from you guys too. Please drop me a comment down below.

**Have you ever used Realm? Do you write unit tests for it? Do you have any tips you wanna share with everyone?**
