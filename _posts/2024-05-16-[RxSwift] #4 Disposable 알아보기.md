---
layout: post
title: "[RxSwift] #4 Disposable 알아보기"
color: rgb(50, 170, 250)
tags: [iOS, Swift, UIKit, RxSwift]
---

<div style="text-align:center;">
    <img src="https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/3e7c5cb2-77b1-4f4b-9c25-efc881787c5d" width="50%">
</div>

<br>

안녕하세요 👋  
오늘은 RxSwift에서 리소스를 정리해주는 Disposable에 대해서 알아보겠습니다.   
Disposable은 Observable의 구독을 중단하고 관련 리소스를 해제하는 데 사용되는 중요한 개념입니다.   

<br>

## Disposable이란
---
RxSwift에서 Disposable은 Observable의 생명 주기 동안 사용된 자원을 정리하는 역할을 합니다.   
구독이 더 이상 필요하지 않을 때 구독을 중지하고 메모리 누수를 방지하기 위해 사용됩니다.  

Observable의 생명 주기는 크게 다음과 같이 세 단계로 나뉩니다.  
1. **생성(Create)**: Observable은 어떤 데이터를 발행할지 정의됩니다. 하지만 이 단계에서는 아직 아무런 데이터도 발행하지 않습니다.  
2. **구독(Subscribe)**: 실제로 데이터를 발행하기 시작합니다. 이 시점에 Observer가 Observable에 붙어서 데이터 스트림을 받기 시작합니다.  
3. **종료(Dispose)**: Observable이 더 이상 데이터를 발행하지 않고 종료됩니다. 구독은 이 시점에서 해제됩니다.  

Observable의 생명 주기(life Cycle)을 저번에도 설명을 했었는데요.  
completed나 error 이벤트가 Observable의 생명 주기 마지막에 이루어진다고 했습니다.  
명시적으로 Observable에서 이 두 이벤트를 발생하면 리소스 정리를 따로 해줄 필요가 없습니다.  
말했듯이 이 두 이벤트는 Observable의 종료를 의미하고 종료된 Observable은 자동으로 메모리에서 해제되기 때문이죠.  

하지만, 그렇지 않다면 수동으로 Observable를 종료 시키고 정리해줄 무언가가 필요한데 그것이 Disposable 객체 입니다.  

(참고로 Rx에서는 Observable에서 명시적으로 completed나 error 이벤트를 발생해도 Disposable를 통해 따로 정리해주라고 나와있습니다.)  

<br>

## Disposable의 호출 순서
---

~~~swift
let observable = Observable<Int>.create { (observer) -> Disposable in
    observer.onNext(1)
    observer.onNext(2)
    observer.onNext(3)
    observer.onCompleted()

    return Disposables.create()
}


observable.subscribe({ event in
    print(event)
}).disposed(by: DisposeBag())

~~~

위 코드를 그냥 실행시켜보면 

~~~bash
next(1)
next(2)
next(3)
completed
~~~

이렇게 출력됩니다.  
위에서 Disposable은 Observable의 맨 마지막에 모든 리소스가 제거된 뒤 호출된다고 했지만  
completed이벤트가 발생되고 리소스가 정리된 후에도 Disposable은 출력 되지 않았죠.  

여기서 Disposed가 출력되지 않은 이유는 Disposed는 Observable이 전달하는 이벤트가 아니기 때문입니다.  
Observable은 next, completed, error 이 3가지 이벤트들만 전달합니다.  

<br>

~~~swift
/**
Subscribes an event handler to an observable sequence.

- parameter on: Action to invoke for each event in the observable sequence.
- returns: Subscription object used to unsubscribe from the observable sequence.
*/
/**
    Subscribes an element handler, an error handler, a completion handler and disposed handler to an observable sequence.
    
    - parameter onNext: Action to invoke for each element in the observable sequence.
    - parameter onError: Action to invoke upon errored termination of the observable sequence.
    - parameter onCompleted: Action to invoke upon graceful termination of the observable sequence.
    - parameter onDisposed: Action to invoke upon any type of termination of sequence (if the sequence has
    gracefully completed, errored, or if the generation is canceled by disposing subscription).
    - returns: Subscription object used to unsubscribe from the observable sequence.
    */
