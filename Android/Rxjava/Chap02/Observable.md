## Observable

<img src="../Images/Rxjava1.xVS2,x.png">

 > RxJava 1.x에서는 Observable 만 있었지만 RxJava 2.x에서는 세분화 되었다.

  - Observable

  - Maybe

  - Flowable 
- Observable (관찰자) 아직 관찰되진 않았지만, 이론을 통하여 관찰할 가능성을 의미한다.
> 라이프 사이클은 존재하지 않는다.

### RxJava의 Observable은 세 가지의 알림을 구독자에게 전달한다.
 - **onNext :** Observable의 데이터 발행을 알린다.
 - **onComplete :** 모든 데이터의 발행을 완료했음을 알림, 단 한번만 발생하며 발생한 후에는 onNext 이벤트가 못발생함.
- **onError :** Observable에 어떠한 이유로 에러가 났다는걸 알린다. onError 이벤트가 밸생하면 onNext와 onComplete 이벤트가 발생하지 않는다. 즉 Observable의 실행이 중단된다.
 

## Observable Class의 함수들

### just함수

데이터를 발행하는 가장 쉬운 방법은 기존의 자료구조를 사용하는 것이다.

just() 함수는 인자로 넣은 데이터를 차례대로 발행하려고 Observable을 생성한다. 
> 실제 발행은 subscribe() 함수를 실행해야한다.
>> 한 개의 값을 넣을 수 있고 **최대 10개의 값**을 넣을 수 있다. *단 타입은 모두 같아야한다.*<br>

<img src="../Images/just.png" width="500" height="250"></br>
중앙으 원은 Observable에서 발핸하는 데이터로 just() 함수를 거치면 입력한 원을 그대로 발행한다. 파이프(|) 표시는 모든 데이터 발행이 완료되었다는 의미이다.(onComplete 이벤트)

>인자가 N개인 just() 함수의 다이어그램

<img src="../Images/justs.png" width="500" height="250">

just() 함수로 1~6 까지의 원을 1개씩 발행한다. 모두 발행한 이후에는 완료(|) 한다.


*코드*
~~~kotlin

class FirstExampleKotlinJust {
    fun emit(){
        Observable.just(1,2,3,4,5,6).subscribe(System.out::print)
    }
}

fun main(){
    val demo=FirstExampleKotlinJust()
    demo.emit()
    
}
~~~
*결과*
~~~kotlin
1
2
3
4
5
6   
~~~

### subscribe() 함수와 Disposable 객체

Observable은 just() 등의 팩토리 함수로 데이터 흐름을 정의한 후 subsrcibe() 함수를 호출해야 실제로 데이터를 발행한다.

Rxjava는 동작시키길 원하는걸 미리 선언한다음 그것이 실행되는 시점을 조절할 수 있다.
 
>Rxjava는 선언형 프로그래밍이다.
>>먼저 선언하고 필요할 때마다 가져다 쓸 수 있다. 
>>실행할 동작을 구체적으로 명시하는 명령어 프로그래밍과 달리 선언형 프로그래밍은 단순히 목표만 선언한다

- Disposable 인터페이스의 함수
  
dispose()는 Observable에게 더 이상 데이터를 발행하지 않도록 구독을 해지하는 함수.

Observable 계약에 따르면 Observable이 onComplete 알림을 보냈을 때 자동으로 dispose()를 호출해 Observable과 구독자의 관계를 끊는다.

따라서 onCompose 이벤트가 끝나면 구독자가 별도로 dispose()를 호출할 필요가 없다.


>*코드*
```kotlin
   class ObservableNotificationsKotlin {

    fun emit() {
        val source = Observable.just("RED", "GREEN", "YELLOW")

        val d = source.subscribe(
            { v -> println("onNext() : value : $v") },
            { err -> println("onError() : err : ${err.message}") },
            { println("onComplete()") }

        )
        print("isDisposed() : "+d.isDisposed)
    }
}

fun main() {
    val demo = ObservableNotificationsKotlin()
    demo.emit()

```
>*출력*
``` kotlin
onNext() : value : RED
onNext() : value : GREEN
onNext() : value : YELLOW
onComplete()
isDisposed() : true
```

~~~
subscribe는 Observable을 찍기 위한 함수이고, Disposed는 subscribe에 있는 함수 인건가 ?? 아시면 답 부탁드립니다.
~~~

### create() 함수

just() 함수는 데이터를 넣으면 자동으로 알림 이벤트가 발생 되지만, create() 함수는
- onNext
- onComplete
- onError
   
 같은 알림을 개발자가 직접 호출해줘야 한다.
