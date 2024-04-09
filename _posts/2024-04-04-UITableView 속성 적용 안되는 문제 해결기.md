---
layout: post
title: "[iOS, UIKit] UITableView 속성 적용 안되는 문제 해결기"
color: rgb(50, 170, 250)
tags: [iOS, UIKit]
---

안녕하세요 👋   
오늘은 **UIView**의 UITableViewCell SeparatorStyle에 대한 포스팅 입니다.  
최근에 겪은 TablewView의 SeparatorStyle 설정 코드가 제대로 적용되지 않는 문제가 있었어서 해결 후 그 과정에 대해 공유하려고 합니다.  

원인을 파악하고 해결하는 과정에서 이것저것 시도하다보니   
View의 생명주기와 객체 인스턴스의 생명주기도 다시 공부하게 되는 계기가 되었네요... 😅

<br>

## 문제 상황  
---
MVVM 패턴을 적용한 UIKit 프로젝트에서 리팩토링 과정에서 문제가 발생했습니다.   
리팩토링 전에는 ViewController에 모든 UI 관련 코드가 작성되어 있었지만 **MVVM 패턴**의 원칙에 따라 UI 코드를 별도의 **UIView** 객체로 분리했습니다.   
ViewController에서 viewModel의 데이터를 받아와 View에 바인딩 하는 방식으로 기존 코드에서 ViewController의 역할을 줄이고자 하였습니다.    
하지만 이 과정에서 잘 적용되던 TableView의 SeparatorStyle 설정 코드가 제대로 적용되지 않는 문제가 발생했습니다.  

<br>

## 문제 파악 및 분석
---
우선 SeparatorStyle에 대하여 알아보겠습니다.  
공식 문서를 참고하자면  

![스크린샷 2024-04-04 오후 11 38 15](https://github.com/Sangmin-Jeon/Weather_APP/assets/59474775/4f5ccea6-2703-4e8e-bb30-557d9f0df413)

라고 되어있습니다.  
별거 없이 UITableView의 속성으로 **singleLine**과 **none**으로만 구분되며 여기서 저는 none으로 하여 Cell 구분선을 없애주려 하였습니다.     

#### - 기존 코드 
기존 코드에서는 TableView 초기 설정 코드를 View의 init 생성자에 작성한 후, ViewController의 전역으로 view인스턴스를 생성하였습니다.      
View객체의 초기화 과정에서 **tableView**의 초기 설정 코드를 **생성자(init)**에 작성하고     
ViewController의 전역 변수로 **View 인스턴스**를 생성하였습니다.  
이로 인해 **viewDidLoad**가 호출되기 전에 **tableView의 속성**에 접근하여 제대로 설정되지 않는 문제가 발생 했던겁니다.   

<script src="https://gist.github.com/Sangmin-Jeon/b110bb7c2963f34048fcbb79da7558cb.js"></script>

<br>

## 문제 해결
---
View 인스턴스의 생성 시점을 조정하였습니다.  
ViewController의 **viewDidLoad** 메서드에서 View 인스턴스를 생성하고, 이를 View 계층에 추가 하였습니다.    
이렇게 하면 viewDidLoad가 호출되는 시점에 올바른 초기화가 이루어지며, 안전하게 tableView의 속성에 접근할 수 있습니다.    
즉, View 계층 구조가 로드되고 설정된 후 tableView속성에 접근하는 방식으로 해결하였습니다.    

#### - 수정한 코드
<script src="https://gist.github.com/Sangmin-Jeon/2ae9ae35d0831eb8b7e8d69802265a58.js"></script>

추가적으로 코드의 깔끔함과 유지 보수성 강화하기 위해 기존의 init에 있던 tableView 초기 설정 코드를 View 내부에 있는 tableView 선언 시   
적용하도록 수정하여 코드를 더 깔끔하고 유지 보수하기 좋게 수정하였습니다.  

<br>

## 배운점
---
View객체는 viewDidLoad가 호출되는 시점에 초기화가 이루어져야 안정적으로 동작할 수 있다는것을 알게되었습니다.  
그리고 이번에 tableView 속성들과 관련 코드들을 한곳에서 관리 함으로서 추후 유지보수 측면에서도  
View의 추가나 초기화 코드를 보기에도 쉽고 관리하기도 편하다는것을 느꼈습니다 🙂

앞으로 UI관련 코드 작성 시 이런식으로 관리 해봐야겠습니다.  



