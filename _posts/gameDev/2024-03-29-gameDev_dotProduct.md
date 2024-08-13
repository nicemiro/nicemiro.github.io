---
title: 벡터의 내적
titleEn: Dot Product
author: BabyK
date: 2024-04-01
category: GameDev
layout: post
published: false
---
<br>

벡터의 내적은 게임 코드내에서 굉장히 많이 사용된다.  
빛이 투영 (Projection) 하는 부분과 그림자가 지는 부분을 구분한다던가,   
게임캐릭터의 시야각과 그 사이에 위치한 물체의 포착여부를 판별해 주거나,  
정방향의 공격인지 뒤에서 공격하는 백어택인지 하는 방향성을 판별하는데에도 쓰인다. 
또한 작용된 힘의 포션중 물체가 이동하는 방향으로 작용된 부분을 표현 하는 기본적인 용도에도 쓸 수 있다.  

내적은 결국 두 방향으로 작용된 벡터 방향의 유사성을 판단하는 로직인데  
Adjacent (밑변) 에 vectorB의 값을 곱해주는 이유는  
두 벡터가 방향성이 유사할 수록 큰 값의 내적을 얻기 위함인데 그 둘의 진행방향이 평행할때 최대치,  
수직으로 직교하여 움직일 때 값은 0이 된다.  


벡터의 실수배

벡터의 평행조건 - 두벡터중 한 벡터가 다른 한 벡터의 실수배로 표현될때 두 백터는 평행하다고 볼 수 있다.