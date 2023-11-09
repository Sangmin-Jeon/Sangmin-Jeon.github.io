---
layout: post
title: "[BOJ 5547번] 일루미네이션"
color: rgb(50, 170, 250)
feature-img: "assets/img/algoritms-img/boj_image.png"
thumbnail: "assets/img/algoritms-img/boj_image.png"
tags: [BOJ, 그래프 이론, 그래프 탐색, BFS, DFS]
---

# [BOJ] 5547번 일루미네이션 (Swift)
자료구조, 알고리즘 문제 풀이 및 설명

||| 
|--| -- |
| 제목 | [BOJ] 5547번 일루미네이션 |
| 출처 | [5547번 일루미네이션](https://www.acmicpc.net/problem/5547) |
| 문제 제공| Baekjoon |
| 정답 비율 | 60.674% |
| 난이도 | [Solved.ac] 골드4 |
| 분류 | 그래프 이론, 그래프 탐색, BFS, DFS |

## 문제
부유한 집안의 상속자 상근이네 집은 그림과 같이 1미터의 정육각형이 붙어있는 상태이다. 크리스마스가 다가오기 때문에, 여자친구에게  
잘 보이기 위해 상근이는 건물의 벽면을 조명으로 장식하려고 한다. 외부에 보이지 않는 부분에 조명을 장식하는 것은 낭비라고 생각했기 때문에,   
밖에서 보이는 부분만 장식하려고 한다.  

![building](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/30054ecf-76af-4675-92ea-eaf0ab786d93)

위의 그림은 상공에서 본 상근이네 집의 건물 배치이다. 정육각형 안의 숫자는 좌표를 나타낸다. 여기서 회색 정육각형은 건물의 위치이고,  
흰색은 건물이 없는 곳이다. 위에서 붉은 색 선으로 표시된 부분이 밖에서 보이는 벽이고, 이 벽에 조명을 장식할 것이다.   
벽의 총 길이는 64미터이다.  

상근이네 집의 건물 위치 지도가 주어졌을 때, 조명을 장식할 벽면의 길이의 합을 구하는 프로그램을 작성하시오.  
지도의 바깥은 자유롭게 왕래 할 수 있는 곳이고, 인접한 건물 사이는 통과할 수 없다.  

## 입력
첫째 줄에 두 개의 정수 W와 H가 주어진다. (1 ≤ W, H ≤ 100) 다음 H줄에는 상근이네 집의 건물 배치가 주어진다. i+1줄에는 W개의 정수가   공백으로 구분되어 있다. j번째 (1 ≤ j ≤ w) 정수의 좌표는 (j, i)이며, 건물이 있을 때는 1이고, 없을 때는 0이다. 주어지는 입력에는 건물이   적어도 하나 있다.  

지도는 다음과 같은 규칙에 의해 만들어졌다.  

지도의 가장 왼쪽 위에 있는 정육각형의 좌표는 (1,1)이다.  
(x,y)의 오른족에 있는 정육각형의 좌표는 (x+1,y)이다.  
y가 홀수 일 때, 아래쪽에 있는 정육각형의 좌표는 (x,y+1)이다.  
y가 짝수 일 때, 오른쪽 아래에 있는 정육각형의 좌표는 (x,y+1)이다.  

### 예제 입력 1
8 4  
0 1 0 1 0 1 1 1  
0 1 1 0 0 1 0 0  
1 0 1 0 1 1 1 1  
0 1 1 0 1 0 1 0  

### 예제 출력 1
64  

### 예제 입력 2
8 5  
0 1 1 1 0 1 1 1  
0 1 0 0 1 1 0 0  
1 0 0 1 1 1 1 1  
0 1 0 1 1 0 1 0  
0 1 1 0 1 1 0 0    
 
### 예제 출력 2
56  

## 출력
조명을 장식하는 벽면의 길이의 합을 출력한다.

## 전체 코드

### [Swift]

```swift
import Foundation

func setGraph(r: Int, c: Int) -> [[Int]] {
    var graph = Array(repeating: Array(repeating: 0, count: r + 2), count: c + 2)
    for i in 1 ... c {
        let row = readLine()!.split(separator: " ").map{Int(String($0))!}
        graph[i][1 ... r] = row[0 ..< r]
    }

    return graph

}

func graphBFS(graph: [[Int]], visited: inout [[Bool]]) -> Int {
    var wallCnt = 0
    let dr = [-1, -1, 0, 1, 1, 0]
    let even_dc = [-1, 0, 1, 0, -1, -1]
    let odd_dc = [0, 1, 1, 1, 0, -1]
    var queue = [(r: Int, c: Int)]()

    queue.append((r: 0, c: 0))
    visited[0][0] = true

    while !queue.isEmpty {
        let visit = queue.removeFirst()
        let (_r, _c) = visit
        
        for i in 0 ..< 6 {
            let _dr = _r + dr[i]
            var _dc = 0
            if _r % 2 == 0 {
                _dc = _c + even_dc[i] 
            }
            else {
                _dc = _c + odd_dc[i]
            }

            if _dr >= 0, _dc >= 0, (col + 2) > _dr, (row + 2) > _dc, !visited[_dr][_dc] {
                if graph[_dr][_dc] == 0 {
                    queue.append((r: _dr, c: _dc))
                    visited[_dr][_dc] = true
                }
                else {
                    wallCnt += 1
                }
            }
        }
    }

    return wallCnt

}

let input = readLine()!.split(separator: " ").map({ Int($0)! })
let row = input[0]
let col = input[1]

var visited = Array(repeating: Array(repeating: false, count: row + 2), count: col + 2)
let answer = graphBFS(graph: setGraph(r: row, c: col), visited: &visited)
print(answer)

```

<br>
<br/>

## 풀이 및 설명

> 기본적인 Graph 탐색 문제에서 조금 변형된 문제 입니다.  
> 저는 너비 우선 탐색(BFS)를 사용하여 문제를 해결했습니다.
>> 이 문제의 경우 입력 범위가 <span style="font-size: 0.9em">$$(1 ≤ W, H ≤ 100)$$</span>로 최대 10000개의 입력이 들어오기 때문에     
시간복잡도 <span style="font-size: 0.9em">$$O(n^2)$$</span>이내로만 해결하면 됩니다.

이 문제에서 핵심은 그래프의 열 개수가 홀수 인지 짝수인지를 구분하는 부분과  
탐색 시 1(집)을 기준으로 queue를 돌리지 않고 처음에 그래프를 0영역으로 감싼 후 0을 기준으로   
queue를 돌려서 1(집)을 만나면 벽(wallCnt)를 증가 시키는 것 입니다.  
만약 1을 기준으로 할 경우 그래프에서 0부분이 (1)집 내부 영역에 포함되는지 안되는지를 판별해야 하기 때문에   
구현이 복잡해 질 수 있습니다.

BFS를 완료 후 wallCnt의 값이 문제의 최종 답이 됩니다.

### 1 ) 그래프의 열 개수가 홀수 인지 짝수인지 구분  
문제 설명을 보면
```
y가 홀수 일 때, 아래쪽에 있는 정육각형의 좌표는 (x,y+1)이다.  
y가 짝수 일 때, 오른쪽 아래에 있는 정육각형의 좌표는 (x,y+1)이다.  
```
라고 설명 되어있습니다.  

쉽게 보기 위해 y가 짝수일때의 경우와 홀수일때의 경우를 그림으로 표시해 보았습니다.

- 짝수  
![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/7fdafc28-e62c-479f-ac9f-aae03f9fb215)  

- 홀수  
![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/e40da149-13cf-45ff-8620-69ce00c03f04)  


graph는 2차원 배열이고, 위 그림에서 (r, c)를 기준으로 r은 동일하지만 c(문제 설명에서의 y)가 짝수인지 홀수인지에 따라  
주변 요소의 index가 다른것을 확인할 수 있습니다.  
따라서 BFS 시 아래 코드와 같이 구분하여 구현 하였습니다.  

```swift
let dr = [-1, -1, 0, 1, 1, 0]
let even_dc = [-1, 0, 1, 0, -1, -1] // 짝수
let odd_dc = [0, 1, 1, 1, 0, -1] // 홀수

...

let _dr = _r + dr[i]
var _dc = 0
if _r % 2 == 0 {
    _dc = _c + even_dc[i] 
}
else {
    _dc = _c + odd_dc[i]
}

...

``` 

<br>
<br/>

### 2 ) 그래프의 끝 영역들을 0으로 감싸 주기  

말 그대로 입력 예시를 예로 들어 표현하자면  
0 1 0 1 0 1 1 1  
0 1 1 0 0 1 0 0  
1 0 1 0 1 1 1 1  
0 1 1 0 1 0 1 0   
를  

0 0 0 0 0 0 0 0 0 0  
0 0 1 0 1 0 1 1 1 0   
0 0 1 1 0 0 1 0 0 0   
0 1 0 1 0 1 1 1 1 0   
0 0 1 1 0 1 0 1 0 0  
0 0 0 0 0 0 0 0 0 0  
와 같이 0으로 둘러서 감싸주는 것을 말합니다.  

이 부분도 그림으로 표현해 보자면 아래 와 같습니다.  

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/0556dd5d-aa2c-4012-b2fc-379a958fcb2f)


