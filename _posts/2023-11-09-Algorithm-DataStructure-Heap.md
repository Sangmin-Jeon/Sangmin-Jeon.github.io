---
layout: post
title: "[자료구조 & 알고리즘] Heap 알아보기 (Swift)"
color: rgb(50, 170, 250)
tags: [알고리즘, 자료구조, Heap]
---


# [자료구조] Heap 알아보기 (Swift)
우선 순위 큐를 설명하기 전에 먼저 Heap(힙) 자료구조에 대해서 알아보겠습니다.  

## - Heap 이란
---
* 정의   
> **힙(heap)**은 **최댓값** 및 **최솟값**을 찾아내는 연산을 빠르게 하기 위해 고안된   
**완전이진트리(complete binary tree)**를 기본으로 한 자료구조(tree-based structure)입니다.  
또한, 노드 값의 대소관계는 오로지 부모노드와 자식노드 간에만 성립하며, 특히 형제 사이에는 대소관계가 정해지지 않습니다.  
...  
[Wiki Heap 내용 참고](https://ko.wikipedia.org/wiki/%ED%9E%99_(%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0))


Heap은 트리의 일종으로 **완전 이진 트리**입니다.  

#### 완전 이진 트리란     
위에서 아래로, 왼쪽에서 오른쪽으로 채워지는 트리를 말합니다.  
완전 이진 트리는 트리의 마지막 레벨 이전 레벨의 노드는 모두 채워져 있어야 하고  
따라서 완전 이진 트리의 노드의 개수는   
<span style="font-size: 0.9em; color:green">$$2^(h-1)-1 <= n <= 2^h-1$$</span> 가 됩니다.  

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/84333da8-c8ae-40a5-9d09-c02bdd13f31c)

**참고)** 포화 이진 트리(perfect binary tree)와 헷갈리지 않도록 합시다.  
포화 이진 트리란 이진 트리를 만족하면서 모든 부모 노드가 자식 노드를 갖는 트리형태를 말합니다.  
따라서 노드의 개수는 <span style="font-size: 0.9em; color:green">$$n = 2^h - 1$$</span>(h: 트리의 높이)가 됩니다.  
(한마디로 부모 노드는 왼쪽, 오른쪽 자식 노드를 모두 갖거나 모두 가지지 않거나 이어야 합니다.)

## - Heap의 종류  
---  
Heap은 조건에 따라 **최대(max) Heap**과 **최소(min) Heap**으로 나뉘어 집니다.  

<span style="color:green">
여기서는 [7, 3, 8, 1, 5]를 순서대로 모두 최소, 최대 heap에 삽입(insert)했다고 가정 하겠습니다.  
삽입(insert), 삭제(remove) 과정은 아래 코드 구현에서 같이 설명 하겠습니다.  
</span>     

배열에 힙을 저장할 때는 각 노드가 배열의 인덱스에 매핑됩니다.   
이때, 부모 노드의 인덱스와 자식 노드의 인덱스 간에는 다음과 같은 관계가 성립합니다.  
배열의 0번 인덱스는 항상 Heap의 최 상단 루트 노드의 값이 됩니다.  
~~~js
- 노드의 인덱스가 i 일 때,
1. 왼쪽 자식 노드의 인덱스: 2i + 1
2. 오른쪽 자식 노드의 인덱스: 2i + 2
3. 부모 노드의 인덱스: (i - 1) / 2
~~~
즉, 배열의 각 요소는 특정 노드에 대응되며, 부모-자식 관계는 배열의 인덱스 간의 계산으로 표현됩니다. 

<span style="color:orange">(아래에서 설명할 heap 배열은 모두 위 인덱스 간의 관계로 보시면 됩니다.)</span>  

#### 1. 최소 Heap
부모 노드가 자식 노드의 값보다 항상 작은 값을 가지는 완전 이진 트리를 최소 Heap이라 합니다.  
완전 이진 트리의 [7, 3, 8, 1, 5]을 최소 힙으로 조건으로 표현하면 [1, 3, 8, 7, 5]이 됩니다.  

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/1e91f874-521d-4c8b-bdfe-d6732ea84c17)

위 트리를 보면 부모 노드의 값은 항상 자식 노드의 값보다 작으므로   
최소 힙의 조건을 만족하는 최소 Heap인 것을 알 수 있습니다.  

#### 2. 최대 Heap
부모 노드가 자식 노드의 값보다 항상 큰 값을 가지는 완전 이진 트리를 최소 Heap이라 합니다.  
완전 이진 트리의 [7, 3, 8, 1, 5]을 최대 힙으로 조건으로 표현하면 [8, 5, 7, 1, 3]이 됩니다.   

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/25a012fe-b909-4f46-af38-d576b74c784a)

위 트리를 보면 부모 노드의 값은 항상 자식 노드의 값보다 큽니다.   
따라서, 최대 힙의 조건을 만족하는 최대 Heap인 것을 알 수 있습니다. 

