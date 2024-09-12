## [译] - Jetpack Compose: Best Practices 最佳实践
Jetpack Compose is a modern UI toolkit for Android app development that allows you to create beautiful and efficient user interfaces with ease. However, as with any new technology, it’s essential to follow best practices to ensure that your app is scalable, maintainable, and easy to debug.  

Jetpack Compose 是一款用于 Android 应用程序开发的现代 UI 工具包，可让您轻松创建美观且高效的用户界面。然而，与任何新技术一样，必须遵循最佳实践，以确保您的应用程序可扩展、可维护且易于调试。

In this article, we’ll cover some best practices for a Jetpack Compose project.  
在本文中，我们将介绍 Jetpack Compose 项目的一些最佳实践。

> **_By following these best practices, you’ll be able to:  
> 通过遵循这些最佳实践，您将能够：_**

-   Create a consistent and visually appealing user interface  
    创建一致且具有视觉吸引力的用户界面
-   Ensure that your app is accessible to all users  
    确保所有用户都可以访问您的应用
-   Handle gestures and animations with ease  
    轻松处理手势和动画
-   Write clean and efficient code  
    编写干净高效的代码
-   Debug and test your app with confidence  
    充满信心地调试和测试您的应用程序

## 1\. Keep Composable Functions Simple and Focused 保持可组合函数简单且集中 
-   Break down complex UI components into smaller, reusable composable functions.  
    将复杂的 UI 组件分解为更小的、可重用的可组合函数。
-   Each composable function should have a single responsibility and a clear, descriptive name.  
    每个可组合函数都应该有一个单一的职责和一个清晰的描述性名称。

```kotlin
@Composable
fun UserProfile() {
    Column {
        UserAvatar()
    }
}

@Composable
fun UserAvatar() {
    Image(
        painter = painterResource(R.drawable.avatar),
        contentDescription = "User Avatar"
    )
}
```

## 2\. Use Stable Composable Functions 使用稳定的可组合函数 
-   Use the **remember** function to cache the result of a composable function and prevent unnecessary recompositions.  
    使用`remember`函数来缓存可组合函数的结果并防止不必要的重组。
-   Use the **derivedStateOf** function to create a stable state that depends on other states.  
    使用`derivedStateOf`函数创建依赖于其他状态的稳定状态。

```kotlin
@Composable
fun UserList(users: List<User>) {
    val sortedUsers = remember(users) { users.sortedBy { it.name } }
    LazyColumn {
        items(sortedUsers.size) { user ->
            UserItem(user)
        }
    }
}
```

## 3\. Minimize Side Effects 尽量减少副作用

-   Avoid making **network requests or database queries** directly from composable functions.  
    避免直接从可组合函数发出网络请求或数据库查询。
-   Use the **LaunchedEffect** function to launch a coroutine that performs a side effect.  
    使用 LaunchedEffect 函数启动执行副作用的协程。

```kotlin
@Composable
fun UserList(users: List<User>) {
    LaunchedEffect(Unit) {
        viewModel.loadUsers()
    }
    LazyColumn {
        items(users) { user ->
            UserItem(user)
        }
    }
}
```

## 4\. Use CompositionLocal 4.使用CompositionLocal

-   Use **CompositionLocal** to provide a value to a composable function and its children.  
    使用**CompositionLocal**为可组合函数及其子函数提供值。
-   Use **LocalContext** to access the current context.  
    使用**LocalContext**访问当前上下文。

```kotlin
val LocalCounter = compositionLocalOf { 0 }

@Composable
fun CounterDisplay() {
    val count = LocalCounter.current
    Text("Count: $count")
}

@Composable
fun MyApp() {
    CompositionLocalProvider(LocalCounter provides 10) {
        CounterDisplay()
    }
}
```

## 5\. Use Modifier 使用Modifier

-   Use the **Modifier** class to apply styles and layouts to composable functions.  
    使用**Modifier**类将样式和布局应用于可组合函数。
-   Use the **then** function to chain multiple modifiers together.  
    使用**then**函数将多个修饰符链接在一起。

```
@Composable
fun UserItem(user: User) {
    Text(
        text = user.name,
        modifier = Modifier
            .padding(16.dp)
            .background(Color.White)
            .clickable { /* handle click */ }
    )
}
```

## 6\. Use State and MutableState 使用State和MutableState
-   Use the **State** class to create a state that can be observed by composable functions.  
    使用**State**类创建可由可组合函数观察的状态。
-   Use the **MutableState** class to create a state that can be updated.  
    使用**MutableState**类创建可更新的状态。

```kotlin
@Composable
fun UserList(users: List<User>) {
    var selectedUser by remember { mutableStateOf<User?>(null) }
    LazyColumn {
        items(users.size) { user ->
            UserItem(user, selected = user == selectedUser) {
                selectedUser = user
            }
        }
    }
}
```

## 7\. Use Custom Modifiers 使用自定义修饰符

-   Use the **Modifier** class to create a custom modifier.  
    使用**Modifier**类创建自定义修饰符。
-   Use the **then** function to chain multiple modifiers together.  
    使用**then**函数将多个修饰符链接在一起。

```kotlin
fun Modifier.customModifier(): Modifier {
    return this
        .padding(16.dp)
        .clip(RoundedCornerShape(8.dp))
        .background(Color.Blue)
}

@Composable
fun CustomModifierExample() {
    Text(
        text = "Hello, Compose!",
        modifier = Modifier.customModifier()
    )
}
```

Thank you for taking the time to read this article. If you found the information valuable, please consider giving it a clap or sharing it with others who might benefit from it.  
感谢您花时间阅读这篇文章。如果您发现该信息有价值，请考虑点赞或与可能从中受益的其他人分享。

[原文链接]([Jetpack Compose: Best Practices. Jetpack Compose is a modern UI toolkit… | by Mayur Waghmare | Mobile Innovation Network | Sep, 2024 | Medium](https://medium.com/mobile-innovation-network/jetpack-compose-best-practices-7a25e6c182ac))