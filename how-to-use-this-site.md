### 전반적인 사용법
minimal-mistake의 전반적인 사용법은 [이곳](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)을 참고하면 됩니다.<br>
config 파일을 수정할때, url과 baseurl값은 설정하지 않아야 config값이 적용됩니다.<br>
블로그 포스트는 [이곳](https://ansohxxn.github.io/blog/posting/)을 참고하여, markdown 파일로 작성하면 됩니다.<br>
markdown syntax는 [이곳](https://www.markdownguide.org/basic-syntax/)을 참고하시면 됩니다.
 
 
### 사이드바 커스터마이즈
includes 폴더에 있는 nav_list_main 파일을 통해 커스터마이즈 할 수 있습니다. <br>
이후, 해당 파일을 sidebar.html에 적용시켜야 하는데, 이러한 방법은 [사이드바 사용법](https://ansohxxn.github.io/blog/category/)을 읽어보면 쉽게 적용할 수 있습니다.


### 폰트 바꾸기
[이곳](https://oilmlio.com/blog/Change-the-GitHub-Blog-Font-RIDIBatang/)을 참고하시면 됩니다.


### index.html에 있는 Quick-Start Guide 지우는 방법
data 폴더에 있는 navigation.yml 파일에서 지우면 됩니다. <br>
이때, Quick-Start Guide 대신 다른 url과 title을 설정하여, 원하는 navigation button을 만드는 것도 가능합니다.


### github 블로그를 VS Code에 clone 하는 방법
1. git for windows를 설치
2. git cmd를 열어서 git config --global user.name "username"을 입력
3. git config --global user.email "email"을 입력
4. VS Code을 실행 후, F1을 누른 후, git: clone을 검색하여 enter를 입력
5. 알맞는 repository를 선택 (이때, sign-in과정이 진행됨)
6. 이후, clone된 repository를 저장할 local address를 선택
7. Done


### clone된 repository를 VS Code에서 작성 후, 변경사항을 적용시키는 방법
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
local address에 저장된 폴더를 지우면 됩니다. <br>
이후, 다시 사용하고 싶으면, clone을 동일한 방법으로 진행하면 됩니다.


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
8. 주의할점 (가끔씩 vscode에서는 이미지를 포함한 포스팅이 업로드 안될수도있다. 이때에는 그냥 github에서 직접 포스팅해주면 된다.
