---
layout: post
title: "[자료구조 & 알고리즘] 우선순위 큐(Priority Queue) 알아보기 with Queue (Swift)"
color: rgb(50, 170, 250)
tags: [자료구조, 알고리즘, 우선순위 큐, Queue, Heap]
---

# [자료구조] 우선순위 큐 알아보기 (Swift)
오늘은 **큐(Queue)**자료구조와 함께 **우선순위 큐(Priority Queue)**에 대해서 알아보겠습니다.  
이번 우선순위 큐에 대한 설명과 구현내용은 Heap자료구조가 대부분 이므로 먼저 **Heap**에 대한 내용을 다 알고 있어야 합니다.  

[힙(Heap) 알아보기 글 바로가기](https://sangmin-jeon.github.io/2023/11/09/Algorithm-DataStructure-Heap.html)

## - 우선순위 큐(Priority Queue)란  

> 컴퓨터 과학에서, **우선순위 큐(Priority queue)**는 평범한 큐나 스택과 비슷한 축약 자료형이다.   
그러나 각 원소들은 우선순위를 갖고 있다.   
우선순위 큐에서, 높은 우선순위를 가진 원소는 낮은 우선순위를 가진 원소보다 먼저 처리된다.  
만약 두 원소가 같은 우선순위를 가진다면 그들은 큐에서 그들의 순서에 의해 처리된다.  
...  
[Wiki 우선순위 큐(Priority queue) 내용 참고](https://ko.wikipedia.org/wiki/%EC%9A%B0%EC%84%A0%EC%88%9C%EC%9C%84_%ED%81%90)

간단하게 설명 하자면 기존의 큐(queue)에서 각 요소들은 우선순위를 가지고 있고 dequeue시 먼저 들어온 순서가 아니라  
우선순위에 따라 처리 된다는 내용입니다.  
<span style="color:green">우선순위가 높은 순서대로 데이터 삭제가 이루어집니다.</span>  

우선순위 큐에 대한 설명에 앞서 먼저 간단하게 **큐(Queue)**에 대해 설명하겠습니다. 


## - 큐(Queue)란  

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/ad80ef60-82bf-4537-a64b-89ac1a338655)
 
**큐(Queue)**는 가장 먼저 들어간 데이터가 먼저 나오는 **FIFO(First In First Out)**의 특성을 가지는 자료구조입니다.  

큐에서 데이터를 **삽입**하는 것을 **enqueue**, 데이터를 **삭제**하는 작업을 **dequeue**라고 하며  
enqueue와 dequeue시 **시간 복잡도**는 $$O(1)$$로 간단하며 상수 시간에 수행 됩니다.  

~~~swift
- Swift에서는 배열로 queue를 간단하게 구현 할 수 있습니다.

// queue 생성
var queue = [Int]()

// enqueue -> 1, 2, 3
queue.append(1)
queue.append(2)
queue.append(3)
print(queue) // queue = [1, 2, 3]

// dequeue -> 1
queue.removeFirst()
print(queue) // queue = [2, 3]

~~~

## - 우선순위 큐의 데이터 삽입(enqueue)과 삭제(dequeue)

Heap자료구조 사용하여 구현한 우선순위 큐의 데이터 삽입과 삭제 방식은 Heap자료구조와  
같다고 보시면 됩니다.  

이번 예제에서는 **최소 Heap**의 조건으로 **우선순위 큐**를 구현하여 설명하였습니다.  

#### 1. 요소 삽입(enqueue)

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/cb5e5875-50d3-4bd7-a96f-597528966a38)

#### 2. 요소 삭제(dequeue)

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/f9ff3d05-264c-4023-be9a-7c996e2b8681)

## - 우선순위 큐 구현(Swift)
우선순위 큐의 경우 Heap자료구조로 구현하였기에  
저번 Heap포스팅때 설명했던  Heap구현 내용과 같습니다.   

#### 1. 요소 삽입(enqueue)
~~~swift
public mutating func enqueue(_ element: T) {
    heap.append(element)
    var currentIndex = heap.count - 1
    while currentIndex > 0 {
        let parentIndex = (currentIndex - 1) / 2
        if order(heap[currentIndex], heap[parentIndex]) {
            heap.swapAt(currentIndex, parentIndex)
            currentIndex = parentIndex
        } 
        else {
            break
        }
    }
}
~~~
1. `elements` 배열의 끝에 새로운 요소를 추가합니다. 
2. `currentIndex`는 현재 새로 추가된 요소의 인덱스를 나타냅니다.  
3. `parentIndex`는 `currentIndex`의 부모 노드의 인덱스를 계산합니다.  
    <span style="color:green">노드가 i일때 heap배열의 인덱스 관계에 대한 설명을 참고 해주세요.</span>  
