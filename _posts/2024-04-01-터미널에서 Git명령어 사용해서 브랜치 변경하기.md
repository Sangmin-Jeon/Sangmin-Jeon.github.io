---
layout: post
title: "[etc, issue] Xcode에서 Branch 변경 안 되는 문제해결"
color: rgb(50, 170, 250)
tags: [git, etc]
---

안녕하세요 👋  
이번에 프로젝트를 진행 중 Xcode에서 사용하려는 branch가 변경되지 않는 이슈가 있어서 해결 후   
같은 문제를 겪으시는 분들에게 도움이 되고자 이렇게 포스팅하게 되었습니다.

Xcode에서 별짓을 다 해도 해결되지 않아 결국 터미널에서 직접 **명령어**를 입력하여  
branch를 변경하니 바로 해결이 되었네요...🤔

<span style="color:green">우선 혹시 모르니 **Xcode 설정 - account**에서 계정과 잘 연동되어 있는지 확인 후 진행해 주세요.</span>   

#### - branch 변경하기 (check out)
변경하기 전 **현재 branch** 위치를 확인하시고 **변경하실 branch**가 **로컬저장소**에 있는지 확인해주세요.
~~~bash
git checkout <your branch name> // <이 부분에 변경하실 brach의 이름을 입력하세요> 
~~~

명령어 입력 후 터미널에서 아래 메세지가 나오면 **branch**가 변경된 것 입니다.
~~~bash
branch <your branch name> set up to track 'origin/your branch name'.
Switched to a new branch 'your branch name'
~~~

<br>

<span style="color:green">이렇게만 알아보면 아쉬우니 다른 명령어들도 같이 보겠습니다.  
아래 명령어들은 필요시 참고하여 적절히 사용하세요.</span>

<br>

#### - 새로운 branch를 생성하고 동시에 그 branch로 이동
**새로운 branch**를 생성하고 해당 **branch**로 이동합니다.
~~~bash
git checkout -b <new branch name>
~~~

#### - 원격(origin) branch 가져와 변경하기  
만약 **로컬저장소**에 branch가 존재하지 않고 **원격저장소**에 있는 branch를 가져와 변경해야 할 경우에는 아래와 같이 합니다.
~~~bash
git fetch origin <your branch name> // origin branch 가져오기
git checkout <your branch name> // branch 변경
~~~

<br>