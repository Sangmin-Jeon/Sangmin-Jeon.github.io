---
layout: post
title: BOJ_9466번_텀_프로젝트
color: rgb(50, 170, 250)
feature-img: "assets/img/algoritms-img/boj_image.png"
thumbnail: "assets/img/algoritms-img/boj_image.png"
tags: [BOJ, 그래프 이론, 그래프 탐색, DFS]
---

# [BOJ] 9466번 텀 프로젝트 (Swift)
자료구조, 알고리즘 문제 풀이 및 설명

||| 
|--| -- |
| 제목 | [BOJ] 9466번 텀 프로젝트 |
| 출처 | [9466번 텀 프로젝트](https://www.acmicpc.net/problem/9466) |
| 문제 제공| Baekjoon |
| 정답 비율 | 24.077% |
| 난이도 | [Solved.ac] 골드3 |
| 분류 | 그래프 이론, 그래프 탐색, DFS |

## 문제

이번 가을학기에 '문제 해결' 강의를 신청한 학생들은 텀 프로젝트를 수행해야 한다. 프로젝트 팀원 수에는 제한이 없다. 심지어 모든    학생들이 동일한 팀의 팀원인 경우와 같이 한 팀만 있을 수도 있다. 프로젝트 팀을 구성하기 위해, 모든 학생들은 프로젝트를 함께하고   싶은 학생을 선택해야 한다. (단, 단 한 명만 선택할 수 있다.) 혼자 하고 싶어하는 학생은 자기 자신을 선택하는 것도 가능하다.  
  
학생들이(s1, s2, ..., sr)이라 할 때, r=1이고 s1이 s1을 선택하는 경우나, s1이 s2를 선택하고, s2가 s3를 선택하고,...,   sr-1이 sr을 선택하고, sr이 s1을 선택하는 경우에만 한 팀이 될 수 있다.  

예를 들어, 한 반에 7명의 학생이 있다고 하자. 학생들을 1번부터 7번으로 표현할 때, 선택의 결과는 다음과 같다.   

|1|2|3|4|5|6|7|
|---|---|---|---|---|---|---|
|3|1|3|7|3|4|6|

위의 결과를 통해 (3)과 (4, 7, 6)이 팀을 이룰 수 있다. 1, 2, 5는 어느 팀에도 속하지 않는다.  

주어진 선택의 결과를 보고 어느 프로젝트 팀에도 속하지 않는 학생들의 수를 계산하는 프로그램을 작성하라.  

## 입력

첫째 줄에 테스트 케이스의 개수 T가 주어진다. 각 테스트 케이스의 첫 줄에는 학생의 수가 정수 n (2 ≤ n ≤ 100,000)으로 주어진다.   각 테스트 케이스의 둘째 줄에는 선택된 학생들의 번호가 주어진다. (모든 학생들은 1부터 n까지 번호가 부여된다.)  

### 예제 입력 1 

2  
7  
3 1 3 7 3 4 6  
8  
1 2 3 4 5 6 7 8  
### 예제 출력 1 

3  
0

## 출력

각 테스트 케이스마다 한 줄에 출력하고, 각 줄에는 프로젝트 팀에 속하지 못한 학생들의 수를 나타내면 된다.

## 전체 코드

### [Swift]

```swift
import Foundation

for _ in 0 ..< Int(readLine()!)! {
    let students = Int(readLine()!) ?? 0
    let stuList = readLine()!.split(separator: " ").map{ Int(String($0)) ?? 0 }
    
    var visited = Array(repeating: false, count: students)

    let answer = count_outSider(students: students, stuList: stuList, visited: &visited)
    print(answer)

}

func count_outSider(students: Int, stuList: [Int], visited: inout [Bool]) -> Int {
    var answer = 0
    var team = [Int]()
    for stu in 1 ... students {
        if !visited[stu - 1] {
            team = []
            answer += dfs(stu: stu, stuList: stuList, team: &team, visited: &visited)
        }
    }

    return (students - answer)
}


func dfs(stu: Int, stuList: [Int], team: inout [Int], visited: inout [Bool]) -> Int {
    let stuIndex = stu - 1
    team.append(stu)
    visited[stuIndex] = true

    let chooseStu = stuList[stuIndex]

    if visited[chooseStu - 1] {
        if team.contains(chooseStu) {
            if let startIndex = team.firstIndex(of: chooseStu) {
                let sublist = Array(team[startIndex...])
                return sublist.count
            }
        }
        return 0
    }

    return dfs(stu: chooseStu, stuList: stuList, team: &team, visited: &visited)
}

```

<br>
<br/>

## 풀이 및 설명

> 문제 자체는 크게 어렵지 않았지만 제한 사항 때문에 시간 초과와 메모리 초과로 많이 애먹었습니다...  
> 저는 그래프 DFS로 탐색하여 문제를 해결 하였습니다.  
>> 문제에서 입력 크기는<span style="font-size: 0.9em">$$n(2 ≤ n ≤ 100,000)$$</span>   
으로 주어지므로 시간 복잡도는<span style="font-size: 0.9em">$$O(n)$$</span> 혹은 <span style="font-size: 0.9em">$$O(nlog n)$$</span>   
안으로 해결 해야 합니다.  

* 입력 예시 :  
1  
7  
2 3 4 5 2 4 7  

위 입력 예시를 그림으로 표현하면 아래 처럼 됩니다.

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/2852ce01-e191-4876-b353-668fff1f8af0)

