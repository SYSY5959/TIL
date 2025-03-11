
## DFS vs BFS 특징

| 비교 항목  | DFS (깊이 우선 탐색)       | BFS (너비 우선 탐색)       |
| ------ | -------------------- | -------------------- |
| 탐색 방법  | 한 방향으로 끝까지 탐색 후 백트래킹 | 현재 노드에서 가까운 노드부터 탐색  |
| 자료구조   | 스택(Stack) 또는 재귀      | 큐(Queue)             |
| 최단 경로  | 보장되지 않음              | 최단 경로 탐색에 유리         |
| 메모리 사용 | 상대적으로 적음             | 큐를 사용하므로 더 많은 메모리 필요 |
| 구현 방식  | 재귀 또는 스택             | 큐                    |

## 코테에서 언제 쓰는지!!!

| **문제 유형**             | **DFS** | **BFS** |
| --------------------- | ------- | ------- |
| **모든 경로를 탐색해야 하는 경우** | ✅       | ❌       |
| **최단 경로를 찾아야 하는 경우**  | ❌       | ✅       |
| **백트래킹이 필요한 경우**      | ✅       | ❌       |
| **그래프에서 연결 요소 개수 찾기** | ✅       | ❌       |
| **미로 탐색 (최단 거리)**     | ❌       | ✅       |
| **트리에서 특정 깊이까지 탐색**   | ✅       | ❌       |


#### 💡 정리하면
- **"최단 거리"를 찾으라고 하면? → BFS**
- **"모든 경우를 탐색"해야 하면? → DFS**
- **"백트래킹(완전 탐색)"이 필요하면? → DFS**
- **"그래프의 연결 요소 개수를 찾는 문제"라면? → DFS**
- **"레벨 순서대로 탐색해야 하면?" → BFS**




## 예제 코드 ; 백준 1260번

dfs : 방문 표시, 노드 append / ans += 1 → for : 인접 노드 → if 방문 X → dfs
bfs : q, v 선언 → 방문 표시, q append → while q → pop → for : 인접 노드 → if 방문 X → 방문 표시, q append


```python
def dfs(c):
    ans_dfs.append(c) # 방문노드 추가
    v[c] = 1 # 방문 표시
    
    for n in adj[c]:
        if not v[n]: # if v[n] == 0 -> 방문하지 않은 노드인 경우 
            dfs(n) # 방문
            
def bfs(s):
    q = [] # 필요한 q, v[], 변수 생성
    
    q.append(s) # q에 초기데이터 삽입
    ans_bfs.append(s)
    v[s] = 1
    
    while q: # 시작할 때 [3]
        c = q.pop(0)
        for n in adj[c]: # 시작할 때 c=3
            if not v[n]: # 방문하지 않은 노드 -> 큐에 삽입
                q.append(n)
                ans_bfs.append(n)
                v[n] = 1


n,m,V = map(int, input().split())
adj = [[] for _ in range(n+1)]
for _ in range(m):
    s,e = map(int, input().split())
    adj[s].append(e)
    adj[e].append(s) # 양방향
    
for i in range(1, n+1):
    adj[i].sort() # 각 노드를 오름차순 정렬

## 메인 블럭 다 만들어 놓고
## 함수는 제일 마지막에 호출!!

v = [0]*(n+1) # 방문 배열 초기화 [0, 0, ... , 0]
ans_dfs = []
dfs(V) # 시작 노드

v = [0]*(n+1) # 방문 배열 초기화
ans_bfs = []
bfs(V) # 시작 노드

print(*ans_dfs) # 리스트 안의 요소 출력 (,없이)
print(*ans_bfs)
```


# DFS (깊이 우선 탐색)

## 재귀함수 → DFS

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


## DFS 예제 
### 음료수 얼려먹기 - 백준
- 0인 뭉텅이 몇 개인지 찾기
- 모든 노드에서 dfs 실행하고 True인 갯수 찾기

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

    # 음료수 들어갈 수 있는 자리이면, 
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
        if dfs(i, j) == True:
            result += 1

print(result)

