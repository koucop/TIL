# Git

## 설치

- [git](https://git-scm.com/downloads)에서 내용 확인 후 다운받기
- [GUI Clients](https://git-scm.com/downloads/guis/)에서 사용가능한 GUI 확인할 수 있다

## 명령어 모음

| `git status` | 현재 변경점을 확인한다. |
| --- | --- |
| `git add <file_name>` | 특정 파일을 stage에 올린다 |
| `git add <folder_name>` | 특정 폴더를 stage에 올린다 |
| `git add .` | 모든 변경점을 stage에 올린다 |
| | |
| `git checkout <branch_name>` | <branch_name> 으로 브랜치를 변경한다 |
| `git checkout -b <branch_name>` | 현재 브랜치로 새로 브랜치를 만들고 스위치 한다 |
| | |
| `git branch -av` | 모든 브랜치를 확인한다 |
| `git branch -D <branch_name>` | 작업 때 사용한 local 의 브랜치를 지워준다 |
| | |
| `git fetch -p -all` | 변경된 것을 fetching 한다 |
| | |
| `git pull upstream <branch_name>` | upstream 브랜치에서 locale 브랜치로 pull 받아온다 |
| | |
| `git commit -m "<message>"` | 현재 add된 stage를 메시지와 함께 커밋한다 |
| `git commit —-amend` | 현재 stage의 변경점을 이전 커밋에 합친다 |
| | |
| `git config --global pager.branch false` | 브랜치를 본다던지 할 때 새로운 페이지가 아닌 지금 페이지에서 보이도록 설정 |