public func subscribe(onNext: ((Element) -> Void)? = nil, 
                    onError: ((Swift.Error) -> Void)? = nil, 
                    onCompleted: (() -> Void)? = nil, 
                    onDisposed: (() -> Void)? = nil) -> Disposable {
        let disposable: Disposable
        
        if let disposed = onDisposed {
            disposable = Disposables.create(with: disposed)
        }
        else {
            disposable = Disposables.create()
        }
        
        #if DEBUG
            let synchronizationTracker = SynchronizationTracker()
        #endif
        
        let callStack = Hooks.recordCallStackOnError ? Hooks.customCaptureSubscriptionCallstack() : []
        
        let observer = AnonymousObserver<Element> { event in
            
            #if DEBUG
                synchronizationTracker.register(synchronizationErrorMessage: .default)
                defer { synchronizationTracker.unregister() }
            #endif
            
            switch event {
            case .next(let value):
                onNext?(value)
            case .error(let error):
                if let onError = onError {
                    onError(error)
                }
                else {
                    Hooks.defaultErrorHandler(callStack, error)
                }
                disposable.dispose()
            case .completed:
                onCompleted?()
                disposable.dispose()
            }
        }
        return Disposables.create(
            self.asObservable().subscribe(observer),
            disposable
        )
}
~~~

위 코드는 Observer가 이벤트를 받기위해 사용하는 subscribe메소드 입니다.  
파라미터로 여러 Event 핸들러를 받고 Disposable을 리턴한다는 것이 보이시나요?  

설명에서도 "returns: Subscription object used to unsubscribe from the observable sequence." 라고 나와 있습니다.  
번역하면 "Observable sequence 구독을 취소하는 데 사용되는 객체"라고 하는데 이것이 Disposable이죠.  

event를 받는 switch문을 보시면 error와 completed 이벤트일 경우 해당 이벤트를 발생시킨 후 명시적으로 dispose()를 호출하는 것을 볼 수 있습니다.  
next이벤트 -> completed, error -> disposed 이 순서로 동작 된다는 것입니다.  

<br>

~~~swift
let _ = Observable.from([1, 2, 3])
    .subscribe(onNext: {
        print($0)
    }) { error in
        print("Error: \(error)")
    } onCompleted: {
        print("Completed")
    } onDisposed: {
        print("Disposed")
    }

~~~

위 코드는 from 연산자를 사용해 (1, 2, 3)을 방출 후 이벤트를 받는 코드인데요.   
**next이벤트 -> completed, error -> disposed** 순서대로 동작하고 onDisposed 클로저를 통해 Observable의 모든 리소스가 제거된 뒤   
호출되는지를 확인 해보았습니다.  

~~~bash
1
2
3
Completed
Disposed

~~~

출력을 보면 1, 2, 3이 출력된 후 Completed 되어 Observable이 종료된 후 리소스가 정리가 되었고 마지막에 Disposed이 출력 되었습니다.  

이것이 Rx에서 Observable을 **구독(subscribe)**하는 시점에 맞춰 이벤트를 **방출(next)**하고 방출된 이벤트를 받아 **완료(completed)** 후 종료되는 시점에 
리소스를 **정리(disposed)**하는 전체적인 과정입니다.  


<br>

## DisposeBag 을 사용하자 
---
프로젝트에서 Rx를 사용하면 객체 내에서 하나의 subscribe만 사용하지는 않을것 입니다.  
Observable은 여러개의 Observer가 구독할 수도 있고 상황에 따라 여러개의 Observable을 사용할 수도 있죠. 
Observable을 구독할 때마다 Disposable 객체가 생성 되는데 이때 마다 각각 disposed를 사용하여 리소스를 정리하는 방식보다  
하나의 객체에서 한번에 관리하면 편할거 같은데 이를 해주는 객체가 바로 DisposeBag입니다.  

<br>