```

dfs 안에서 또 dfs를 호출하는 이유:
**현재 위치 `(x, y)`에서 0이 연결된 모든 방향을 탐색하기 위해** 다시 자기 자신을 호출


### 타겟 넘버 
numbers 배열 주어지면, 여기서 요소들을 더하거나 빼서 target으로 만들 수 있는 경우의 수 세기

```python
def solution(numbers, target):
	answer = 0
	def dfs(index, total):
		nonlocal answer
		if index == len(numbers):
			if total == target:
				answer += 1
			return
		dfs(index+1 , total + numbers[index])
		dfs(index+1 , total - numbers[index])
	dfs(0,0)
	return answer
```

[구현 수형도]
```markdown
numbers = [1,4,2,1]
target = 4

dfs(0, 0)  → index = 0 (첫 번째 숫자 선택)
 ├─ dfs(1, 1)  → index = 1 (두 번째 숫자 선택)
 │   ├─ dfs(2, 5)  → index = 2 (세 번째 숫자 선택)
 │   │   ├─ dfs(3, 7)  → index = 3 (네 번째 숫자 선택)
 │   │   │   ├─ dfs(4, 8)  → index == 4, total = 8 (체크)
 │   │   │   ├─ dfs(4, 6)  → index == 4, total = 6 (체크)
 │   │   ├─ dfs(3, 3)  → index = 3 (네 번째 숫자 선택)
 │   │       ├─ dfs(4, 4)  → index == 4, total = 4 (체크 ✅)
 │   │       ├─ dfs(4, 2)  → index == 4, total = 2 (체크)

```


[구현 트리]
```perl

                    dfs(0, 0)  ← index = 0
                   /          \
         dfs(1, 1)            dfs(1, -1)  ← index = 1
        /        \             /        \
 dfs(2, 5)    dfs(2, -3)   dfs(2, 3)   dfs(2, -1)  ← index = 2
   /    \       /    \       /    \       /    \
(끝)  (끝)  (끝)  (끝)  (끝)  (끝)  (끝)  (끝)  ← index = 4 (len(numbers))

# index = 3일 때 생략
```

### 바이러스 - 백준
- 1번 노드와 연결된 모든 노드의 개수 구하기 → dfs 안에서 방문처리 후 count += 1
- 그래프 표현: 2차원 배열로

```python
def dfs(c):
	global ans
	v[c] = 1
	ans += 1
	for n in arr[c]:
		if v[n] == 0:
			dfs(n)

n = int(input())
m = int(input())
arr = [[] for _ in range(n+1)] # 그래프 리스트 선언
for _ in range(m):
	s,e = map(int, input().split())
	arr[s].append(e)
	arr[e].append(s)

ans = 0
v = [0]*(n+1)
dfs(1)

print(ans-1)
```


### 네트워크 
- 연결된 노드끼리 한 뭉텅이. 뭉텅이 개수 구하기
- 노드 연결정보 바탕으로 2차원 배열 만들기

```python
## dfs : 방문처리 -> for -> if 
def solution(n, computers):
    answer = 0
    arr = [[] for _ in range(n+1)]
    for i in range(n):
        for j in range(n):
            if computers[i][j] == 1 and i != j: # 노드가 연결되어 있으면
                arr[i+1].append(j+1) # 그래프 형식으로 추가
    print(arr)
    
    def dfs(c):
        # dfs : c 입력 받아서 방문
        nonlocal answer
        v[c] = 1 # 방문 처리
        
        for n in arr[c]: # 이웃한 노드
            if v[n] == 0: # 그 노드를 아직 방문하지 않았다면
                dfs(n) # 방문 시작 ~
        
        return
    
    v = [0]*(n+1)
    for i in range(1,n+1): # 모든 노드에 대해 
        if v[i] == 0: # 아직 그 노드를 방문하지 않았다면
            dfs(i) # dfs 출발 (덩어리 찾기)
            answer += 1 # dfs 끝나면 개수 하나 추가 
    return answer
```


```python
def solution(n, computers):            
    
    def DFS(c):
        visited[c] = 1
        for a in range(n):
            if computers[c][a] and not visited[a]: ## 이 부분만 달라짐
                DFS(a)      
                
    answer = 0
    visited = [0 for i in range(len(computers))] # [0]*(len(computers))
    for i in range(n):
        if not visited[i]:
            DFS(i)
            answer += 1
        
    return answer
```

#### BFS 방법으로

```python
from collections import deque