## - Heap의 삽입(insert)과 삭제(remove)  
---  
예제 배열 [7, 3, 8, 1, 5]을 사용하여 그림으로 삽입(insert) 과정을 하나씩 살펴 보겠습니다.  
(위에서 설명했듯이 heap은 완전 이진 트리 이므로 위에서 아래로, 왼쪽에서 오른쪽으로 채워집니다.)  
<span style="color:green">우선 **최소 Heap**을 기준으로 설명 하겠습니다.</span>  

heap의 삽입, 삭제 과정의 시간 복잡도는 $$O(logN)$$으로 배열을 사용했을때에 비해  
매우 빠릅니다.  

배열을 사용했을 때에는 삽입이나 삭제에 따라 모든 요소를 이동시켜야 하기 때문에  
$$O(N)$$의 시간 복잡도가 발생합니다.

#### 1. 요소 삽입(insert)
**배열 [7, 3, 8, 1, 5]**에서 맨 처음 인덱스의 값인 7부터 하나씩 heap에 삽입(insert) 합니다.

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/c17293af-e236-4b35-9fd8-d82be21eccfa)

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/3a489ade-809f-40fa-a17f-af8c5964d7a1)


#### 2. 요소 삭제(remove)
**Heap = [1, 3, 8, 7, 5]**에서 요소를 삭제 해보겠습니다.  
최소 Heap에서는 가장 작고, 최대 Heap에서는 가장 큰 최상단 노드가 삭제됩니다.  

본 예제에서는 최소 Heap을 다루고 있으므로 가장 작은 최상단 노드가 삭제 됩니다.  

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/1930bdde-ebc1-4280-8e91-22deffd881f9)

## - Heap 구현(Swift)
---  
위에서 설명한 Heap자료구조를 코드로 작성 해보겠습니다.  

#### 1. Heap에 요소 추가(insert)
- **새로운 요소를 Heap에 추가하는 메서드**  

~~~swift
// Heap은 구조체 내 elements배열에 계산되어 저장 됩니다.

mutating func insert(_ element: T) {
    elements.append(element)  // 배열 끝에 요소 추가
    heapifyUp()  // 힙의 성질을 만족하도록 요소를 올림
}
...

~~~

`insert` 메서드는 새로운 요소를 힙에 추가 시키기 위해 `heapifyUp` 메소드를 호출 합니다.  
`insert` 메서드는 다음과 같은 단계로 동작합니다  
1. `elements` 배열의 끝에 새로운 요소를 추가합니다.  

2. `heapifyUp` 메서드를 호출하여 새로 추가된 요소를 힙의 올바른 위치로 이동시킵니다.   

- **Heap을 올리는 메서드**  

~~~swift
// 힙을 올리는 메서드
pirvate mutating func heapifyUp() {
    var currentIndex = elements.count - 1

    while currentIndex > 0 {
        let parentIndex = (currentIndex - 1) / 2

        // 새로 추가된 요소와 부모를 비교하여 순서를 조정
        if order(elements[currentIndex], elements[parentIndex]) {
            elements.swapAt(currentIndex, parentIndex)
            currentIndex = parentIndex
        } 
        else {
            break
        }
    }
}
...

~~~

`heapifyUp` 메서드는 힙의 성질을 유지하도록 요소를 올립니다.  
1. `currentIndex`는 현재 새로 추가된 요소의 인덱스를 나타냅니다.  

2. `parentIndex`는 `currentIndex`의 부모 노드의 인덱스를 계산합니다.  
    <span style="color:green">노드가 i일때 heap배열의 인덱스 관계에 대한 설명을 참고 해주세요.</span>  

3. `order` 클로저를 사용하여 현재 요소와 부모 노드를 비교합니다. 만약 현재 요소가 부모보다 우선순위가 높다면  
    (`order`의 결과가  `true`라면), 두 요소의 위치를 바꿉니다.  

4. `currentIndex`를 부모 노드의 인덱스로 업데이트하고, 이 과정을 계속 반복합니다.  

5. 만약 현재 요소가 부모보다 우선순위가 낮다면(`order`의 결과가 `false`라면), 루프를 중단하고   
    `insert` 메서드로 돌아갑니다.    



#### 2. Heap에 요소 삭제(remove)
- 힙에서 최상단 요소를 제거하고 반환하는 메서드
  
~~~swift
// 힙에서 최상단 요소를 제거하고 반환하는 메서드
mutating func remove() -> T? {
    guard !isEmpty else { return nil }

    if elements.count == 1 {
        return elements.removeFirst()
    }

    let first = elements[0]  // 최상단 요소 저장
    elements[0] = elements.removeLast()  // 힙의 끝에 있는 요소를 첫 번째로 이동
    heapifyDown()  // 힙의 성질을 만족하도록 요소를 내림
    return first
}

