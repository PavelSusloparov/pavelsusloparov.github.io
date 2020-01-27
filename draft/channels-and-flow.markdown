# Channels 

Communicate between two coroutines
They are hot - need to take care about closing them

Type of channels:
* SendChannel
* ReceiveChannel
* Channel

## Defining a channel
Val numberChannel = Channel<Int>()

## Sending to a channel
```kotlin
launch {
    for (I in 0..9) {
	    print(i)
	}
}
```

## Closing a channel
need to close a channel after 

# Flows

* Non-blocking streams
* Cold
* Emit/collect

## Emitting

```kotlin
val numbersFlow: Flow<Int> = flow {
    var number = 3 
    while (true) {
        emit(number)
        number++
    }
}
```

flowOf() - another type of building the flow.

asFlow() - extension function. Creates a flow from it's content.

## Collecting
```kotlin
runBlocking {
    numbersFlow
    .filter { number -> number.isPrime() }
    .
}
```

# Extending the flow API