def solution(n, computers):            
    
    def BFS(i):
        q = deque()
        q.append(i)
        while q:
            i = q.popleft()
            visited[i] = 1
            for a in range(n):
                if computers[i][a] and not visited[a]:
                     q.append(a)
                
    answer = 0
    visited = [0 for i in range(len(computers))]
    for i in range(n):
        if not visited[i]:
            BFS(i)
            answer += 1
        
    return answer
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

## BFS 예제 

### 미로탈출 (최단거리) - 백준
- BFS는 시작 지점에서 가까운 노드부터 차례대로 그래프의 모든 노드 탐색
- 상하좌우 연결된 모든 노드로의 거리가 1로 동일
	- (1, 1) 지점부터 BFS 수행하여 모든 노드의 최단거리 값을 기록하면 해결할 수 있음

![[IMG_208E594EE837-1.jpeg]]

```python
from collections import deque

# N, M 입력 받기
n, m = map(int, input().split())

# 2차원 리스트로 미로 정보 입력 받기
graph = []
for i in range(n):
    graph.append(list(map(int, input())))

# 이동할 네 방향 (상, 하, 좌, 우)
dx = [-1, 1, 0, 0]
dy = [0, 0, -1, 1]

# BFS 함수 정의
def bfs(x, y):
    queue = deque()
    queue.append((x, y))

    # 큐가 빌 때까지 반복
    while queue:
        x, y = queue.popleft()

        # 현재 위치에서 네 방향으로 이동
        for i in range(4):
            nx = x + dx[i]
            ny = y + dy[i]

            # 미로 공간을 벗어난 경우 무시
            if nx < 0 or nx >= n or ny < 0 or ny >= m:
                continue
            
            # 벽(0)인 경우 무시
            if graph[nx][ny] == 0:
                continue
            
            # 처음 방문하는 길(1)인 경우 최단 거리 기록
            if graph[nx][ny] == 1:
                graph[nx][ny] = graph[x][y] + 1
                queue.append((nx, ny))

    # 도착 지점의 최단 거리 반환
    return graph[n-1][m-1]

# BFS 실행 후 최단 거리 출력
print(bfs(0, 0))

```


- 미로 정보: map 형태 → `[list(map(int,input())) for _ in range(n)]`
- 최단 경로 까지 거리 구하는 거니까 v$[ci][cj]$ 에 개수 1씩 추가
```python
def bfs(si,sj,ei,ej):
    q = []
    v = [[0]*m for _ in range(n)] # 행 n개, 열 m개
    
    q.append((si,sj)) # 시작 좌표 추가
    v[si][sj] = 1 # 방문 표시
    
    while q:
        ci,cj = q.pop(0)
        if (ci,cj) == (ei,ej):
            return v[ei][ej]
        # 4방향, 범위 내, 조건에 맞으면: arr==1, v==0
        for di,dj in ((-1,0),(1,0),(0,-1),(0,1)):
            ni ,nj = ci+di, cj+dj
            if 0<=ni<n and 0<=nj<m and arr[ni][nj]==1 and v[ni][nj]==0:
                q.append((ni,nj))
                v[ni][nj] = v[ci][cj]+1

n,m = map(int,input().split())
arr = [list(map(int,input())) for _ in range(n)] # 문제에서 주어진 배열

ans = bfs(0,0,n-1, m-1) # 시작 좌표, 끝 좌표
print(ans)
```


### 단지 번호 붙이기 - 백준
- 1로 연결된 뭉텅이가 아파트 단지. 한 뭉텅이 속 1 개수 (단지 안에 있는 아파트 개수) 오름차순 출력
→ 리스트에 아파트 개수 append
- input: 아파트 정보 배열(0/1) map 형태
- → 2차원 배열로 `arr = [list(map(int,input())) for _ in range(n)]`