그래서 create() 함수는 개발자가 무언가를 직접 해야하는 느낌이 강하다.

just() vs create()
> just() 함수는 자동으로 데이터가 발행이 되지만, create() 함수는 개발자가 일일이 정해줘야한다.

<br>

> create() 함수 마블다이어그램

<img src="../Images/create()%20함수.png">

> 구독자 에게 알림을 발행하려면 onNext() 함수를 발행해야하며, 모두 발행한 후에는 onComplete 함수를 발행해야 한다.


[FirstExampleJava](#just함수) 예제와 다르게 Observable(Integer) 타입의 변수를 분리했는데 source 변수는 차가운 Observable입니다. 즉 첫 번째 문장으로는 실행이 안되고, 두 번째 subscribe() 함수를 호출 호출 했을 때 값이 찍힙니다. 만약 호출을 안하면 값이 안찍힙니다.

>*코드*
```kotlin
class ObservableCreateExampleKotlin {

    fun emit() {
        val source = Observable.create { emitter: ObservableEmitter<Int> ->
            emitter.onNext(100)
            emitter.onNext(200)
            emitter.onNext(300)

        }
        source.subscribe { data -> println("Result : $data") }
    }
}

fun main() {
    val demo=ObservableCreateExampleKotlin()
    demo.emit()
}
```

>*출력*
```kotlin
Result : 100
Result : 200
Result : 300
```

> 람다 형식으로 변경
```kotlin
public class subscribeLambda {
    private void emit() {
        Observable<Integer> source = Observable.create((ObservableEmitter<Integer> emitter) -> {

            emitter.onNext(100);
            emitter.onNext(200);
            emitter.onNext(300);
            emitter.onComplete();
        });
        source.subscribe(data->System.out.println("Result : " + data));
    }


    public static void main(String[] args) {

        subscribeLambda demo=new subscribeLambda();
        demo.emit();
    }
}
```
실행 결과는 위와 똑같다.
data는 변수 이름이므로 자신이 원하는 변수명을 적으면 된다.

* 리액티브 프로그래밍 에서는 람다와 래퍼런스를 적극적으로 사용하는 것이 좋다. 그리고 우선순위를 잘 고려해야한다.

> - 1. 메서드 레퍼런스로 축약할 수 있는지 확인
> - 2. 그다음 람다 표현식을 활용할 수 있는지 확인
> - 3. 1~2를 활용할 수 없으면 익명 객체나 멤버 변수로 표현


>익명 객체로 변환
```kotlin
public class subscribeAnonymously {
    private void emit() {
        Observable<Integer> source = Observable.create((ObservableEmitter<Integer> emitter) -> {

            emitter.onNext(100);
            emitter.onNext(200);
            emitter.onNext(300);
            emitter.onComplete();
        });

        source.subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer data) throws Throwable {
                System.out.println("Result : " + data);
            }
        });
    }



    public static void main(String[] args) {

        subscribeAnonymously demo=new subscribeAnonymously();
        demo.emit();
    }
}
```
실행결과는 위와 똑같다.

익명 객체는 subsribe()의 원형을 알아야 하고 Consumer(T) 클래스의 메서드도 매번 입력을 해줘어야 하므로 번거롭다. 

#### Observable.create()를 사용할 때 주의할점

- 1. Observable 이 구독해지(dispose) 되었을때 callback 을 모두 해제해야 한다. 안 그러면 메모리 누수가 생긴다 
- 2. 구독자가 구독하는 동안에만 onNext와 onComplete 이벤트를 호출해야 한다.
- 3. 애러가 발생하면 onError 이벤트로만 에러를 전달해야 한다.
- 4. 배압을 직접 처리해야 한다.

### fromArray 함수

just() 함수나 create() 함수는 단일 데이터를 처리한다. 여러 데이터를 처리해야할 때는 fromXXX 계열 함수를 사용한다.

그중에 fromArray() 함수로 배열에 있는 데이터 값을 처리해보자
>*코드*
~~~kotlin 
class ObservableFromArrayKotlin {
    fun emit(){
    val arr= arrayOf(100,200,300);
        val source= Observable.fromArray(*arr);
        source.subscribe(System.out::println);
    }
}

fun main() {
    val demo=ObservableFromArrayKotlin()
    demo.emit()
}
~~~
>*결과*
~~~kotlin
100
200
300
~~~
데이터가 차례대로 발행이 된다.