4. `order` 클로저를 사용하여 현재 요소와 부모 노드를 비교합니다. 만약 현재 요소가 부모보다 우선순위가 높다면  
    (`order`의 결과가  `true`라면), 두 요소의 위치를 바꿉니다.  
5. `currentIndex`를 부모 노드의 인덱스로 업데이트하고, 이 과정을 계속 반복합니다.  
6. 만약 현재 요소가 부모보다 우선순위가 낮다면(`order`의 결과가 `false`라면), 루프를 중단합니다.

#### 1. 요소 삭제(dequeue)
~~~swift
public mutating func dequeue() -> T? {
    if heap.isEmpty {
        return nil
    }
    if heap.count == 1 {
        return heap.removeFirst()
    }
    let first = heap[0]
    heap[0] = heap.removeLast()
    var currentIndex = 0
    while true {
        let leftChildIndex = 2 * currentIndex + 1
        let rightChildIndex = 2 * currentIndex + 2
        var nextIndex = currentIndex
        if leftChildIndex < heap.count && order(heap[leftChildIndex], heap[nextIndex]) {
            nextIndex = leftChildIndex
        }
        if rightChildIndex < heap.count && order(heap[rightChildIndex], heap[nextIndex]) {
            nextIndex = rightChildIndex
        }
        if nextIndex == currentIndex {
            break
        }
        heap.swapAt(currentIndex, nextIndex)
        currentIndex = nextIndex
    }
    return first
}

~~~
1. 최상단 요소를 `first`에 저장합니다.  
2. 힙의 마지막 요소를 첫 번째로 이동시킵니다.  
3. 왼쪽 자식과 오른쪽 자식의 인덱스를 계산합니다.  
4. `nextIndex` 변수를 현재 인덱스로 초기화합니다.  
5. 왼쪽 자식과 오른쪽 자식 중 더 우선순위가 높은 자식을 선택하여 `nextIndex`를 갱신합니다.  
6. 만약 `nextIndex`가 현재 인덱스와 같다면 루프를 종료합니다.  
7. 그렇지 않으면 현재 인덱스와 선택된 자식 위치를 스왑합니다.  
8. `currentIndex`을 `nextIndex`로 갱신합니다.  
9. while 루프를 통해 위 작업을 반복합니다.
10. 최상단 요소인 `first`를 반환합니다. 

#### 3. 우선순위가 가장 높은 요소 반환(peek)
~~~swift
public func peek() -> T? {
    return heap.first
}
~~~
1. 현재 우선순위 큐에서 요소를 제거하지 않고 우선순위가 가장 높은 요소를 반환합니다.


### 전체 코드
~~~swift
import Foundation

public struct PriorityQueue<T: Comparable> {
    private var heap: [T] = []
    private let order: (T, T) -> Bool
    
    public init(order: @escaping (T, T) -> Bool) {
        self.order = order
    }
    
    public var isEmpty: Bool {
        return heap.isEmpty
    }
    
    public var count: Int {
        return heap.count
    }
    
    public mutating func enqueue(_ element: T) {
        heap.append(element)
        var currentIndex = heap.count - 1
        while currentIndex > 0 {
            let parentIndex = (currentIndex - 1) / 2
            if order(heap[currentIndex], heap[parentIndex]) {
                heap.swapAt(currentIndex, parentIndex)
                currentIndex = parentIndex
            } 
            else {
                break
            }
        }
    }
    
    public mutating func dequeue() -> T? {
        if heap.isEmpty {
            return nil
        }
        if heap.count == 1 {
            return heap.removeFirst()
        }
        let first = heap[0]
        heap[0] = heap.removeLast()
        var currentIndex = 0
        while true {
            let leftChildIndex = 2 * currentIndex + 1
            let rightChildIndex = 2 * currentIndex + 2
            var nextIndex = currentIndex
            if leftChildIndex < heap.count && order(heap[leftChildIndex], heap[nextIndex]) {
                nextIndex = leftChildIndex
            }
            if rightChildIndex < heap.count && order(heap[rightChildIndex], heap[nextIndex]) {
                nextIndex = rightChildIndex
            }
            if nextIndex == currentIndex {
                break
            }
            heap.swapAt(currentIndex, nextIndex)
            currentIndex = nextIndex
        }
        return first
    }
    
    public func peek() -> T? {
        return heap.first
    }
}

~~~