```python

def bfs(si,sj): # 시작위치 받아서, 해당 위치에 연결된 1의 개수 리턴
	q = [] # q 등 필요 변수 생성
	
	q.append((si,sj)) # 큐에 초기 데이터 삽입
	v[si][sj] = 1 # 방문표시
	cnt = 1 # 기타 정답 관련 작업
	
	while q: ## 갈 수 있는 경로 다 가고 나면 cnt 리턴
		ci,cj = q.pop(0)
		for di, dj in ((-1,0),(1,0),(0,1),(0,-1)):
			ni,nj = ci+di, cj+dj
			## 4방향, 범위내, 미방문이면 큐 삽입
			if 0<=ni<n and 0<=nj<n and v[ni][nj]==0 and arr[ni][nj]==1:
				q.append((ni,nj)) 
				v[ni][nj] = 1
				cnt +=1 
	return cnt


n = int(input())
arr = [list(map(int,input())) for _ in range(n)]

v = [[0]*n for _ in range(n)]
ans = [] # 아파트 개수

# arr 순회하면서 방문 가능하면 방문(bfs)
for i in range(n):
	for j in range(n):
		if v[i][j]==0 and arr[i][j]==1: # 노드에 방문한 적 없고, 방문 가능하면
			ans.append(bfs(i,j)) # 노드 방문 출발: bfs는 아파트 개수 반환
ans.sort()
print(len(ans), *ans, sep='\n')

```

**📌 BFS 함수 밖에서 전체를 순회하는 이유:**
- 문제의 핵심은 **서로 연결된 1들의 덩어리(단지)를 찾는 것**
- **방문하지 않은 1을 찾을 때마다 BFS를 새로 수행**
- BFS 실행 시, **그 단지에 속한 모든 1을 방문 처리**
- BFS가 끝나면 **새로운 BFS가 필요할 때까지 기다림**


➡️ **결론:**
- **여러 개의 독립된 덩어리를 찾는 문제이므로, BFS 호출 자체를 여러 번 수행해야 함.**
- **따라서 BFS를 직접 호출할 좌표를 찾는 과정이 BFS 함수 바깥에 있어야 함.**



### 촌수 계산 - 백준
- 나와 다른 사람이 몇칸 떨어져있는지 계산 → v[ ]에 개수 하나씩 추가
- input : 연결된 두 사람 → 그래프 노드로 만들기 : 그래프 형태 → `arr = [[] for _ in range(n+1)]`하고 for문에서 인접 노드 하나씩 추가

```python
def bfs(s,e):
    q = []
    
    q.append(s)
    v[s] = 1
    
    while q:
        c = q.pop(0)
        # 마지막 노드 도달하면 끝냄!!!1
        if c == e:
            return v[e]-1 # 나와 한칸 떨어져 있으면 촌수:1이니까 자기 자신 제외
        
        # c와 연결된 번호인 경우 미방문이면 방문!!
        for n in arr[c]:
            if v[n] == 0:
                q.append(n)
                v[n] += v[c]+1
    # 이곳의 코드를 실행했다면.. 연결고리 찾지 못함. 연결고리 없음
    return -1
        

n = int(input()) # 사람 수
S,E = map(int,input().split()) # 촌수 구하고 싶은 두 사람
m = int(input()) # 연결 개수
arr = [[] for _ in range(n+1)]
for _ in range(m):
    p,c = map(int, input().split()) # 연결된 노드
    arr[p].append(c)
    arr[c].append(p)

v = [0]*(n+1)
ans = bfs(S,E)
print(ans)
```

### 토마토 - 백준 7569

![[IMG_7E05AD733E24-1.jpeg]]
- input: 토마토 정보 3차원. map 형태
- output: 토마토 익는 데 걸리는 시간 → $v[ch][ci][cj]$ 에 1씩 추가하기

```python
from collections import deque

def bfs(): # 안익은 토마토 모두 익는데 걸리는 날짜. 
    # 1. q, v[] 생성
    q = deque()     
    v = [[[0]*M for _ in range(N)] for _ in range(H)]
    
    # 2. q에 초기 데이터 삽입, 안익은 토마토 카운트
    cnt = 0 # cnt: 안익은 토마토 개수
    for h in range(H):    # 전체 순회하면서 처리
        for i in range(N):
            for j in range(M):
                if arr[h][i][j] == 1: # 익은 토마토
                    q.append((h,i,j))
                    v[h][i][j] = 1
                elif arr[h][i][j] == 0: # 안익은 토마토
                    cnt += 1 
                    
    while q:
        ch,ci,cj = q.popleft()
        
        # 6방향 범위내, 미방문, arr[]==0
        for dh,di,dj in ((-1,0,0),(1,0,0),(0,-1,0),(0,1,0),(0,0,1),(0,0,-1)):
            nh,ni,nj = ch+dh, ci+di, cj+dj
            if 0<=nh<H and 0<=ni<N and 0<=nj<M and v[nh][ni][nj]==0 and arr[nh][ni][nj]==0:
                q.append((nh,ni,nj))
                v[nh][ni][nj] = v[ch][ci][cj] + 1
                cnt -= 1 # 안익은 토마토 한개 익음
    if cnt == 0: # 모든 토마토 익었음
        return v[ch][ci][cj]-1
    else:
        return -1
    

M,N,H = map(int, input().split()) # m:col n:row
arr = [[list(map(int,input().split())) for _ in range(N)] for _ in range(H)]
ans = bfs()
print(ans)
```

