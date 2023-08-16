# git 사용법
1. git 생성
```bash
git init
```
2. 스테이징 영역에 추가
```bash
git add [파일이름]
```
3. 추가된 파일을 커밋 -> 버전 생성
```bash
git commit -m '변경된 내용 텍스트'
```

- Untracked files : 한번도 저장된 적 없는 파일

- Changes not staged for commit : 스테이지가 되지 않은 파일 

- Changes to be committed : 커밋이 될 파일들 
    
4. origin url 지정
```bash
git remote add origin [url]
```

5. 로컬 저장소 branch `master`의 커밋을 리모트 저장소 origin 에 push 한다
```bash
git push origin master
```
---
**버전 마다 달라진 점을 확인하는 방법**
```bash
git diff HEAD HEAD~1
```

- 현재 버전상태와 다른점을 확인하는 명령어
```bash
git status
```

