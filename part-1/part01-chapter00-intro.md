---
description: '20210119 : 초안 작성'
---

# PCL 기반 로봇 비젼

본 장에서는 산업용 로봇의 '빈피킹\(bin-picking\)'이라는 기술을 기반으로 3D비전처리의 전반적 흐름에 대하여 다루어 보도록 하겠습니다. 빈 피킹\(Bin picking, Random Bin picking\)은 로봇이 무작위로 놓인 임의의 물체들을 특징에 따라 구분해 인식하고 각각의 용도나 목적에 맞게 다룰 수 있도록 하는 기술을 의미 합니다. 많은 기술 중에서 3D비전처리 실습으로 빈피킹을 선별한 이유는 제조업 노동자의 38%가 빈 피킹 관련 업무를 진행하고 있으며, 아직도 약 50만 개의 일자리는 필요할 정도로 자동화 업계에서는 중요한 기술이기 때문입니다.

사람에게는 간단한 박스에서 물건을 꺼내 정해진 선반에 놓는 빈피킹 작업을 로봇에 적용하기 위해서는 물체 인식\(object perception\), 운동 계획\(motion planning\), 파지\(grasp planning\), 작업 계획\(task planning\)기술들을 통합적으로 적용하여야 합니다. 이러한 기술의 중요성과 복합성으로 인해 세계 최대 물류회사인 아마존에서는 2015~2017년동안 아마존 피킹 챌린지\(Amazon Picking Challenge\)를 개최 하여 자사의 창고 관리의 효율을 높이는 기술을 발굴하고 있습니다. 그림 \[2.1\]은 챌린지에서 사용된 선반 및 물건들입니.

![&#xADF8;&#xB9BC; 2.1 &#xBE48;&#xD53C;&#xD0B9; &#xCC4C;&#xB9B0;&#xC9C0; &#xB85C;&#xBD07;&#xACFC; &#xBB3C;&#xAC74;](https://user-images.githubusercontent.com/17797922/104982468-17e89700-5a4e-11eb-99e4-f10ffd403e73.png)

빈피킹은 크게 두가지 작업으로 구분 할수 있습니다. 

* 바구니에 담긴 물건들을 각각의 분류 기준에 맞춰 옮겨 담는 `짐을 싣는 작업`\(stow task\) 
* 보관함에서 지정된 물건을 집어서 정해진 박스에 담는 `뽑기 작업`\(pick task\)

두 작업 모두 시작은 바구니나 보관함의 있는 물건\(그림B\)을 파악하는 \[물체 인식\] 기술 부터 적용 됩니다. 이 단계에서 3D 비젼을 통해 물체를 인식할수 있습니다. 최근에는 새로운 물건을 포함시켜 주어진 시간안에 새 물건을 학습하여 인식하는 머신러닝 기술의 적용도 같이 이루어 지고 있습니다 .

> APC 2017우승팀인 ARCV은 RefineNet라는 딥러닝을 이용하여 좋은 성적을 거두었습니다.

본문에서는 3D비젼을 이용한 물체 탐지와 머신러닝과 딥러닝을 이용한 물체 인식 기술에 대하여 살펴 보겠습니다. 

실습을 위해서는 로봇이 필요 하지만 그만한 공간과 비용을 마려하기는 쉽지 않습니다. 따라서, 1장에서 살펴본 ROS의 로봇 시뮬레이터상에 가상의 로봇을 만들어 진행 하도록 하겠습니다. 가상 로봇은 온라인 교육 사이트인 UDACITY의 로봇 강의에서 제공하는 실습 자료를 기반으로 하였습니다.

![&#xADF8;&#xB9BC; 2.2 &#xBE48;&#xD53C;&#xD0B9;&#xC744; &#xC704;&#xD55C; &#xAC00;&#xC0C1;&#xB85C;&#xBD07;&#xACFC; &#xAC00;&#xC0C1; &#xBB3C;&#xAC74;](https://user-images.githubusercontent.com/17797922/104982652-757ce380-5a4e-11eb-93e6-794c942c1efa.png)

전체 동작 과정은 아래와 같습니다. 

* 노이즈 제거 : 잡음 제거 
* Sampling : 계산 부하를 줄이기 위해 샘플링을 합니다. 
* Roi 설정 : 대상 물체가 있는 공간\(관심영역, RoI, Region of Interesting\)을 지정 합니다. 
* Clustering : 점군을 물체별로 구분 합니다. 
* Clasification : 물체를 분류 합니다. 

위 파이프라인은 대부분의 환경에서도 비슷한 흐름으로 진행 됩니다. 각 환경에 따라 사용되는 알고리즘과 방법이 다를뿐 전체 흐름은 같습니다. 실습에서 사용되는 알고리즘은 아래 표와 같습니다.

![3D &#xC601;&#xC0C1;&#xCC98;&#xB9AC; &#xC5F0;&#xAD6C; &#xBD84;&#xC57C;&#xBCC4; &#xC785;&#xB825;&#xACFC; &#xCD9C;&#xB825; &#xACB0;&#xACFC;&#xBB3C; &#xC815;&#xB9AC; ](https://user-images.githubusercontent.com/17797922/104982935-0b187300-5a4f-11eb-80c8-471c0237cec6.png)