#### DisposeBag의 동작 방식
~~~swift
/**
Thread safe bag that disposes added disposables on `deinit`.

This returns ARC (RAII) like resource management to `RxSwift`.

In case contained disposables need to be disposed, just put a different dispose bag
or create a new one in its place.

    self.existingDisposeBag = DisposeBag()

In case explicit disposal is necessary, there is also `CompositeDisposable`.
*/
public final class DisposeBag: DisposeBase {
    private var lock = SpinLock()
    
    // state
    private var disposables = [Disposable]()
    private var isDisposed = false

    ...

}

~~~ 

위 코드는 DisposeBag의 내부 구조인데요.  
주석의 설명을 보면 Swift에서 사용되는 **ARC**와 유사한 방식으로 리소스를 관리하며  
**Thread safe bag**이라고 나와있는데 Thread safe하게 멀티 스레드 환경에서 **SpinLock**방식을 사용하여 동시 접근 문제를 해결한다고 합니다. 

그리고 DisposeBag객체 속성으로 선언되어 있는 disposables이라는 변수를 보면 Disposable타입의 **배열**인것을 알 수 있습니다.  
이 변수에 Disposable객체들을 저장하여 관리하는것이죠.  
그래서 Bag(가방)이라고 표현하는것 같습니다.  

~~~swift
class ViewController: UIViewController {
    
    private let dispoedBag = DisposeBag()

    override func viewDidLoad() {
        super.viewDidLoad()
        setObserver()

    }
    
    func setObserver() {
        let one = 1
        let two = 2
        let three = 3
        let observable1: Observable<Int> = Observable<Int>.just(one)
        
        observable1.subscribe { event in
            print(event)
        }.disposed(by: dispoedBag)
        
        // 2
        let observable2 = Observable.of(one, two, three)
        
        observable2.subscribe { event in
            print(event)
        }.disposed(by: dispoedBag)
        
        
        let observable3 = Observable.from([one, two, three])
        
        observable3.subscribe { event in
            print(event)
        }.disposed(by: dispoedBag)
        
        
    }

~~~

setObserver 메소드를 보면 총 3개의 observable들이 있고 각 observable들을 subscribe로 구독하여 이벤트를 받고 있습니다.  
그렇다면 총 3개의 Disposable 객체가 생성되었고 이 객체들을 하나의 배열에 담아 관리하는것이  
맨 위에 속성으로 선언한 dispoedBag입니다.  
dispoedBag 객체의 인스턴스를 생성하여 관리 하는것입니다.  

<br>

#### 그럼 DisposeBag은 정확히 언제 Disposable객체들을 해제시키는 것일까요?  

~~~swift
public final class DisposeBag: DisposeBase {
    ...

    /// This is internal on purpose, take a look at `CompositeDisposable` instead.
    private func dispose() {
        let oldDisposables = self._dispose()

        for disposable in oldDisposables {
            disposable.dispose()
        }
    }
    

    private func _dispose() -> [Disposable] {
        self.lock.performLocked {
            let disposables = self.disposables
            
            self.disposables.removeAll(keepingCapacity: false)
            self.isDisposed = true
            
            return disposables
        }
    }

    deinit {
        self.dispose()
    }

    ...

}

~~~

위에서 DisposeBag을 사용할때 객체 내에서 DisposeBag인스턴스를 생성하여 사용하였습니다.  
그렇다면 사용하는 객체가 ViewController라고 가정했을 때 이 ViewController가 담당하는 화면이 사라질때  
DisposeBag인스턴스도 해제 될것이고 이는 View의 생명주기와 같다고 할 수 있습니다.  

ViewController객체가 해제될 때 같이 해제된다고 이해하시면 될것 같습니다.  

그럼 DisposeBag객체가 소멸 될때(해제 될때) 호출되는 deinit 부분을 보면 내부적으로 사용되는 dispose메소드를 호출하는것을 볼 수 있습니다.     
이 dispose메소드에서 _dispose메소드를 호출하여 모든 Disposable객체를 담은 배열인 disposables을 지워 해제시키는 것입니다.  

이 모든 과정은 Rx에서 DisposeBag객체를 통해 자동으로 처리해주니 너무나도 편하죠.  
그냥 DisposeBag인스턴스를 생성 하여 사용하기만 하면 되는겁니다.  

<br>