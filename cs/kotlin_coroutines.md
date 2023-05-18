# Kotlin Coroutines

## 코루틴이란?

Co(협력) + Routines(함수) → 함수의 협력을 통해 동작

함수의 일부분을 Idle 한 Thread가 수행할 수 있도록 돕는 수단으로 멀티태스킹을 위한 도구이나 Thread 는 OS 를 통해 관리되기 때문에, 함수의 협업을 통해 함수의 일부를 수행할 수 있도록 도와주는 (스레딩 위에 작성된) 프레임워크가 Coroutines 라고 볼 수 있음

코루틴은 Stackless, Stackful 유형이 있는데, 코틀린 코루틴은 이중 Stackless 유형으로 자체 스택을 가지지 않아 네이티브 스레드에 매핑되지 않는다. 따라서 Context Switching 이 필요 없는 **경량 스레드**라고도 불린다.

## 코루틴이 필요한 이유는?

안드로이드 내에서는 네트워크 작업을 통해 서버와 통신하는 작업이 많다.
이 때 메인 스레드는 네트워크 호출을 사용할 수 없기 때문에 이를 해결할 수 있는 몇몇 방법이 있다.

**코루틴은 긴 비동기작업을 처리하는 동안 다른 작업을 처리하는 동시성을 이용하는 방법**

**Callback** 사용 → callBack (Any)→ Unit 으로 Unit 함수를 호출하여  연계 작업 수행

중첩된 작업이 많아질수록 ‘콜백 지옥’이라는 코드 중첩이 생성됨

**RxJava** (리액티브 프로그래밍:환경이 변하면 동작하는 프로그래밍 을 돕는 라이브러리) 사용 →

