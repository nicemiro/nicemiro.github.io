---
title: 블록체인의 기초(1)
titleEn:
author: BabyK
date: 2025-09-21
category: BlockChain
layout: post
tags: [BlockChain]
published: true
---

### 페르마의 소정리
 
<p>암호학의 기초수학원리로 사용되며 소수 p에서의 유한체 (F<sub>p</sub>) 안에서 성립하는 성질.  </p>

#### 정의
* p : Prime Number (소수) => 1보다 큰 자연수중 1과 자신만을 약수로 갖는 수  
* n 과 P는 서로소 관계 (둘 사이에 1이외의 공약수가 없는 상태)

<div class="formula_01" style="text-align: center; font-size: large; margin-top: 20px;">
<b >n<sup>(p-1)</sup> % p = 1 </b>
</div>
<br>

#### 유한체의 곱셈
정수 0 ~ p 까지의 원소로 구성되는 유한체(Field) 를 아래와 같이 표현하고 유한체에서 곱셈군의 성질을 확인해 보자.
<div class="formula_01" style="text-align: center; font-size: large; margin-top: 20px;">
<p>F<sub>p</sub> = {0, 1, 2, 3, ... p-1} </p>
<p>ex) F<sub>9</sub> = {0, 1, 2, 3, 4, 5, 6, 7, 8}</p>
</div>

소수19 유한체의 서로소집합 {1, 3, 7, 13, 18} 을 소수 19의 유한체 원소들과 (0~18) 각각 순서대로 곱한뒤 19로 나눈 나머지 (Modulo)를 표현하면

<div class="formula_01" style="text-align: center; font-size: large; margin-top: 20px;">
<p>F<sub>19</sub> 에서  => {k ⋅ <sub>f</sub>0, K ⋅ <sub>f</sub>1, K ⋅ <sub>f</sub>2, K ⋅ <sub>f</sub>3, ... K ⋅ <sub>f</sub>18}</p>
</div>

```python
prime = 19 # 소수 19
for k in [1, 3, 7, 13, 18]: # 소수 19의 유한체
    print(sorted([k*i % prime for i in range(prime)]))

# 결과
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18]
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18]
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18]
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18]
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18]

```
결과는 19의 유한체 (0~19) 와 동일한 집합이 된다.  
모듈로연산(%) 으로 인해 소수 P의 유한체 F<sub>p</sub> 안에서 0과 서로소 원소 k를 곱한 값은 항상 F<sub>p</sub> 안에 머무른다는 것을 확인했다.

<div class="ex_01" style="text-align: left; font-size: medium; color : #0300beff; margin-top: 20px;">
  <p>ex) F<sub>19</sub> 에서</p>
  <ul>
    <li><p>8 + 9 = (8 + 9) % 19 = 15 (15는 소수 P의 유한체 원소)</p></li>
    <li><p>14 + 9 = (14 + 9) % 19 = 5</p></li>
    <li><p>-7 = (-7) % 19 = 12 ( 유한체에서의 뺄셈의 경우 덧셈계산 )</p> </li>
    <li><p>9 + 10 = 19 % 19 = 0 ( 여기서 10 = -9 로 볼수 있다.)</p> </li>
  </ul>
</div>
<br>

#### 증명
<p>이제 유한체 F<sub>19</sub> 에 페르마의 소정리식을 적용해 확인해보자.</p>  

```python
for i in [7, 11, 17, 31]: # 소수 p의 집합
    print([ (j**(i-1)) % i for j in range (1, i)])  # 소수 P 의 

# 결과
# [1, 1, 1, 1, 1, 1]
# [1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
# [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
# [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1]
```
<div class="formula_01" style="text-align: left; font-size: medium; margin-top: 20px;">
  <p><b >n<sup>(p-1)</sup> % p = 1 </b> &nbsp;  이용해서 정리하면 </p>
</div>

<div class="formula_01" style="text-align: left; font-size: medium; margin-top: 20px;">
    <ul>
      <li><p>F<sub>7</sub> 에서 0과 7자신을 제외하고 n = 1~6 까지 <b>n<sup>(7-1)</sup> % 7 = 1 </b> (모든 결과가 1)</p></li>
      <li><p>F<sub>11</sub>&nbsp; n=1~10 &nbsp;&nbsp; n<sup>(11-1)</sup> % 11 = 1 </p></li>
      <li><p>F<sub>17</sub>&nbsp; n=1~16 &nbsp;&nbsp; n<sup>(17-1)</sup> % 17 = 1</p></li>
      <li><p>F<sub>31</sub>&nbsp; n=1~30 &nbsp;&nbsp; n<sup>(31-1)</sup> % 31 = 1</p></li>
    </ul>
</div>

0과 소수p자신을 제외한 유한체 F<sub>p</sub> = {1,2,...,p-1}의 모든 원소 n에대해  
<div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">
<b> n<sup>p−1</sup> ≡ 1 (mod p) </b>
</div>
<br>
모든 결과값은 1이된다.   
0은 어떤 수를 곱해도 1이 나오지 않기 때문에(역원이 없다) 제외하고 p가 소수이기 때문에 0을 제외한 유한체의 원소 {1, 2, 3, ...p-1} 가 모두 곱셈 역원을 가지는 유한체 구조가 성립한다. (당연히 p mod p는 0 이기 때문에 p자신또한 제외된다)  
<ul>
<li><p>역원 : 곱셈연산으로 1, 덧셈연산으로 0이 나오는 수</p></li>
</ul>

또 다른 방법으로 증명해보자.  
아래식의 좌항은 유한체 F<sub>p</sub>이고 우항은 0이 아닌 임의의 원소n을 유한체의 모든 원소에 각각 곱한 집합이다.
<div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">  
<p>{1, 2, 3 ... p-2, p-1} = {n%p, 2n%p, 3n%p ... (p-2)n%p, (p-1)n%p}</p>
</div>

