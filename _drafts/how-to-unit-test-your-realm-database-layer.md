---
layout: post
title:  'How to unit test your Realm database layer'
categories: ios testing
tags: ios realm unittest database
---

# Setup Realm for testing:

# 1. Testing models:

## 1.1. Test custom initializer:

What you have:

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

Steps to test:

{% highlight abc %}
1. Create a person using the custom initializer.
2. Expect that all properties of that person are correctly assigned.
{% endhighlight %}

Create the test:

{% highlight swift %}
describe("initialize with name and age") {
  it("initializes and assign properties correctly") {

  }
}
{% endhighlight %}

Fill in the test body:

{% highlight swift %}
describe("initialize with name and age") {
  it("initializes and assign properties correctly") {
    // 1. Create a person using the custom initializer
    let person = Person(name: "name A", age: 18)

    // 2. Expect that all properties of that person are correctly assigned.
    expect(person.name) == "name A"
    expect(person.age) == 18
  }
}
{% endhighlight %}

## 1.2. Test relationships:

### To-One Relationships:

What you have:

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

What you have:

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

What you have:

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

# 2. Testing Realm transactions:

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

### Sorting:

### Limiting results:

## 2.3. Update:

### Update properties:

### Update with primary key:

## 2.4. Delete:
