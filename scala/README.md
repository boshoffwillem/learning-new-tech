# Scala Cheat Sheet

## 1. Building Scala

There are a couple of different ways to build Scala files/projects.

### 1.1 With scalac

 Compile individual files using ```scalac```, for example,

 ```bash
 scalac Main.scala
 ```

 This produces two files:

- Main$.class
- Main.class

To run the compiled files

```bash
scala Main
```

### 1.2 With sbt

This is for running Scala projects that contains a ```build.sbt``` file.
Run ```sbt``` in the directory that contains the ```build.sbt``` file.
This will provide a prompt from which you can do things with the project.
Type ```run``` to compile and run the project.
Type ```~run``` to compile and run the project and cause sbt to re-run your
project on every file save.

## 2. Language documentation

### 2.1 Variables

```scala
val x = 1 // immutable
var y = 0 // mutable
```

Scala has **type inference** which means you can either explicitly specify a
variable's type of leave it and then Scala will infer it.
The following are all valid Scala variable declarations

```scala
val x1 = 1
val x2: Int = 1

val s1 = "string"
val s2: String = "string"

val o1 = new SomeObject()
val o2: SomeObject = new SomeObject()
```

### 2.2 Control structures

#### 2.2.1 if/else

Similar to most languages

```scala
if (test1) {
    doA()
} else if (test2) {
    doB()
} else if (test3) {
    doC()
} else {
    doD()
}
```

**However**, in Scala the if/else returns a value.

```scala
val x = if (a < b) a else b
```

#### 2.2.2 Match expressions

Match expressions are like ```switch``` statements, but a lot more powerful.
It can be used on any data type.

```scala
val result = i match {
    case 1 => "one"
    case 2 => "two"
    case _ => "not 1 or 2"
}

val booleanAsString = bool match {
    case true => "true"
    case false => "false"
}

def getClassAsString(x: Any):String = x match {
    case s: String => s + " is a String"
    case i: Int => "Int"
    case f: Float => "Float"
    case l: List[_] => "List"
    case p: Person => "Person"
    case _ => "Unknown"
}
```

#### 2.2.3 try/catch

Scala's try/catch control structure lets you catch exceptions.
It's similar to Java, but its syntax is consistent with match expressions:

```scala
try {
    writeToFile(text)
} catch {
    case fnfe: FileNotFoundException => println(fnfe)
    case ioe: IOException => println(ioe)
}
```

#### 2.2.4 for loops and experssions

```scala
for (arg <- args) println(arg)

// "x to y" syntax
for (i <- 0 to 5) println(i)

// "x to y by" syntax
for (i <- 0 to 10 by 2) println(i)

val x = for (i <- 1 to 5) yield i * 2
```

```scala
val fruits = List("apple", "banana", "lime", "orange")

val fruitLengths = for {
    f <- fruits
    if f.length > 4
} yield f.length
```

#### 2.2.5 while and do/while

```scala
// while loop
while(condition) {
    statement(a)
    statement(b)
}

// do-while
do {
   statement(a)
   statement(b)
}
while(condition)
```

### 2.3 Classes

Example of a Scala class

```scala
class Person(var firstName: String, var lastName: String) {
    def printFullName() = println(s"$firstName $lastName")
}
```

How you would use this class

```scala
val p = new Person("Julia", "Kern")
println(p.firstName)
p.lastName = "Manes"
p.printFullName()
```

Notice that there's no need to create "get" and "set" methods
to access the fields in the class.

### 2.4 Methods

Just like other OOP languages, Scala classes have methods.

```scala
def sum(a: Int, b: Int): Int = a + b
```

You don't have to declare a method's return type.

```scala
def sum(a: Int, b: Int) = a + b
```

### 2.5 Traits

Traits in Scala are a lot of fun, and they also let you break
your code down into small, modular units. To demonstrate traits,
here's an example from later in the book. Given these three traits:

```scala
trait Speaker {
    def speak(): String  // has no body, so it's abstract
}

trait TailWagger {
    def startTail(): Unit = println("tail is wagging")
    def stopTail(): Unit = println("tail is stopped")
}

trait Runner {
    def startRunning(): Unit = println("I'm running")
    def stopRunning(): Unit = println("Stopped running")
}
```

You can create a ```Dog``` class that extends all of those traits
while providing behavior for the ```speak``` method:

```scala
class Dog(name: String) extends Speaker with TailWagger with Runner {
    def speak(): String = "Woof!"
}
```

Similarly, here's a ```Cat``` class that shows how to override multiple trait methods:

```scala
class Cat extends Speaker with TailWagger with Runner {
    def speak(): String = "Meow"
    override def startRunning(): Unit = println("Yeah ... I don't run")
    override def stopRunning(): Unit = println("No need to stop")
}
```

### 2.6 Collection classes

Scala has the following basic collections: ```List```, ```ListBuffer```,
```Vector```, ```ArrayBuffer```, ```Map``` and ```Set```.

#### 2.6.1 Populating lists

There are times when it's helpful to create sample lists that are populated
with data, and Scala offers many ways to populate lists.
Here are just a few:

```scala
val nums = List.range(0, 10)
val nums = (1 to 10 by 2).toList
val letters = ('a' to 'f').toList
val letters = ('a' to 'f' by 2).toList
```

#### 2.6.2 Sequence methods

oe
