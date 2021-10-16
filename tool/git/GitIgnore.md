# GitIgnore

git 으로 작업할 때, commit에 포함시키고 싶지 않을 때가 있다.  
commit 에 포함시키지 않기 위해서는 그 규칙을 해당 repository 최상위에 .gitignore 를 만들고  
해당 파일에 규칙들을 적어 넣어주면 된다.

Itellij 기반의 좋은 IDE에서는 gitignore 파일을 내부에서 만들 수도 있고,  
AndroidStudio 등에서는 프로젝트 생성시에 자동으로 gitignore 파일이 만들어지는 것을 확인할 수 있다.  
하지만, 자동으로 만들어진 파일들은 원하는 내용이 들어가 있지 않은 경우가 많으므로  
이 문서에서는 온라인 웹사이트에서 gitignore 파일을 만드는 방법 + 커스텀 하는 방법에 대해서 설명하려고 한다.

## 웹사이트에서 gitignore 파일 만들기

1. [toptal-gitignore](https://www.toptal.com/developers/gitignore) 사이트에 접속한다.
2. 정확한 gitignore 파일 작성을 위해 아래 내용을 적어준다.  
  - OS (MacOS, Linux, Windows, etc...)
  - IDEs (Visual Studio Code, Intellij, Android Studio, etc...)
  - Programming Languages (Java, Kotlin, Python, etc...)
  - Framework (OpenFrameworks, Firebase, etc...)
3. 이렇게 추가해주고 난 후 create 버튼을 눌러주면, 새창이 열리면서 gitignore 파일에 집어넣을 내용이 나온다.  
4. 해당 텍스트를 gitignore 파일에 넣기만 하면 된다.

## gitignore 커스텀하기

상황에 따라서 gitignore 파일을 손봐야 되는 경우도 더러(?) 있다.
이 경우에는 아래 문법에 맞게 원하는 부분을 추가해주면 된다.

| Pattern Format | Description |
| `blank line(공백)` | 공백은 아무런 역할도 하지 않는다. 단순히 내용 분리와 가독성을 위해 존재한다. |
| `hash(#)` | 주석을 의미한다. (e.g. # This is comment) |
| `asterisks(*)` | 모든을 의미한다. |
| `Two consecutive asterisks(**)` | 더 넓은 의미의 모든을 의미한다. |
| `important mark(!)` | 앞에 ! 가 붙어있는 부분은 ignore 에서 제외시킨다. (e.g. !src) |
| `*.foo` | 모든 확장자가 foo 인 파일들을 ignore 시킨다. |
| `/*.foo` | 현재 폴더의 모든 확장자가 foo 인 파일들을 ignore 시킨다. |
| `foo.*` | foo 로 시작하는 모든 파일 및 폴더들을 ignore 시킨다. |
| `**/foo` | 모든 foo 를 ignore 시킨다. |
| `/**/foo` | 현재 폴더의 모든 foo 를 ignore 시킨다. |
| `a/**/b` | a 폴더 안에있는 모든 b 를 ignore 시킨다. (e.g. `a/b`, `a/x/b`, `a/x/y/b`) |

예를 들어, key.xml 파일을 commit 에 포함시키지 않고 싶다면 아래처럼 추가해주면 된다.

```gitignore
key.xml
```

프로젝트 전체에서 `foo/bar` 만을 추가하고 싶은 경우에는 아래처럼 추가해주면 된다.  

```gitignore
/*
!/foo
/foo/*
!/foo/bar
```

