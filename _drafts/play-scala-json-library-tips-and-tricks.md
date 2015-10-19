---
layout: post
title:  Play Scala JSON Library Tips & Tricks
---
The following sections describe how to serialize / deserialize as JSON various object structure patterns by using the JSON library from Play 2:

Nested Objects
--------------

Object:

{% highlight scala %}
case class Name(first: String, last: String)
case class Person1(name: Name, age: Short)

val person1 = Person1(Name("Calin", "Burloiu"), 27)
{% endhighlight %}

Expected JSON:

```json
{
  "name" : {
    "first" : "Calin",
    "last" : "Burloiu"
  },
  "age" : 27
}
```

Solution:

```scala
implicit val nameFormat: Format[Name] = Json.format[Name]
val person1NestedFormat: Format[Person1] = (
  (__ \ "name").format[Name] and
  (__ \ "age").format[Short]
)(Person1, unlift(Person1.unapply))
```

Flattening a Nested Object
--------------------------

Object:

```scala
case class Address(country: String, city: String)
case class Person2(username: String, age: Short, address: Option[Address])

val person2 = Person2("calinburloiu", 27, Address("Romania", "Bucharest"))
```

Expected JSON:

```json
{
  "username" : "calinburloiu",
  "age" : 27,
  "country" : "Romania",
  "city" : "Bucharest"
}
```

Solution:

```scala
implicit val addressFormat: Format[Address] = Json.format[Address]

implicit val person2Format: Format[Person2] = (
  (__ \ "username").format[String] and
  (__ \ "age").format[Short] and
  __.format[Address]
)(Person2, unlift(Person2.unapply))
```

Flattening an Optional Nested Object
------------------------------------

Object:

```scala
case class Address(country: String, city: String)
case class Person3(username: String, age: Short, address: Option[Address])

val person3 = Person3("calinburloiu", 27, Some(Address("Romania", "Bucharest")))
```

Expected JSON:

```json
{
  "username" : "calinburloiu",
  "age" : 27,
  "country" : "Romania",
  "city" : "Bucharest"
}
```

Solution:

```scala
implicit val optionalAddressReads: Reads[Option[Address]] = (
  (__ \ "country").readNullable[String] and
  (__ \ "city").readNullable[String]
) { (countryOpt: Option[String], cityOpt: Option[String]) =>
  for {
    country <- countryOpt
    city <- cityOpt
  } yield Address(country, city)
}
implicit val addressWrites: Writes[Address] = Json.writes[Address]

implicit val person3Reads: Reads[Person3] = (
  (__ \ "username").read[String] and
  (__ \ "age").read[Short] and
  __.read[Option[Address]]
)(Person3)
implicit val person3Writes: Writes[Person3] = (
  (__ \ "username").write[String] and
  (__ \ "age").write[Short] and
  __.writeNullable[Address]
)(unlift(Person3.unapply))
```

Extra Fields in JSON
--------------------

Objects:

```scala
abstract class Person4(name: String, age: Short)
case class Man4(name: String, age: Short) extends Person4(name, age)
case class Woman4(salutation: String, name: String, age: Short)
    extends Person4(name, age)

val man4 = Man4("Bob", 25)
val woman4 = Woman4("Mrs.", "Alice", 36)
```

Expected JSON for `man4`:

```json
{
  "sex" : "male",
  "name" : "Bob",
  "age" : 25
}
```

Expected JSON for `woman4`:

```json
{
  "sex" : "female",
  "salutation" : "Mrs.",
  "name" : "Alice",
  "age" : 36
}
```

Solution A:

```scala
implicit val man4Reads: Reads[Man4] = (
  (__ \ "sex").read[String](Reads.verifying[String](_ == "male")) andKeep
  (__ \ "name").read[String] and 
  (__ \ "age").read[Short]
)(Man4)
implicit val man4Writes: Writes[Man4] = (
  (__ \ "sex").write[String] and
  (__ \ "name").write[String] and
  (__ \ "age").write[Short]
 ) { p: Man4 => ("male", p.name, p.age) }
implicit val woman4Reads: Reads[Woman4] = (
  (__ \ "sex").read[String](Reads.verifying[String](_ == "female")) andKeep
  (__ \ "salutation").read[String] and
  (__ \ "name").read[String] and 
  (__ \ "age").read[Short]
)(Woman4)
implicit val woman4Writes: Writes[Woman4] = (
  (__ \ "sex").write[String] and
  (__ \ "salutation").write[String] and
  (__ \ "name").write[String] and
  (__ \ "age").write[Short]
 ) { p: Woman4 => ("female", p.salutation, p.name, p.age) }
```