이 식을 다시 정리하면
<div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">  
<p>1 ⋅ 2 ⋅ 3 ⋅ ... ⋅(p-2)⋅(p-1) % p = n ⋅ 2n ⋅ 3n⋅ ... ⋅ (p-2)n ⋅ (p-1)n % p</p>
</div>

다시 축약하면 ( n! 은 팩토리얼(factorial)로 5! = 5⋅4⋅3⋅2⋅1 = 120와 같이 계산 )
<div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">  
<p> <span style="color : #be0000;">(p-1)!</span> % p = <span style="color : #be0000;">(p-1)!</span> ⋅ n<sup>(p-1)</sup> % p</p>
</div>

최종적으로 <span style="color : #be0000;">(p-1)!</span> 로 약분하면 페르마의 소정리 식이 된다.  
<div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">  
<p>1 = n<sup>(p-1)</sup> % p</p>
</div>
<br>

#### 나눗셈 연산에 응용하기

나눗셈은 곱셈의 역연산으로 아래와 같이 치환할 수 있다.  
<div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">  
<p>a / b = a · (1 / b) = a · b<sup>-1</sup></p>
</div>

공식을 이용하면
<div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">  
<p>b<sup>(p - 1)</sup> = 1</p>
</div>

따라서 아래와 같이 치환할 수 있다.
<div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">  
<p>b<sup>-1</sup> = b<sup>-1</sup> · 1 = b<sup><span style="color : #be0000;">-1</span></sup> · b<sup><span style="color : #be0000;">(p - 1)</span></sup> = b<sup><span style="color : #be0000;">(p - 2)</span></sup></p>
</div>

최종적으로 b의 역원과 b<sup>(p - 2)</sup> 는 같다.
<div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">  
<p style="color : #be0000;">b<sup>-1</sup> = b<sup>(p - 2)</sup> &nbsp; (mod p)</p>
</div>


<div class="ex_01" style="text-align: left; font-size: medium; color : #0300beff; margin-top: 20px;">
  ex) <br>  
  p = 19, b = 3 에서 페르마의 소정리를 사용하면
  <div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">  
  <p> 3<sup>(19 - 1)</sup> = 1 &nbsp;(mod 19)</p>
  <p> 3<sup>17</sup> · 3 = 1 &nbsp;(mod 19)</p>
  </div>

  양번을 3으로 나누면
  <div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">  
  <p> 3<sup>17</sup> = 3<sup>-1</sup> &nbsp;(mod 19) </p>
  <p style="color : #be0000;">b<sup>(p - 2)</sup> = b<sup>-1</sup> &nbsp;(mod p)</p>
  </div>
  <br>

  양변 3<sup>17</sup>과 3<sup>-1</sup> 은 모두 3의 곱셈 역원이되고 아래와 같이 표현가능하다.
  <div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">
  <p>3 · 3<sup>17</sup> = 3 · 3<sup>-1</sup> = 1 &nbsp; (mod 19) </p>
  </div>
</div>

모듈러연산에서 곱셈의 역원을 구하는 위의 방법을 사용하면 <b style="color : #be0000;">음의 지수를 양의 지수로 바꿀 수 있다.<b>

아래 예시의 나눗셈을 곱셈연산으로 치환해보자.
<div class="formula_01" style="text-align: left; font-size: medium; margin-top: 20px;">
ex) F<sub>31</sub><br>
  <ul>
  <li>
    <p>3 / 24 &nbsp; (mod 31)</p>
    <p>= 3 · 24<sup>(31 - 2)</sup> % 31</p>
    <p>= 3 · 24<sup>29</sup> % 31</p>
    <p>= 3 · (24<sup>16</sup> · 24<sup>8</sup> · 24<sup>4</sup> · 24<sup>1</sup>) % 31</p>
    <p>= 3 · (7 · 10 · 14 · 24) % 31 = 3 · 22 % 31</p>
    <p>= 4</p>
  </li>
  <li>
    <p>17<sup>-3</sup> &nbsp; (mod 31)</p>
    <p>= ( 17<sup>-1</sup>)<sup>3</sup> % 31</p>
    <p>= (17<sup>29</sup> )<sup>3</sup> % 31</p>
    <p>= 17<sup>87</sup> % 31</p>
    <p><b>페르마의 소정리에 의해 17<sup>(31-1)</sup> = 1 따라서 <span style="color : #be0000;">87 mod 30 = 27</span> 지수계산이 가능하다</b></p>
    <p>= 17<sup>-3</sup> = 17<sup>27</sup> % 31</p>
    <p>= 29 &nbsp; (mod 31)</p>
  </li>
  <li>
    <p>4<sup>-4</sup> · 11&nbsp; (mod 31)</p>
    <p>= 11 · (4<sup>-1</sup>)<sup>4</sup> % 31</p>
    <p>= 11 · (4<sup>29</sup>)<sup>4</sup> % 31</p>
    <p>= 11 · 4<sup>116</sup> % 31</p>
    <p>= 11 · 4<sup>26</sup> % 31</p>
    <p>= 13 &nbsp; (mod 31)</p>
  </li>
  </ul>
</div>
<br>

#### 정리
F<sub>p</sub> = {1, 2, 3...p-1} 유한체 소수p와 서로소관계인 n 사이에 다음의 관계가 성립한다.
<div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">
  <b >n<sup>(p-1)</sup> = 1  &nbsp; (mod p)</b>
</div>

<div class="formula_01" style="text-align: center; font-size: medium; margin-top: 20px;">
  <b >n<sup>(p - 2)</sup> = n<sup>-1</sup> &nbsp; (mod p)</b>
</div>