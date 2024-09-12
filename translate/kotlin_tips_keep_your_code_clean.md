# [译] - Kotlin Tips : Keep Your Code Clean - Kotlin技巧：保证你的代码整洁。

As a developer, clean and maintainable code is key. In this post, we’ll explore valuable tips to help you achieve a cleaner codebase.  
作为开发人员，干净且可维护的代码是关键。在这篇文章中，我们将探讨一些有价值的技巧来帮助您实现更清晰的代码库。

## 1\. Use Extension Functions - 使用扩展函数

Extension functions allows you to add functionality to existing classes without modifying their source code.  
扩展函数允许你向现有类添加功能，而无需修改其源代码。

-   Reduce boilerplate code 减少样板代码
-   Improve code readability 提高代码可读性

```kotlin
// Function to split the string into individual words
fun String.splitToWords(): List<String> {
    return this.split("\\s+".toRegex())
}

fun main() {
    val sentence = "Hello World, this is an example sentence"
    val words = sentence.splitToWords()
    println(words) // [Hello, World,, this, is, an, example, sentence]
}
```

## 2\. Use Data Classes - 使用数据类

-   Concise way to create data-holding classes  
    创建数据保存类的简洁方法
-   Automatically generates: equals(), hashCode(), toString()  
    自动生成：equals()、hashCode()、toString()
-   Makes your code more efficient  
    让你的代码更加高效

```kotlin
// Instead of writing a class with getters and setters
class User(val name: String, val age: Int) {
    override fun equals(other: Any?): Boolean {
        // implementation
    }

    override fun hashCode(): Int {
        // implementation
    }

    override fun toString(): String {
        // implementation
    }
}

// Use a data class
data class User(val name: String, val age: Int)
```

## 3\. Use Coroutines - 使用协程

-   Easier to read and maintain asynchronous code  
    更容易阅读和维护异步代码
-   Avoids callback hell 避免回调地狱
-   Makes code more efficient  
    让代码更高效

```kotlin
suspend fun performRequest(): String {
    // perform request
    return "result"
}
```

## 4\. Use Null Safety - 使用空安全

-   Helps to avoid null pointer exceptions  
    有助于避免空指针异常
-   Ensures code is safe and reliable  
    确保代码安全可靠
-   Prevents runtime crashes and errors due to null references  
    防止由于空引用而导致运行时崩溃和错误

```kotlin
// Instead of using the !! operator
val name: String? = "John"
println(name!!) // throws NPE if name is null


// Use the safe call operator ?.
val name: String? = "John"
println(name?.toUpperCase()) // prints "JOHN" or null
```

## 5\. Use Smart Casts - 使用智能转换

-   Reduces boilerplate code 减少样板代码
-   Makes code more readable 使代码更具可读性
-   Automatically casts an object to a specific type when its type is checked beforehand  
    当事先检查对象的类型时，自动将对象转换为特定类型

```kotlin
// Instead of using the as keyword to cast an object
val obj: Any = "John"
val name = obj as String


// Use a smart cast
val obj: Any = "John"
if (obj is String) {
    val name = obj // name is automatically cast to String
}
```

## 6\. Use when Expression - 使用when表达式

-   Reduces boilerplate code 减少样板代码
-   Makes code more readable 使代码更具可读性
-   Allows for concise and expressive handling of different scenarios based on the value of an object  
    允许根据对象的值简洁而富有表现力地处理不同的场景
-   Can replace lengthy if-else chains with a more concise and elegant syntax  
    可以用更简洁和优雅的语法替换冗长的 if-else 链

```kotlin
// Instead of using series of if-else statements
val color = "red"
if (color == "red") {
    println("The color is red")
} else if (color == "green") {
    println("The color is green")
} else {
    println("The color is unknown")
}


// Use a when expression
val color = "red"
when (color) {
    "red" -> println("The color is red")
    "green" -> println("The color is green")
    else -> println("The color is unknown")
}
```

## 7\. Use String Templates - 使用字符串模板

-   Reduces boilerplate code 减少样板代码
-   Makes code more readable 使代码更具可读性
-   Allows for concise and expressive string manipulation  
    允许简洁且富有表现力的字符串操作
-   Enables easy insertion of variables and expressions into strings  
    可以轻松地将变量和表达式插入字符串中
-   Eliminates the need for concatenation or StringBuilder usage  
    消除了连接或 StringBuilder 使用的需要
-   Improves code clarity and maintainability  
    提高代码清晰度和可维护性

```kotlin
// Instead of using the + operator to concatenate strings
val name = "John"
val age = 30
val message = "My name is " + name + " and I am " + age + " years old."

// Use a string template
val name = "John"
val age = 30
val message = "My name is $name and I am $age years old."
```

## 8\. Use apply Function - 使用apply函数

-   Allows for concise and expressive object configuration  
    允许简洁且富有表现力的对象配置
-   Enables chaining of multiple operations on an object in a fluent manner  
    允许以流畅的方式链接一个对象上的多个操作
-   Returns the object itself, making it suitable for method chaining and builder patterns  
    返回对象本身，使其适合方法链和构建器模式
-   Improves code clarity and maintainability by reducing the need for temporary variables and repetitive code  
    通过减少对临时变量和重复代码的需求，提高代码清晰度和可维护性

```kotlin
// Instead of creating an object and then setting its properties
val person = Person()
person.name = "John"
person.age = 30


// Use the apply function
val person = Person().apply {
    name = "John"
    age = 30
}
```

## 9\. Use sealed Classes - 使用密封类

-   Enables exhaustive when expressions  
    启用详尽、穷举的when表达式
-   Defines a fixed set of subclasses  
    定义一组固定的子类
-   Improves code clarity and maintainability  
    提高代码清晰度和可维护性
-   Enables compiler checks 启用编译器检查

```kotlin
sealed class Color {
    object Red : Color() {
        val hexCode = "#FF0000"
    }
    object Green : Color() {
        val hexCode = "#00FF00"
    }
    object Blue : Color() {
        val hexCode = "#0000FF"
    }
}

fun getHexCode(color: Color): String {
    return when (color) {
        is Color.Red -> color.hexCode
        is Color.Green -> color.hexCode
        is Color.Blue -> color.hexCode
    }
}

fun main() {
    println(getHexCode(Color.Red)) // #FF0000
    println(getHexCode(Color.Green)) // #00FF00
    println(getHexCode(Color.Blue)) // #0000FF
}
```

[Kotlin Tips : Keep Your Code Clean | by Mayur Waghmare | Aug, 2024 | Medium](https://medium.com/@waghmaremayur855/kotlin-tips-keep-your-code-clean-d08a3fcfbb5e)