이때 행과 열에 개수가 2개씩 늘어나게 됨으로 그래프의 범위가 8, 4이면  
10, 6으로 증가하는것을 BFS 시 꼭 반영해 주어야 합니다.  

*시간복잡도를 조금이라도 줄이겠다고 BFS 시 조건문에 count함수를 사용하지 않고 입력 값에 + 2를 하여 사용 하려다 보니  
범위를 헷갈려 그래프를 제대로 탐색하지 못해 문제 푸는데 시간을 소모 하게 되었습니다..*  

위 코드에서 그래프 생성 부분을 살펴보겠습니다.

```swift
1. 그래프 생성
func setGraph(r: Int, c: Int) -> [[Int]] {
    var graph = Array(repeating: Array(repeating: 0, count: r + 2), count: c + 2)
...

}

setGraph함수 내부에서 입력 받은 행과 열 정보를 사용해 그래프를 만들었습니다.
이 때 행과 열에 각각 + 2씩 해주어 모든 배열의 요소를 0으로 채워주었습니다.

2. 그래프 기존 영역 채우기
...

for i in 1 ... c {
    let row = readLine()!.split(separator: " ").map{Int(String($0))!}
    graph[i][1 ... r] = row[0 ..< r]
}

... 

그 후 입력 받은 그래프 정보를 생성한 그래프에 반영하여 최종적으로 BFS를 위한 그래프를 만들었습니다. 
위 이미지의 검은색 영역입니다.

```  

