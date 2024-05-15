---
layout: post
title: "[RxSwift] #3 Observer란 무엇인가 "
color: rgb(50, 170, 250)
tags: [iOS, Swift, UIKit, RxSwift]
---

<div style="text-align:center;">
    <img src="https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/3e7c5cb2-77b1-4f4b-9c25-efc881787c5d" width="50%">
</div>

안녕하세요 👋  
이전 포스팅에서 Observable에 대해서 알아보았는데요.  
글을 작성하다 보니 Observer에 대해서도 설명해야 될거 같은데 어디에 넣어야 할지 고민하다가  
이렇게 따로 다루게 되었습니다 😅  

<br>

## Observer 란? 
---
지난 포스팅의 Observable에서 next, completed, error 이벤트에 대해서 알아보았죠?  
next이벤트는 요소를 방출하고 completed과 error는 Observable의 종료를 의미한다 였습니다.  
완료되었거나 errer로 종료가 된것이지요.  

이런 이벤트들을 방출하거나 발생하면 그 이벤트들을 받아서 처리 하는 곳이 있을텐데 그것이 Observer입니다.  
Observer를 구독자라고 표현하기도 합니다.  
Observable을 구독하여 이벤트를 받는 구독자라고 생각하시면 될거 같습니다.  

<br>

![stream2](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/8893c65d-5bae-429c-9939-a7499291b4b9)

이렇게 하나의 Observable을 여러개의 Observer가 구독할 수도 있습니다.  
Swift에서 기본으로 제공하는 NotificationCenter를 생각하시면 이해가 쉬울것 같네요.  
 
<br>

## Observer가 이벤트를 받고 처리하는 방식
---
~~~swift
let observable = Observable<Int>.create { (observer) -> Disposable in
    observer.onNext(1)
    observer.onNext(2)
    observer.onNext(3)
    observer.onCompleted()

    return Disposables.create()
}

~~~

요소 (1, 2, 3)을 방출하고 completed 이벤트를 발생하는 Observable을 생성하겠습니다.  
그럼 이 Observable을 구독하여 (1, 2, 3)과 completed 이벤트를 받는 Observer를 구현해야 하는데  
subscribe으로 모든 Event를 처리하는 방법 부터 설명하겠습니다.  

<br>

~~~swift
observable.subscribe({ event in
    // event 받기
}).disposed(by: DisposeBag())

~~~

위 코드에서 event라는 subscribe로 받은 요소의 타입은 `event: Event<Int>`가 됩니다.  
공식 문서에서 Event를 살펴보면 

<br>

![스크린샷 2024-05-14 오후 11 50 28](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/4a4e32e5-fe4b-4859-beee-5ccbc545a197)

Sequence의 문법으로 next, completed, error를 나타낸다고 나와있습니다.  
아래 Topics를 보니 열거형으로 next, completed, error가 있고 Observable이 구현되어 있는 방식과 같다는것을 알 수 있습니다.  
열거형으로 되어있으니 이를 switch문으로 풀어서 확인해보면 되겠네요.  

<br>

~~~swift
observable.subscribe({ event in
    switch event {
    case .next(let elem):
        print(elem)
    case .completed:
        print("completed")
    case .error(let error):
        print(error.localizedDescription)
    }
}).disposed(by: DisposeBag())

~~~

위 코드를 실행 시키면 아래와 같이 출력됩니다.  

~~~bash
1
2
3
completed
~~~

이는 Observable이 onNext로 1, 2, 3을 순서대로 방출하고 마지막에 completed를 발생시켰고    
Observable을 구독하고 있는 subscribe에서 next타입의 1, 2, 3을 받아 출력 후 completed 이벤트를 받아 완료가 되어
completed가 출력된것입니다.  

<br>

subscribe에서 이벤트를 받는 또 다른 방법을 알아보겠습니다.  
각각의 핸들러를 사용하는 방식인데 실제 프로젝트에서는 이 방식을 많이 사용했습니다.  
event들을 구분지어 관리하기 편하기 때문입니다.  

~~~swift
observable.subscribe(
    onNext: { elem in
        print(elem)
    },
    onCompleted: {
        print("completed")
    },
    onError: { error in
        print(error.localizedDescription)
    }
).disposed(by: disposeBag)

~~~
위 코드를 보면 처음 방식과 달리 onNext, onCompleted, onError를 통해 각각의 이벤트들을  
처리하게 됩니다.  
onNext로 방출된 요소는 onNext로 들어오고, onCompleted와, onError는 각각의 핸들러로 들어오게 됩니다.  
실제로 코드를 실행해 보면   

~~~bash
1
2
3
completed
~~~

처음 방식과 똑같이 출력되는것을 확인 할 수 있습니다.   

위와 같이 구현함으로서 onNext에서 방출된 요소를 받아 처리하고, onCompleted에서 Observable이 완료된 이후 동작을  
onError에서 에러가 발생되었을때 알림을 띄우거나 화면을 전환시키는 동작을 수행하는 등  
한 곳에서 비동기 작업시 발생하는 다양한 이벤트들을 처리 할 수 있습니다.  

<br>

## 마무리
---
이렇게 Observer에 대해서 알아보았는데요.  
간단하게 설명하자면 Observable에서 데이터를 보내고 Observer로 받는다.  
라고 할수 있겠습니다.  

이제 Rx에서 리소스를 정리해주는 DisposeBag에 대해서만 다루면 Rx에서 기본이 되는 개념은 어느정도 정리가 될것 같습니다.  

<br>