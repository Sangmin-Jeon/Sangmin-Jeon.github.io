---
layout: post
title: BOJ_19598번_최소_회의실_개수
color: rgb(50, 170, 250)
feature-img: "assets/img/algoritms-img/boj_image.png"
thumbnail: "assets/img/algoritms-img/boj_image.png"
tags: [BOJ, 그리디, 우선순위 큐, 정렬]
---

# [BOJ] 19598번 최소 회의실 개수 (Swift)
자료구조 && 알고리즘 문제 풀이 및 설명

||| 
|--| -- |
| 제목 | [BOJ] 19598번 최소 회의실 개수 |
| 출처 | [19598번 최소 회의실 개수](https://www.acmicpc.net/problem/19598) |
| 문제 제공| Baekjoon |
| 정답 비율 | 43.291% |
| 난이도 | [Solved.ac] 골드5 |
| 분류 | 그리디, 우선순위 큐, 정렬 |

## 문제

서준이는 아빠로부터 N개의 회의를 모두 진행할 수 있는 최소 회의실 개수를 구하라는 미션을 받았다. 각 회의는 시작 시간과 끝나는 시간이 주어지고   한 회의실에서 동시에 두 개 이상의 회의가 진행될 수 없다. 단, 회의는 한번 시작되면 중간에 중단될 수 없으며 한 회의가 끝나는 것과 동시에 다음   회의가 시작될 수 있다. 회의의 시작 시간은 끝나는 시간보다 항상 작다. N이 너무 커서 괴로워 하는 우리 서준이를 도와주자.  

## 입력

첫째 줄에 배열의 크기 N(1 ≤ N ≤ 100,000)이 주어진다. 둘째 줄부터 N+1 줄까지 공백을 사이에 두고 회의의 시작시간과 끝나는 시간이 주어진다.시작 시간과 끝나는 시간은 231−1보다 작거나 같은 자연수 또는 0이다.  

#### 예제 입력 1

3  
0 40  
15 30  
5 10  

#### 예제 출력 1

2

2개 회의실로 3개 회의를 모두 진행할 수 있다. 예를 들어, 첫번째 회의실에서 첫번째 회의를 진행하고 두번째 회의실에서 두번째 회의와 세번째    회의를 진행하면 된다. 1개 회의실로 3개 회의를 진행할 수 없고 3개 이상의 회의실로 3개 회의를 모두 진행할 수 있지만 최소 회의실 개수를 구해야 하기 때문에 2가 정답이 된다.   

#### 예제 입력 2

2  
10 20  
5 10  

#### 예제 출력 2

1

## 출력

첫째 줄에 최소 회의실 개수를 출력한다.  

## 전체 코드

### [Swift]

```swift
import Foundation

let n = Int(readLine()!) ?? 0
var meetings = [(Int, Int)]()
for _ in 0 ..< n {
    if let input = readLine() {
        let _time = input.split(separator: " ").map({ Int($0)! })
        meetings.append((_time.first!, _time.last!))
    }
}

meetings.sort(by: {
    if $0.0 == $1.0 {
        return $0.1 < $1.1
    }
    return $0.0 < $1.0
})


var pQueue = PriorityQueue<Int>(order: <)
pQueue.enqueue(meetings.removeFirst().1)

for meet in meetings {
    if meet.0 >= pQueue.peek()! {
        let _ = pQueue.dequeue()
        pQueue.enqueue(meet.1)
        continue
    }
    pQueue.enqueue(meet.1)
}

print(pQueue.count)


```

#### 우선순위 큐 코드

```swift
// MARK: 우선순위 큐
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


``` 

<br>
<br/>
    
## 풀이 및 설명

> 우선순위 큐를 사용하여 N개의 회의를 모두 진행할 수 있는 최소 회의실의 개수를 구하는 문제입니다.     
>> 문제에서 입력 크기는<span style="font-size: 0.9em">$$N(1 ≤ N ≤ 100,000)$$</span>   
으로 주어지므로 시간 복잡도는<span style="font-size: 0.9em">$$O(n)$$</span> 혹은 <span style="font-size: 0.9em">$$O(nlog n)$$</span>   
안으로 해결 해야하고     
기존 배열로 정렬해가면서 순회하는 방식은 <span style="font-size: 0.9em">$$O(n^2)$$</span>로 비 효율적임으로 우선순위 큐를 사용하였습니다.  

>> <span style="color:green">*Swift에는 우선순위 큐가 제공되지 않아 우선순위 큐를 별도로 구현하여 사용하였습니다.  
추후 우선순위 큐에 대해서도 포스트 할 예정입니다.*</span>

문제의 입력 예시로   

| 회의 | 회의시간 | 
|--|--|
| 회의 1 | 0 ~ 40 |
| 회의 2 | 15 ~ 30 |
| 회의 3 | 5 ~ 10 |

의 회의 시간대가 주어집니다.     
먼저 회의시간대를 `시작 시간순`으로 정렬하고, 만약 시작시간이 같다면 `종료 시간 기준`으로 정렬합니다.  
그러면 아래 처럼 정렬 될 것입니다. 

| 회의 | 회의시간 | 
|--|--|
| 회의 1 | 0 ~ 40 |
| 회의 3 | 5 ~ 10 |
| 회의 2 | 15 ~ 30 |   

회의 시간대를 정렬한 배열(meetings)을 반복문으로 순회하면서 회의시간(meet)과 우선순위 큐(pQueue)를 사용하여 큐에서 가장 빠르게 사용가능한 회의실을 찾아 비교하여 회의실을 추가 할지, 기존 사용한 회의실에 시간을 교체할지를 결정합니다.   

우선순위 큐(pQueue)에는 회의시간의 종료시간을 *enqueue*합니다.  

1 ) 여기서 회의실 추가는 **enqueue**   
2 ) 기존 회의실에 시간을 교체하는 작업은 **dequeue**후 **enqueue**로 qQueue의 count를 증가시키기 않습니다.  