<br>
<br/>

### 3 ) 그래프 탐색 (BFS)  

코드에서 BFS 부분을 요약 설명한 그림을 보겠습니다.

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/359c9db0-f616-4e13-b96e-6f5af18a9494)

0으로 그래프를 감싸 주었기 때문에 BFS시작은 항상 (0, 0)에서 시작 할 수 있게 됩니다.  
위 코드와 같이 보자면 아래설명과 같습니다.

```swift 
1. 그래프 탐색 방식은 기존 BFS와 같고 0을 기준으로 탐색하기에 그래프의 요소가 0인 경우 Queue에 추가되게 됩니다. 
재방문을 하지 않게 하기 위해 한 번 방문한 영역은 기억하면서(visited) 그래프를 탐색하고  
(graph[_dr][_dc] == 0)이 아닌경우 wallCnt를 1증가 시켜줍니다.
... 

// Graph BFS

if _dr >= 0, _dc >= 0, (col + 2) > _dr, (row + 2) > _dc, !visited[_dr][_dc] {
    if graph[_dr][_dc] == 0 {
        queue.append((r: _dr, c: _dc))
        visited[_dr][_dc] = true
    }
    else {
        wallCnt += 1
    }
}

... 

```

결과 적으로 한번의 그래프 탐색으로 밖에서 보이는 모든 건물 벽면의 개수를 구할 수 있습니다.  

<br>
<br/>