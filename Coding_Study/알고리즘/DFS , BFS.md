

## 재귀함수

- 펙토리얼 구현 예제
```python

# 반복적으로 구현한 n!
def factorial_iterative(n):
    result = 1
    # 1부터 n까지의 수를 차례대로 곱하기
    for i in range(1, n + 1):
        result *= i
    return result

# 재귀적으로 구현한 n!
def factorial_recursive(n):
    if n <= 1:  # n이 1 이하인 경우 1을 반환
        return 1
    # n! = n * (n - 1)!를 그대로 코드로 작성하기
    return n * factorial_recursive(n - 1)

# 각각의 방식으로 구현한 n! 출력 (n = 5)
print('반복적으로 구현:', factorial_iterative(5))
print('재귀적으로 구현:', factorial_recursive(5))


```

# DFS (깊이 우선 탐색)


## DFS 코드 구현

노드 1부터 시작. 가장 작은 노드부터 탐색

![[스크린샷 2025-02-28 오후 4.33.06.png|500]]

```python
def dfs(gragh, v, visited):

  # 현재 노드를 방문 처리
  visited[v] = True
  print(v, end=' ')

  # 현재 노드와 연결된 다른 노드를 재귀적으로 방문
  for i in gragh[v]:
    if not visited[i]:
      dfs(gragh, i, visited)

# 각 노드가 연결된 정보를 표현 (2차원 리스트)
graph = [
  [], # 0 번째 노드는 비워두기
  [2, 3, 8], # 노드1과 연결된 노드
  [1, 7],
  [1, 4, 5],
  [3, 5],
  [3, 4],
  [7],
  [2, 6, 8],
  [1, 7]
]

# 각 노드가 방문된 정보를 표현 (1차원 리스트)
visited = [False]*9

# 정의된 DFS 함수 호출
dfs(graph, 1, visited)

## 결과
1 2 7 6 8 3 4 5
```


# BFS (너비 우선 탐색)

- 시작 노드부터 가까운 노드부터 우선적으로 탐색하는 알고리즘 → 큐 자료구조 이용
- 탐색 시작 노드를 큐에 넣고 방문 처리
- 큐에서 노드를 꺼낸  뒤에 해당 노드의 인접 노드 중에서 방문하지 않은 노드를 모두 큐에 삽입하고 방문 처리

 
![[IMG_32276C2BE76C-1.jpeg]]
탐색 순서 : 1 → 2 → 3 → 8 → 7 → 4 → 5 → 6
방문한 노드 빼내고 인접한 노드 넣기

```python

from collections import deque

def bfs(graph, start, visited):
    queue = deque([start])  # 시작 노드를 큐에 넣기
    visited[start] = True

    while queue:
        v = queue.popleft()  # 큐에서 하나의 원소를 뽑음 -> popleft 하면 첫번째 요소 반환됨
        print(v, end=' ')  # 방문한 노드 출력

        for i in graph[v]:  # 인접한 모든 노드 확인
            if not visited[i]:  # 방문하지 않았다면
                queue.append(i)  # 큐에 추가
                visited[i] = True  # 방문 처리

graph = [
    [], 
    [2, 3, 8], 
    [1, 7],
    [1, 4, 5],
    [3, 5],
    [3, 4],
    [7],
    [2, 6, 8],
    [1, 7]
]

visited = [False] * 9
bfs(graph, 1, visited)

## 결과
1 2 3 8 7 4 5 6

```

## DFS 예제 - 음료수 얼려먹기

```python
# 음료수 얼려 먹기 문제 - DFS
n, m = map(int, input().split())

# 2차원 리스트의 맵 정보 입력 받기
graph = []
for i in range(n):
    graph.append(list(map(int, input())))

# DFS로 특정 노드를 방문하고 연결된 모든 노드들도 방문
def dfs(x, y):
    # 주어진 범위를 벗어나면 즉시 종료
    if x < 0 or x >= n or y < 0 or y >= m:
        return False

    # 현재 노드를 방문하지 않았다면
    if graph[x][y] == 0:
        # 방문 처리
        graph[x][y] = 1
        
        # 상, 하, 좌, 우 탐색
        dfs(x - 1, y)  # 상
        dfs(x + 1, y)  # 하
        dfs(x, y - 1)  # 좌
        dfs(x, y + 1)  # 우
        return True
    
    return False

# 모든 위치에서 DFS 수행하여 연결된 덩어리 개수 세기
result = 0
for i in range(n):
    for j in range(m):
        # 현재 위치에서 DFS 수행
        if dfs(i, j):
            result += 1

print(result)

```

dfs 안에서 또 dfs를 호출하는 이유:
**현재 위치 `(x, y)`에서 0이 연결된 모든 방향을 탐색하기 위해** 다시 자기 자신을 호출