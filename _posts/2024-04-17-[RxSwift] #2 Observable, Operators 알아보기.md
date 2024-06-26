---
layout: post
title: "[RxSwift] #2 Observable, Operators 알아보기 "
color: rgb(50, 170, 250)
tags: [iOS, Swift, UIKit, RxSwift]
---

<div style="text-align:center;">
    <img src="https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/3e7c5cb2-77b1-4f4b-9c25-efc881787c5d" width="50%">
</div>

안녕하세요 👋  
저번 글에서 RxSwift의 기본개념을 알아보았는데요.  
이제 부터 본격적으로 RxSwift에 대해 포스팅하겠습니다.  

## Rx의 구성요소
---
RxSwift의 구성요소에는 3가지가 있는데  
바로 `Observable`, `Operators`, `Schedulers` 입니다.  
`Schedulers`는 다음에 따로 포스팅 할 예정이기에 여기에서는 3요소들중 `Observable`, `Operators`를 다루겠습니다.  

<br>

## Observable이란
---
Observable = `관찰 가능한` 이라는 뜻을 가지고 있습니다.  
RxSwift에서 Observable은 데이터나 이벤트의 스트림을 나타내며, 이를 통해 관찰자가 데이터나 이벤트의 변화를 감지하고 반응할 수 있습니다.     
Observable는 next, completed, error 라는 3가지 event를 방출하는데요.   

~~~swift
/// Represents a sequence event.
///
/// Sequence grammar:
/// **next\* (error | completed)**
public enum Event<Element> {
    /// Next element is produced.
    case next(Element)
    /// Sequence terminated with an error.
    case error(Swift.Error)
    /// Sequence completed successfully.
    case completed
}

~~~
RxSwift는 이렇게 event들을 열거형으로 표현합니다.  
여기서 next를 통해 event를 방출 하고 completed와 error 이벤트 중 하나가 발생될때 까지 event를 방출할 수 있습니다.  
즉, completed와 error event는 Observable의 lifeCycle에서 가장 마지막에 전달되며 Observable의 종료를 의미합니다.   

#### 1. A next event
최신 또는 다음 데이터 값을 전달 하는 이벤트 입니다.  
Observable에서 발생한 새로운 event는 이 next로 Observer에게 전달 하게 됩니다.  

위에서 Observable은 completed와 error event가 발생하기 전까지는 계속 next event로 
데이터를 방출할 수 있다고 했죠?  
그럼 확인을 해보겠습니다.  

여기에서는 create연산자를 사용하여 Observable을 생성하고 확인 해보겠습니다.  
~~~swift
let observable = Observable<Int>.create { (observer) -> Disposable in
    observer.onNext(1)
    observer.onNext(2)
    observer.onNext(3)

    return Disposables.create()
}

observable.subscribe({ event in
    print(event) // 1, 2, 3 출력
}).disposed(by: DisposeBag())

~~~

예상 했던 대로 onNext evnet를 구독자로 잘 전달하여 1, 2, 3이 출력되는것을 알 수 있습니다.  
위 코드에서는 create연산자로 Observable를 직접 구현하여 마지막 next event방출 후에도 completed와 error event가  
발생하지 않았기 때문에 Observable이 종료되지 않고 계속 다른 event를 방출 할 수 있습니다.  

마블 다이어 그램으로 표현하자면  
![스크린샷 2024-05-13 오후 9 50 52](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/9fb2687b-9920-4ab2-bb42-6f6eafc9df98)

이런식으로 표현 할 수 있겠네요.  

#### 2. A completed event
이벤트 시퀀스를 성공적으로 종료 했음을 알립니다.  

~~~swift
let observable = Observable<Int>.create { (observer) -> Disposable in
    observer.onNext(1)
    observer.onNext(2)
    observer.onNext(3)

    observer.onCompleted()

    return Disposables.create()
}

observable.subscribe({ event in
    print(event) // 1, 2, 3 출력
}).disposed(by: DisposeBag())

~~~
여기서 onCompleted event를 발생시키면 completed이벤트 전달 후 Observable이 종료되어  
더 이상 다른 event들을 방출하지 못하게 되는것 입니다.  

~~~swift
let observable = Observable<Int>.create { (observer) -> Disposable in
    observer.onNext(1)
    observer.onNext(2)
    observer.onNext(3)

    observer.onCompleted()

    observer.onNext(4) // 방출 되지 않음

    return Disposables.create()
}

