---
title: "이코테(AKA 동빈북) 문제 풀기(큰 수의 법칙)  python"
use_math: true
categories:
  - dongbinbook
  - algorithm
  - greedy
tags:

---


# Greedy 실전문제  

## 큰 수의 법칙  

난이도가 `하`여서 좀 우습게 봤는데 풀이 제한시간 30분을 꼬박 썼다..  



### 1. 문제 읽기  

- m을 초과하면 멈춘다  
- m을 초과하기 전까지 for문 + 1로 반복  
- 정렬 후 큰 수 두 개만 사용  

간단하게 말하면 k 값 만큼 가장 큰 값을 반복하고, 1번만 두번째 큰 값을 더해주면 된다.  
이 로직을 m을 초과하기 전까지 더해주면 된다.  

### 2. 제출 코드  

count 변수를 만들지 않아도 더할때마다 m에서 1씩 빼주고, m이 0일 때 반복문을 종료해주는 조건문을 달아주면 간단하게 해결된다.  

```python
n, m, k = map(int, input().split())
arr = list(map(int, input().split()))

arr.sort(reverse=True) 

ans = 0
count = 0
while 1:
    for _ in range(k):  # 최대 반복 가능 수 k
        if count >= m:
            break
        count += 1
        ans += arr[0]

    if count >= m:
        break
    count += 1
    ans += arr[1]

print(ans)
```


### 3. 공부할 것  

책에서 언급한 것처럼, M 크기가 더 커져서 기존 코드가 시간초과 되었다고 가정하고, 더 효율적인 문제 코드를 만들어보자.  



m이 8이고, k가 3일 때,   
(k+1) +( k+1) 의 수열이 반복된다.  
따라서 M을 k+1로 나눈 몫이 수열이 반복되는 횟수이다.  
그리고 몫에 k를 곱해주면 가장 큰 수가 등장하는 횟수가 된다.  
또한 m이 수열로 나누어 떨어지지 않는 경우는 나머지만큼 가장 큰 수가 추가로 더해지므로   
가장 큰 수의 반복 횟수는 m을 k+1으로 나눈 몫\*k과 나머지를 더한 수가 된다.   
두번째로 큰 수의 반복 횟수는 m에서 가장 큰 수의 반복 횟수를 빼주면 된다.  
두 반복 횟수를 더해주면 답이 도출된다.  

```python
n, m, k = map(int, input().split())
arr = list(map(int, input().split()))

arr.sort(reverse=True)

ans = 0

max1_num_cnt = k*(m//(k+1)) + m%(k+1)
max2_num_cnt = m-max1_num_cnt

ans += max1_num_cnt*arr[0]
ans += max2_num_cnt*arr[1]

print(ans)
```