Solution B:
```scala
implicit val person4bReads: Reads[Person4] = (
  (__ \ "sex").read[String] and
  (__ \ "salutation").readNullable[String] and
  (__ \ "name").read[String] and 
  (__ \ "age").read[Short]
) { (t: String, salutation: Option[String], name: String, age: Short) => 
  (t, salutation) match {
    case ("male", None) | ("male", Some("Mr.")) => Man4(name, age)
    case ("female", Some(s)) => Woman4(s, name, age)
    case _ => throw new RuntimeException("invalid JSON")
  }
}
implicit val person4bWrites: Writes[Person4] = (
  (__ \ "sex").write[String] and
  (__ \ "salutation").writeNullable[String] and
  (__ \ "name").write[String] and
  (__ \ "age").write[Short]
) { p: Person4 => 
  p match {
    case Man4(name, age) => ("male", None, name, age)
    case Woman4(salutation, name, age) => ("female", Some(salutation), name, age)
  }
}
```

Recursive Objects
-----------------

Object:

```scala
case class Person5(name: String, age: Short, friends: Seq[Person5] = Seq())

val person5 = Person5(
  "John",
  25,
  Seq(
    Person5(
      "Bob",
      24,
      Seq(Person5("Alice", 20))
    ),
    Person5(
      "George",
      26
    )
  )
)
```

Expected JSON:

```json
{
  "name" : "John",
  "age" : 25,
  "friends" : [ {
    "name" : "Bob",
    "age" : 24,
    "friends" : [ {
      "name" : "Alice",
      "age" : 20,
      "friends" : [ ]
    } ]
  }, {
    "name" : "George",
    "age" : 26,
    "friends" : [ ]
  } ]
}
```

Solution:

```scala
implicit val person5Format: Format[Person5] = (
  (__ \ "name").format[String] and
  (__ \ "age").format[Short] and
  (__ \ "friends").lazyFormat(
    Reads.seq[Person5],
    Writes.seq[Person5]
  )
)(Person5, unlift(Person5.unapply))
```

Object with Generic Fields
--------------------------

Objects:

```scala
case class Car(brand: String, horsePower: Float) extends Preference
case class Computer(brand: String, cpuFreq: Float, memory: Short) extends Preference
case class Person6[+P <: Preference](name: String, age: Short, preference: P)

val person6a = Person6("John", 25, Car("BMW", 300.0f))
val person6b = Person6("George", 22, Computer("Toshiba", 2.8f, 4096))
```

Expected JSON for `person6a`:

```json
{
  "name" : "John",
  "age" : 25,
  "preference" : {
    "brand" : "BMW",
    "horsePower" : 300.0
  }
}
```

Expected JSON for `person6b`:

```json
{
  "name" : "George",
  "age" : 22,
  "preference" : {
    "brand" : "Toshiba",
    "cpuFreq" : 2.8,
    "memory" : 4096
  }
}
```

Solution:

```scala
def person6Reads[P <: Preference](reads: Reads[P]): Reads[Person6[P]] = (
  (__ \ "name").read[String] and
  (__ \ "age").read[Short] and
  (__ \ "preference").read[P](reads)
) { (name: String, age: Short, preference: P) =>
  Person6(name, age, preference)
}
implicit val carLoverReads: Reads[Person6[Car]] = person6Reads(carFormat)
implicit val computerLoverReads: Reads[Person6[Computer]] =
    person6Reads(computerFormat)

def preferenceWrites = new Writes[Preference] {
  def writes(preference: Preference): JsValue = {
    preference match {
      case car: Car => Json.toJson(car)
      case computer: Computer => Json.toJson(computer)
    }
  }
}
implicit val person6Writes: Writes[Person6[Preference]] = (
  (__ \ "name").write[String] and
  (__ \ "age").write[Short] and
  (__ \ "preference").write[Preference](preferenceWrites)
) { p: Person6[Preference] =>
  (p.name, p.age, p.preference)
}
```