*이 토마토 문제에서*
**📌 BFS 함수 내부에서 전체를 순회하는 이유:**
- 익은 토마토(`1`)가 **동시에** 여러 곳에서 퍼져나가야 함 → **출발점이 여러 개이고 동시에 진행해야 함**
- BFS 실행 전에 **모든 익은 토마토를 큐에 한꺼번에 넣어야 함**
- `cnt`를 세서 남은 안 익은 토마토(`0`) 개수를 관리해야 함
- BFS 과정에서 `cnt -= 1`로 줄여가면서 **모든 토마토가 익었는지 판단**

➡️ **결론:**
- 토마토 익히기 문제에서는 **여러 출발점을 한 번에 큐에 넣고 동시에 시작하는 BFS**를 해야 함.
- **따라서 BFS 내부에서 전체 배열을 순회하면서 초기 상태를 큐에 넣는 과정이 필요**함.

*다른 문제에서...*
**📌 BFS 함수 외부에서 전체를 순회하는 이유:**
- **출발점이 하나씩 주어지고, 따로따로 BFS를 실행해야 함**
- BFS를 실행할 필요가 있는 곳(방문하지 않은 1)을 찾을 때마다 새로 시작
- BFS는 하나의 덩어리(단지)만 처리하고 종료

| **구분**        | **BFS 함수 안에서 전체 순회**       | **BFS 함수 밖에서 순회**          |
| ------------- | -------------------------- | -------------------------- |
| **언제 사용?**    | 여러 개의 출발점이 동시에 시작해야 할 때    | 출발점을 하나씩 찾아서 BFS를 반복해야 할 때 |
| **예제 문제**     | 토마토 익히기, 불이 퍼지는 문제 등       | 단지 찾기, 섬 개수 세기, 미로 탐색 등    |
| **BFS 실행 횟수** | 1번만 실행 (큐에 여러 출발점 추가)      | 여러 번 실행 (필요할 때마다 BFS 호출)   |
| **출발점 처리 방식** | BFS 실행 전에 미리 모든 출발점을 큐에 넣음 | BFS 실행할 좌표를 하나씩 찾아가며 호출    |

### 숨바꼭질 - 백준 1697
수빈이는 동생과 숨바꼭질을 하고 있다. 수빈이는 현재 점 N(0 ≤ N ≤ 100,000)에 있고, 동생은 점 K(0 ≤ K ≤ 100,000)에 있다. 수빈이는 걷거나 순간이동을 할 수 있다. 만약, 수빈이의 위치가 X일 때 걷는다면 1초 후에 X-1 또는 X+1로 이동하게 된다. 순간이동을 하는 경우에는 1초 후에 2*X의 위치로 이동하게 된다. 수빈이가 동생을 찾을 수 있는 가장 빠른 시간이 몇 초 후인지 구하는 프로그램을 작성하시오.
수빈이가 있는 위치 N과 동생이 있는 위치 K가 주어진다.


```python
## 예: 5, 17
# 5 → 4,6,10 → ... → bfs
from collections import deque

def bfs(s,e):
	q = deque()
	q.append(s)
	v[s] = 1
	
	while q:
		c = q.popleft()
		if c == e:
			return v[e]-1
		for n in (c-1, c+1, 2*c):
			if 0<=n<200001 and v[n]==0:
				v[n] = v[c]+1
				q.append(n)
	return -1

N,K = map(int, input().split())
v = [0]*200001 # 적당히 여기까지 할당해줌..
print(bfs(N,K))

```  

