vs code 열기
git이 연결되어있는 로컬 폴더를 들어가기
- seungyeon/TIL 열기
- on main이 되어있는지 확인! (TIL을 열면 되어있을거임)

on main : git에 연결이 되어있는 상태 
없으면, git 폴더로 들어가기
1. 바보같이 파일 open  folder
2. 멋있게,,  cd 폴더명 (cd →  tab 누르기)

작업 시작 전 변동사항 가져오기
```shell
git fetch
git pull origin main
```


``` shell

git add . # 내가 올린 파일을 깃허브가 볼 수 있도록 해줌 / tab으로 찾기
git commit -m "MESSAGE" # 이런 변경사항이 있습니다~
git push origin main # origin(내 맥북)을 main 브랜치에 push(올리겠습니다)

```


### 꿀팁
문장 앞 뒤 : command + → / <- 
한 토큰 앞 뒤 : option + → / <- 
여기 shift 동시에 누르면 드래그 가능

#### vs code 한정
멀티 커서 : option + 클릭
문장 위치 바꾸기 : option + 위 / 아래
문장 복사 : option + shift + 위 / 아래

