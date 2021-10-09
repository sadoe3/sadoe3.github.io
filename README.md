# Github.io 사용하는 방법

### 전반적인 사용법
- minimal-mistakes의 전반적인 사용법은 [이곳](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)을 참고하면 됩니다.
- config 파일을 수정할때, url과 baseurl값은 설정하지 않아야 config값이 적용됩니다.
- config 파일에서 layout값을 single로 설정해야 toc(table of contents)를 사용할수 있습니다.
- 블로그 포스트는 [이곳](https://ansohxxn.github.io/blog/posting/)을 참고하여, markdown 파일로 작성하면 됩니다.
- markdown syntax는 [이곳](https://www.markdownguide.org/basic-syntax/)을 참고하시면 됩니다.
- README.md 파일을 만들면, github.io repository를 들어왔을 때, 해당 repository에 대한 설명에 관한 내용을, README.md 파일을 통해 보여줄 수 있다
 
 
### jekyll-theme의 색상 변경하는 방법
1. \_sass -> minimal-mistakes -> skins -> 에서 알맞은 스킨 파일을 선택합니다.
2. 이후 변경하고자 하는 것의 hex값을 원하는 색의 hex값으로 변경하면 됩니다.
 
 
### 사이드바 커스터마이즈
- includes 폴더에 있는 nav_list_main 파일을 통해 커스터마이즈 할 수 있습니다.
- 이후, 해당 파일을 sidebar.html에 적용시켜야 하는데, 이러한 방법은 [사이드바 사용법](https://ansohxxn.github.io/blog/category/)을 읽어보면 쉽게 적용할 수 있습니다.
- 해당 사이트를 참고하여 naming을 동일하게 , sidebar_main: true 값을 적용해야 sidebar가 적용된다.


### 폰트 바꾸기
- [이곳](https://oilmlio.com/blog/Change-the-GitHub-Blog-Font-RIDIBatang/)을 참고하시면 됩니다.


### index.html에 있는 Quick-Start Guide 버튼 지우는 방법
- \_data 폴더에 있는 navigation.yml 파일에서 지우면 됩니다.
- 이때, Quick-Start Guide 대신 다른 url과 title을 설정하여, 원하는 navigation button을 만드는 것도 가능합니다.


### index.html에 있는 Categories 버튼 추가하는 방법
- github에서는 자동적으로 category별로 포스트를 정렬해준 페이지를 제공합니다.
- 해당 페이지의 주소는 {사이트주소}/categories/ 인데, 이 주소를 추가하면 되는 것입니다.
- 즉, 위에서 나온 방법을 적용하여, data 폴더에 있는 navigation.yml 파일에 새로운 title을 Categories로하고, 새로운 url을 해당 주소로 설정해주시면 됩니다.


### Github 블로그를 VS Code에 clone 하는 방법
1. git for windows를 설치
2. git cmd를 열어서 git config --global user.name "username"을 입력
3. git config --global user.email "email"을 입력
4. VS Code을 실행 후, F1을 누른 후, git: clone을 검색하여 enter를 입력
5. 알맞는 repository를 선택 (이때, sign-in과정이 진행됨)
6. 이후, clone된 repository를 저장할 local address를 선택
7. Done
- 주의 할 점: 가끔씩 github관련 terminal을 작성해야 하는 경우가 있다.
- 이러한 경우, git cmd에서 진행하는 것이 아니라, vs code에 있는 terminal에서 진행을 해야한다.


### Clone된 repository를 VS Code에서 작성 후, 변경사항을 적용시키는 방법
1. 변경사항을 저장
2. vs code 왼쪽 하단에 synchronize changes 선택
3. vs code 왼쪽 부분에 source control 선택
4. changes category에 왼쪽 부분에 있는 +(stage all changes) 선택
5. 위쪽 버튼들 중 체크표시버튼(commit)을 선택
6. commit message를 입력후 enter 입력
7. 이후, 위쪽 버튼들 중 ... 버튼을 선택후, push to 선택
8. 알맞은 repository를 선택
9. Done


### VS Code에서 clone 된 repository 제거 방법
- local address에 저장된 폴더를 지우면 됩니다. 


### VS Code에서 clone 된 repository 현재 변경 상태로 sync 맞추는 방법
- 위에서 알려준 respository 제거방법을 이용한 후, 해당 repository를 다시 clone하면 됩니다.


### Github 블로그에 코드를 올리는 방법
1. backtick(\`)문자 3개를 연속으로 쓴 후, 원하는 프로그래밍 언어를 적은 다음 다음 줄부터 원하는 코드를 복사 및 붙여넣기 합니다.
2. 다 붙여넣었으면, 마지막 코드 다음 줄에 backtick(\`)문자 3개를 연속으로 적음으로써 code block를 끝내시면 됩니다.
- 예) c++ code block <br>
\`\`\`c++ <br>
~~ // this is the sample code <br>
\`\`\` <br>


### Github 블로그에 이미지를 올리는 방법
[이곳](https://ahribori.com/article/5a03bcfd6c9eef13d882e29a)을 참고하면 됩니다. <br>
간단하게 설명하자면 <br>
1. 이미지 업로드 용 repository를 private visibility로 만든다.
2. Issues 선택, create issue 선택
3. Drag & Drop으로 이미지 업로드
4. 그러면 자연스럽게 markdown syntax로 되어있는 업로드된 사진의 링크가 나오게됨
5. 해당 링크들을 복사해서 포스트할 곳에 붙여넣는다.
6. 조심해야할 점은 링크를 복사한 후 꼭 br tag를 써줘야한다 (그렇지 않으면, 사진이 링크로 출력됨)
7. 이후 해당 issue를 open 하고, 바로 close 해주면 된다.


### VS Code에서 push를 하는데, git error: failed to push some refs to remote 라는 오류가 발생하는 경우
- 이러한 경우 [이곳](https://stackoverflow.com/questions/24114676/git-error-failed-to-push-some-refs-to-remote)을 참고하자.
- 다음은 간단한 해결 방법이다.
1. VS Code의 terminal을 실행한다. 
 - Terminal -> New Terminal
2. 다음 코드를 작성한다.
 ```
git pull --rebase
git push
 ```
3. 그렇게 되면, 오류가 발생했던 변경사항이 성공적으로 push가 된다.
