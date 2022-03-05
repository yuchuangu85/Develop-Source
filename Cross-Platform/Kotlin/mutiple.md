<h1 align="center">What is the difference between "remember" and "mutableState"</h1>



`remember` is a composable function that can be used to cache expensive operations. You can think of it as a cache which is local to your composable.

```kotlin
val state: Int = remember { 1 }
```

The `state` in the above code is immutable. If you want to change that state and also update the UI, you can use a `MutableState`. `Compose` observes any reads and writes to the `MutableState` object and triggers a [recomposition](https://developer.android.com/jetpack/compose/state#composition-and-recomposition) to update the UI.

```kotlin
val state: MutableState<Int> = remember { mutableStateOf(1) }

Text(
   modifier = Modifier.clickable { state.value += 1 },
   text = "${state.value}",
 )
```

Another variant (added in `alpha12`) called `rememberSaveable` which is similar to `remember`, but the stored value can survive process death or configuration changes.

```css
val state: MutableState<Int> = rememberSaveable { mutableStateOf(1) }
```

**Note**: You can also use property delegates as a syntactic sugar to unwrap the `MutableState`.

```css
var state: Int by remember { mutableStateOf(1) }
```

Regarding the last part of your question, if you are doing something like shown below within your composable, you are just creating a `MutableState` object.

```kotlin
val state: MutableState<Int> = mutableStateOf { 1 }
```

`MutableState` is an alternative to using `LiveData` or `Flow`. `Compose` does not observe any changes to this object by default and therefore no recomposition will happen. If you want the changes to be observed and the state to be cached use `remember`. If you don't need the caching but only want to observe, you can use `derivedStateOf`. Here is a [sample](https://android-review.googlesource.com/c/platform/frameworks/support/+/1434397/5/compose/runtime/runtime/samples/src/main/java/androidx/compose/runtime/samples/EffectSamples.kt#131) of how to use it.

## 参考

* [What is the difference between "remember" and "mutableState" in android jetpack compose?](https://stackoverflow.com/questions/66169601/what-is-the-difference-between-remember-and-mutablestate-in-android-jetpack)