학생 1번 부터 7번까지 순서대로 DFS하여 방문한 곳을 기억하면서 순회를 하다가 방문한 곳을 다시 만났다면  
이때가 한 개의 팀이 만들어지는 조건입니다. 

예시의 그림으로 표현해 보자면

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/2ac113ac-fa84-4b24-9a70-6e513e0e0fc8)


1번 학생 부터 5번 학생까지 순회 하면 2번 학생 부터 5번 학생까지 팀이 만들어 지는것을 알 수 있습니다.  
(2번 학생 ~ 5번 학생까지 이어지다가 5번 학생이 2번 학생을 지목했기 때문에)  

이때 앞에 1번 학생의 경우 팀이 될 수 없어서 제외 됩니다.
```swift
if team.contains(chooseStu) {
    if let startIndex = team.firstIndex(of: chooseStu) {
        let sublist = Array(team[startIndex...]) // [2...5]까지 담고 1 제외
        return sublist.count // 팀이 된 학생들의 수를 리턴
    }
}

- 시간 복잡도
contains()메소드로 배열의 요소(선택한 학생)를 찾는 작업: O(n)
firstIndex(of:)메소드로 해당 요소의 첫번째 인덱스 찾기: O(n)
이므로 총 시간복잡도는 O(n)으로 볼 수 있다.

```

한번 이라도 제외 되었던 학생은 뒤에 다른 학생이 제외된 학생을 지목 하여도 팀으로 만들어 질 수가없는데   
만약, 제외 되었던 학생A를 다른 학생B가 지목 하였다면  
A학생은 앞전에 이미 다른 학생C를 지목했다가 팀이 되지 못하였기 때문에 B학생이 A학생을 지목 했다고 하더라도  
A학생을 통해 다시 B학생과 만나는 루트가 만들어질 수 없기 때문입니다.  

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/05e8c719-a295-4697-a001-80e5c5c78aa6)

쉽게 이해하기 위해 그림으로 표현해 보았습니다.  
1번 학생을 지목 했더라도 이미 2 ~ 5번 학생이 팀으로 연결되었기 때문에 6번 학생과는  
만날 수 가없다는걸 알 수 있습니다.  

![image](https://github.com/Sangmin-Jeon/Sangmin-Jeon.github.io/assets/59474775/334874a3-8df2-4227-a947-be4a7768afcb)

마지막으로 7번 학생의 경우 자신을 지목 했기에 혼자 팀이 될 수 있습니다.

결과 적으로 팀이 되지 못한 학생들은 1, 6번 학생이고  
팀이 된 학생들은 2, 3, 4, 5, 7번 학생들이 됩니다.

```swift
func count_outSider(students: Int, stuList: [Int], visited: inout [Bool]) -> Int {
    var answer = 0
    var team = [Int]()
    for stu in 1 ... students {
        if !visited[stu - 1] {
            team = []
            // 팀이 된 학생들의 수를 리턴받아 answer변수에 더해 줌
            answer += dfs(stu: stu, stuList: stuList, team: &team, visited: &visited)
        }
    }

    return (students - answer) // 총 학생수 - 팀이 된 학생 수
}

```
제가 작성한 dfs()함수는 팀이 된 학생들의 수를 리턴함으로 처음 입력받은 모든 학생 수에서  
리턴 된 학생 들의 수를 빼서 어느 팀에도 속하지 않는 학생들의 수를 구할 수 있습니다.  

따라서   
입력 :  
1  
7  
2 3 4 5 2 4 7   
의 답은 2가 됩니다.

<br>
<br/>

* 참고로 swift로 입력 받는 부분이 궁금 하신 분들은 아래 코드를 참고 해주세요.   
```swift
import Foundation

for _ in 0 ..< Int(readLine()!)! { // T (Int)
    let _ = Int(readLine()!) ?? 0 // n (Int)
    let _ = readLine()!.split(separator: " ").map{ Int(String($0)) ?? 0 } // 선택된 학생들의 번호 ([Int])
...    
    // 입력 받은 데이터로 여기에 코드를 작성하세요.

...

```
<br>
<br/>