2번의 경우 아래 코드와 같이   
```swift
if meet.0 >= pQueue.peek()! {
    let _ = pQueue.dequeue()
    pQueue.enqueue(meet.1) 
...

```
회의 시작시간**meet.0**이 기존 회의실의 종료시간**pQueue.peek()!**보다 크거나 같으면   
사용중인 회의실의 시간을 교체합니다.  

위 입력예시를 참고하여 코드를 설명하자면   
```swift
1. 맨 처음 pQueue에 정렬된 meeting의 첫번째 index의 시간대의 종료시간을 추가합니다.   
(meeting배열은 queue 자료구조 입니다.)
meeting = [(0, 40), (5, 10). (15, 30)]
pQueue = [40] 
meeting = [(5, 10). (15, 30)]

2. 회의 시작시간과 회의실의 종료시간을 비교합니다.
meet.0 (5) 와 pQueue.peek() (40) 을 비교
아직 종료가 안되었기 때문에 enqueue(meet.1)로 회의실 추가 (이때 종료시간을 추가합니다)
pQueue = [10, 40] 

3. 위 2번을 반복합니다.
meet.0 (15) 와 pQueue.peek() (10) 을 비교
회의가 종료되었기 때문에 회의실에 시간을 교체해줍니다.
pQueue = [40]
pQueue = [30, 40]  
continue

4. meeting배열을 모두 순회후 pQueue에 추가되어 있는 데이터의 수를 확인합니다.
pQueue.count = 2

```

이렇게 우선순위 큐를 사용하여 회의실의 회의 종료시간을 정렬 한다음   
시작해야하는 회의 시작시간과 비교하여 모든 회의를 할 수 있게하는 최소개수의 회의실의 개수를 알 수 있습니다.   

<br>
<br/>