observable.subscribe({ event in
    print(event) // 1, 2, 3 출력
}).disposed(by: DisposeBag())

~~~
위 코드에서 onCompleted 이후 onNext로 4라는 요소를 방출 하고 있는데  
completed이벤트 전달 후 Observable이 종료되었기 때문에 4는 방출되지 않게 됩니다.  

#### 3. An error event
오류로 종료되었으며 다른 이벤트를 발생 시키지 않습니다.  

말 그대로 입니다.  
Observable이 이벤트를 방출 시 에러가 발생할 때 error event를 구독자에게 전달 후 Observable이 종료 됩니다.  

~~~swift
let observable = Observable<Int>.create { (observer) -> Disposable in
    observer.onError(RxError.unknown)

    return Disposables.create()
}

observable.subscribe({ event in
    print(event) // error 메세지 출력
}).disposed(by: DisposeBag())

~~~

이 또한 Observable이 종료되기 때문에 error event발생 후에는 다른 event를 방출 할 수 없습니다.  

<br>

## Operators란
---
Operator는 연산자로 Rx에서는 사용되는 메서드는 메서드라고 부르지 않고 연산자(Operator)라고 부릅니다.  
Observable에 `.(dot)`으로 접근하여 사용되는것이 연산자(Operator) 입니다.  

그럼 just, of, from 연산자 통해 Observable을 생성하면서 설명하겠습니다.  

~~~swift
let one = 1
let two = 2
let three = 3

// 1번 Observable
let observable1: Observable<Int> = Observable<Int>.just(one)

// 1번 Observable 구독자
observable1.subscribe { event in
    print(event) // 1
}.disposed(by: DisposeBag())

// 2번 Observable
let observable2 = Observable.of(one, two, three)

// 2번 Observable 구독자
observable2.subscribe { event in
    print(event) // 1, 2, 3
}.disposed(by: DisposeBag())

~~~

#### 1. just
위 코드에서 1번 Observable의 `just`는 단 하나의 요소로 관찰 가능한 시퀀스를 생성하는 연산자 입니다.  
just연산자를 통해 1이라는 데이터를 발생시킵니다.  
따라서 1번 Observable 구독자에서 1이라는 데이터를 받아 출력하게 됩니다.  
just는 가장 간단한 연산자로 하나의 요소를 생성할 때 사용됩니다.  

#### 2. of
이제 2번 Observable을 보겠습니다.  
여기서는 `of`라는 연산자를 사용하여 (1, 2, 3)을 순서대로 발생시키고 있습니다.  
여러개의 데이터를 발생하여 배열이라고 생각할 수도 있는데 타입을 확인해 보면 `Observable<Int>` 인것을 알 수 있습니다.  
그 이유는 of연산자가 전달되는 데이터들의 타입에 따라 다양한 데이터들을 가변적으로 받을 수 있기 때문입니다.  
즉, (1, 2, 3)을 배열이 아닌 개별적인 데이터를 가진 Observable을 생성하여 각각 개별적으로 처리 할 수 있게 해줍니다.   

참고로 배열을 생성하고 싶으면 
~~~swift
Observable.of([one, two, three])

~~~
이렇게 배열을 넣어서 생성하면 됩니다.  
just를 사용해서 생성할 수도 있는데 이는 just는 배열이라는 단일 요소를 생성하는 것이지 내부 데이터가 아니기 때문입니다.  

#### 3. from
~~~swift
let observable = Observable.from([one, two, three])

observable.subscribe { event in
    print(event) // 1, 2, 3 출력
}.disposed(by: DisposeBag())
~~~
이제 마지막으로 `from` 연산자인데요 from은 배열만 취급한다고 알고 계시면 됩니다.  
입력은 배열을 받지만 배열에서 개별적인 값들을 생성하는 것이기 때문에 of 연산자와 같이 `Observable<Int>` 타입이 됩니다.    

마블 다이어그램으로 표현하면 아래 이미지와 같은데요.  

<img alt="from operator" src="https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/0aae95c9-f4c7-4a0e-a999-2044f825e39d">

배열에서 요소를 뽑아 각각의 Observable을 생성하는것입니다.  


<br>