일종의 옵저버 패턴으로, 연계 작업에 데이터를 Push 하며 함수형 프로그래밍(side effect 가 없는 으로 작업을 처리 (데드락과 동기화 문제 등 경쟁 조건 문제 해결)

콜백의 Pull 방식을 사용하지 않고 Push 방식으로 데이터를 줘 콜백 지옥 해결

**Coruintes** 사용 →

코루틴 내에 동기식처럼 작성된 작업이 비동기식으로 작동.
suspend 함수를 이용하여 일시중지, 재개가 가능한 함수를 작성하고, 이것을 CoroutineScope 내에서 실행하는 방식으로 작동한다.

## 코루틴 세부사항

Dispatcher : 코루틴이 작업을 수행할 스레드를 결정하도록 돕는 요소

Main : 안드로이드 UI 스레드

IO : 네트워크 및 디스크 관련 작업 수행

Default : CPU 를 많이 사용하는 작업을 수행

```kotlin
suspend fun fetchData(): List<Data> {
    return withContext(Dispatchers.IO) {
        // 데이터를 가져오는 작업
    }
}

fun fetchAndDisplayData() {
    CoroutineScope(Dispatchers.Main).launch {
        try {
            val data = fetchData()
            displayData(data)
        } catch (e: Exception) {
            displayError()
        }
    }
}

```

위 코드는 fetchData 를 IO 관련 스레드를 통해 처리하고, 해당 데이터를 받아 Main(UI) 스레드에서 displayData 작업을 처리할 수 있도록 작성됨

**→ UI 스레드는 차단되지 않았고, 비동기 작업과 유기적으로 ‘협력’ 하여 작동**

## Launch Vs Async

async 는 반환값이 존재하고, launch 는 작업을 수행 후 결과를 반환하지 않는다

async 코드블럭 내에서 retun@async 를 사용하여 반환한 결과를

```kotlin
val result = asyncCoroutine.await() 
```

를 통해 얻을 수 있다.

*runBlocking은 수행 결과 나올때까지 대기하는 코루틴 빌더이다. 여기서 설명하는 둘은 blocking이 없다.*

**withContext**

Async 와 Launch 가 새로운 코루틴을 생성하는데에 비해

withContext 는 Dispatcher를 지정하여 진행중인 코루틴의 context 를 전환하는 역할을 한다.

오직 suspend Function 이나 Coroutines 내에서만 실행될 수 있는 특성을 가지고 있으며,

withContext 를 통해서 suspend function 이 특정 데이터를 반환하도록 하는 것도 가능하다.

값을 반환하는 withContext 함수 doLongRunningTaskOne/Two 의 반환값으로 UI를 업데이트하는 예시이다. 예시에서 약 1초 후 결과가 표시될 것이다.

만약 각 함수가 async 의 인자로 전달되지 않았다면, withContext는 코루틴을 시작하지 않았기 때문에 각 작업이 직렬적으로 실행되어 2초 후 결과가 표시된다는 점에 주의하자.

```kotlin
GlobalScope.launch {

    val deferredOne = async {
        doLongRunningTaskOne() // 1초 소요
    }

    val deferredTwo = async {
        doLongRunningTaskTwo() // 1초 소요
    }

    val result = deferredOne.await() + deferredTwo.await()

    showResult(result) // back on UI thread

}
```

## CoroutineScope?

코루틴 작업이 존재해야하는 영역을 의미한다.

만약 Activity와 함께 코루틴 작업이 종료되어야 한다면, Activity 내에서

lifecycleScope.launch{ } 를 통해 코루틴 작업을 수행하면 된다.

viewModelScope.launch{} 도 비슷하게 작동한다. 즉 코루틴 작업이 유효한 lifeCycle을 바인딩한다고 볼 수 있겠다.

globalScope는 어떠한 job에도 바인딩되지 않는다. 애플리케이션과 함께 종료되나, 조기 취소되지 않는다. 여러 잠재 문제가 많기 떄문에 항상 로그를 남기는 등 필수적이지 않을 땐 사용을 금하자.

## 예외처리

**A. Launch 를 사용할 때의 예외처리**

try - catch 블럭 사용하기 : 예외가 발생하면 e 에 따라 catch 구문 실행

```kotlin
// Launch 사용시

someScope.launch(Dispatchers.Main){
 try {
     suspendFoo() // IO Dispatcher를 사용하는 작업
 } catch (e: Exception) {
   //ex
   Log.d(TAG,"$exception handled!")
 } 
}

// Async 사용시

val defferedData = someScope.async {
    returnAsyncValue()
}
try {
    val data = defferedData.await()
} catch (exception: Exception) {
    Log.d(TAG, "$exception handled !")
}
```

handler 사용하기 :  예외가 발생하면 handler 람다 작동

```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    Log.d(TAG, "$exception handled !")
}

someScope.launch(Dispatchers.Main + handler) {
    suspendFoo() // IO Dispatcher를 사용하는 작업
}
```

만약 launch 구문을 통해 두 네트워크 데이터 반환 작업 a() 와 b() 를 병렬적으로 처리한다고 했을 때

```kotlin
launch {
    try {
        coroutineScope {
            val usersDeferred = async {  getUsers() }
            val moreUsersDeferred = async { getMoreUsers() }
            val users = usersDeferred.await()
            val moreUsers = moreUsersDeferred.await()
        }
    } catch (exception: Exception) {
        Log.d(TAG, "$exception handled !")
    }
}
```

coroutineScope 속에 작업을 작성해야 네트워크 예외도 처리할 수 있으며

만약 한 작업이 실패해도 다른 작업은 계속하고 싶다면 새로운 Scope, supervisorScope 속에서 두 코루틴을 실행해야 할 것이다. (여러 자식을 거느리는 coroutineScope로 이해할 수 있겠다.)

```kotlin
launch {
    supervisorScope {
        val usersDeferred = async { getUsers() }
        val moreUsersDeferred = async { getMoreUsers() }
        val users = try {
            usersDeferred.await()
        } catch (e: Exception) {
            emptyList<User>()
        }
        val moreUsers = try {
            moreUsersDeferred.await()
        } catch (e: Exception) {
            emptyList<User>()
        }
    }
}
```
