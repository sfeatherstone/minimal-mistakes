---
title: "Concatenating arrays in Koltin"
excerpt: "Exploring different ways to concatenate Arrays in Kotlin."
header:
  image: /assets/images/arrays.jpg
  
---


Adding the contents of one array to another would seem a very simple task. It Kotlin it’s a little bit more complicated…<br>
In my case I wanted to add one additional element to the end of an array. This is what I found.<br>
First up: The array size is immutable. This is the same as Java or C++ as memory is only allocated for the size the array is, so adding more requires a new array to be created(allocated), and the contents of the two arrays copied in. So lets try this with IntArray:

```kotlin
//Copy using arraycopy
fun catTwoIntArrays1(array1 :IntArray, array2 :IntArray) : IntArray {
    val newArray = IntArray(array1.size + array2.size)
    System.arraycopy(array1, 0, newArray, 0 , array1.size)
    System.arraycopy(array2, 0, newArray, array1.size , array2.size)
    return newArray
}
```

Here we are creating a new array with and using the Java function to copy the original arrays into the new array.



---

Next we use for loops to do the copying:


```kotlin
//Copy using for loops
fun catTwoIntArrays2(array1 :IntArray, array2 :IntArray) : IntArray {
    val newArray = IntArray(array1.size + array2.size )

    for ((index, value) in array1.withIndex()) {
        newArray[index] = value
    }

    for ((index, value) in array2.withIndex()) {
        newArray[index + array1.size] = value
    }
    return newArray
}
```

---

Next we are going to use the Array constructor to do the element copying

```kotlin
//Copy using the lamda in the constructor
fun catTwoIntArrays3(array1 :IntArray, array2 :IntArray) : IntArray {
    return IntArray(array1.size + array2.size,
            { if (it<array1.size) array1[it] else array2[it-array1.size]})
}
```

The lambda passed as the second parameter to the constructor fills the new array. The previous two examples could not use the Array<> because the constructor requires a lambda to construct the contents. But with this version I can now use Array<>

```kotlin
//Copy using the lamda in the constructor
inline fun <reified T> catTwoArrays3(array1 :Array<T>, array2 :Array<T>) : Array<T> {
    return Array<T>(array1.size + array2.size,
            { if (it<array1.size) array1[it] else array2[it-array1.size]})
}
```

All of this will have complexity of O(n+m) and a size of O(n+m) because we are creating a new array, and coping element from both into the new array.


---

My next though is could I be a bit lazy about this. Could I do this through a delegate? The delegate would need to handle the get operator overloading, so it seemed this was the wrong approach, but I simple class could do it.

```kotlin
//Create a class that mimics an immutable Array<>
class CatTwoArrays<T>(internal val array1 : Array<T>, internal val array2 : Array<T>) {
    public operator fun get(index: Int): T {
        return if (index < array1.size) array1[index] else array2[index - array1.size]
    }

    public val size: Int = array1.size + array2.size

    internal inner class CatTwoArraysIterator() : Iterator<T> {
        internal var position = 0

        override fun hasNext(): Boolean = (position >= 0) && (position < array1.size + array2.size)

        override fun next() = CatTwoArrays@get(position++)
    }

    public operator fun iterator(): Iterator<T> = CatTwoArraysIterator()
}
```

Here I am saving the two arrays into internal var’s. I then overload the `size` and `get` operator. I also provide a `Iterator`.
Size will not increase and construction complexity will be `O(1)`


---

Takeaways
`Array<T>` and `IntArray`/`ByteArray`/`ShortArray`/`CharArray`/`BooleanArray`/etc… all have simular interfaces, but share no inheritance and are also final. So my custom Array requires it’s own extension functions to be defined.
I should be looking at `ArrayList<>` which is more powerful, yet still has the continuous buffer which is super efficient with modern processors.

Links
[Gist Full Source](https://gist.github.com/sfeatherstone/79dd77329b7d27470e6b1d7f985cd35d)
[Research](https://www.programiz.com/kotlin-programming/examples/concatenate-two-arrays) / [Alternatives](https://rosettacode.org/wiki/Array_concatenation#Kotlin)


[Original post](https://medium.com/@ubikspace/concatenate-arrays-in-koltin-4679831b788d)