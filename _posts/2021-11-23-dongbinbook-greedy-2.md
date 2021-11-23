---
title: "이코테(AKA 동빈북) 문제 풀기(숫자 카드 게임)  python"
use_math: true
categories:
  - dongbinbook
  - algorithm
  - greedy
tags:

---


# Greedy 실전문제  

## 숫자 카드 게임  

1번 문제보다 훨씬 쉬운 문제였다.  



### 1. 문제 읽기  

- 각 행의 가장 낮은 숫자 뽑기  
- 그 안에서 가장 높은 숫자 뽑기  

책에서 언급한 것처럼, 굉장히 쉬운 문제였다.  

### 2. 제출 코드  

```python
n, m = map(int, input().split())

arr = []
# 각 행의 가장 낮은 숫자 뽑기
for i in range(n):  # 각 행을 돌면서
    arr.append(min(map(int, input().split())))
# 그 안에서 가장 높은 숫자 뽑기
print(max(arr))
```


### 3. 공부할 것  


