---
title: "Kotlin: Declaration Order matters"
excerpt: "Getting the declaration order wrong in Kotlan can lead to a NPE."
header:
  image: /assets/images/facepalm.jpg
---


I encountered a NPE implementing the new Android `ViewModel` class recently. Fixing the problem took a little longer due something I didn’t understand about Kotlin until now.
Declaring class member values after the init block will not be initialised when the `init` block is executed.
Example:

```kotlin
class MyClass {
    init {
        updateAnswer("42")
    }

    val answer by lazy { MutableLiveData<String>() }

    fun updateAnswer(newAnwser: String) {
        answer.value = newAnwser
    }
}
```

This will cause a NPE as as the `answer` delegate has not been initialised in the `init` block.
Attempting to do this:

```kotlin
class MyClass {
    init {
        answer.value = "42"
    }

    val answer by lazy { MutableLiveData<String>() }

}
```

will be caught by the editor and compiler.
This can be easily fixed by changing the order in which you declare the data member.

```kotlin
class MyClass {
    val answer by lazy { MutableLiveData<String>() }
    init {
        updateAnswer("42")
    }

    fun updateAnswer(newAnwser: String) {
        answer.value = newAnwser
    }
}
```

This time the `answer` delegate will be set in the `init` block

Thanks for reading, but I would like to point out that the example where I caught this was more complicated than this simplified example. My example was using [Eventbus](http://greenrobot.org/eventbus/) which I have written about in a previous post. My example looked a bit more like the following:

```kotlin
class MyClass {
    init {
        EventBus.getDefault().register(this)
    }

    val answer by lazy { MutableLiveData<MyDomainObject>() }

    @Subscribe(sticky = true, threadMode = ThreadMode.BACKGROUND)
    protected fun bind(newAnwser : MyDomainObject) {
        answer.value = newAnwser;
    }
}
```

I didn’t realise that by this time `Eventbus` already had a `MyDomainObject` and thus would call the `bind()` function synchronously from the `EventBus.getDefault().register(this)` call.

[Original post](https://medium.com/@ubikspace/kotlin-declaration-order-matters-9b2788567a78)