~~~
1. 최상단 요소를 `first`에 저장합니다.  
2. 힙의 마지막 요소를 첫 번째로 이동시킵니다.  
3. `heapifyDown` 메서드를 호출하여 힙의 성질을 만족하도록 요소를 내립니다.  
4. 최상단 요소인 `first`를 반환합니다.  

- Heap을 내리는 메서드

~~~swift
// 힙을 내리는 메서드
private mutating func heapifyDown() {
    var currentIndex = 0

    while true {
        let leftChildIndex = 2 * currentIndex + 1
        let rightChildIndex = 2 * currentIndex + 2
        var nextIndex = currentIndex

        // 왼쪽 자식과 오른쪽 자식 중 더 우선순위가 높은 자식 선택
        if leftChildIndex < elements.count && order(elements[leftChildIndex], elements[nextIndex]) {
            nextIndex = leftChildIndex
        }

        if rightChildIndex < elements.count && order(elements[rightChildIndex], elements[nextIndex]) {
            nextIndex = rightChildIndex
        }

        // 다음 위치가 현재 위치와 같으면 종료
        if nextIndex == currentIndex {
            break
        }

        // 현재 위치와 선택된 자식 위치를 스왑
        elements.swapAt(currentIndex, nextIndex)
        currentIndex = nextIndex
    }
}

~~~

1. 왼쪽 자식과 오른쪽 자식의 인덱스를 계산합니다.  
2. `nextIndex` 변수를 현재 인덱스로 초기화합니다.  
3. 왼쪽 자식과 오른쪽 자식 중 더 우선순위가 높은 자식을 선택하여 `nextIndex`를 갱신합니다.  
4. 만약 `nextIndex`가 현재 인덱스와 같다면 루프를 종료합니다.  
5. 그렇지 않으면 현재 인덱스와 선택된 자식 위치를 스왑합니다.  
6. `currentIndex`을 `nextIndex`로 갱신합니다.  
7. while 루프를 통해 위 작업을 반복합니다.  

### 전체 코드
~~~swift
// Heap 구조체 정의
struct Heap<T: Comparable> {
    private var elements: [T] = []   // 힙의 요소를 저장하는 배열
    private let order: (T, T) -> Bool // 힙의 순서를 결정하는 클로저

    // 초기화 메서드
    init(order: @escaping (T, T) -> Bool) {
        self.order = order
    }

    // 힙이 비어 있는지 확인하는 속성
    var isEmpty: Bool {
        return elements.isEmpty
    }

    // 힙의 요소 개수를 반환하는 속성
    var count: Int {
        return elements.count
    }

	// 새로운 요소를 힙에 추가하는 메서드
    mutating func insert(_ element: T) {
        elements.append(element)  // 배열 끝에 요소 추가
        heapifyUp()  // 힙의 성질을 만족하도록 요소를 올림
    }

    // 힙에서 최상단 요소를 제거하고 반환하는 메서드
    mutating func remove() -> T? {
        guard !isEmpty else { return nil }

        if elements.count == 1 {
            return elements.removeFirst()
        }

        let first = elements[0]  // 최상단 요소 저장
        elements[0] = elements.removeLast()  // 힙의 끝에 있는 요소를 첫 번째로 이동
        heapifyDown()  // 힙의 성질을 만족하도록 요소를 내림
        return first
    }

    // 힙을 올리는 메서드
    pirvate mutating func heapifyUp() {
        var currentIndex = elements.count - 1

        while currentIndex > 0 {
            let parentIndex = (currentIndex - 1) / 2

            // 새로 추가된 요소와 부모를 비교하여 순서를 조정
            if order(elements[currentIndex], elements[parentIndex]) {
                elements.swapAt(currentIndex, parentIndex)
                currentIndex = parentIndex
            } 
            else {
                break
            }
        }
    }

    // 힙을 내리는(private) 메서드
    private mutating func heapifyDown() {
        var currentIndex = 0

        while true {
            let leftChildIndex = 2 * currentIndex + 1
            let rightChildIndex = 2 * currentIndex + 2
            var nextIndex = currentIndex

            // 왼쪽 자식과 오른쪽 자식 중 더 우선순위가 높은 자식 선택
            if leftChildIndex < elements.count && order(elements[leftChildIndex], elements[nextIndex]) {
                nextIndex = leftChildIndex
            }

            if rightChildIndex < elements.count && order(elements[rightChildIndex], elements[nextIndex]) {
                nextIndex = rightChildIndex
            }

            // 다음 위치가 현재 위치와 같으면 종료
            if nextIndex == currentIndex {
                break
            }

            // 현재 위치와 선택된 자식 위치를 스왑
            elements.swapAt(currentIndex, nextIndex)
            currentIndex = nextIndex
        }
    }
}

~~~

<br>
<br/>