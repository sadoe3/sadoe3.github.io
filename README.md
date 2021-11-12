# Github.io 사용하는 방법

### Github 블로그 만드는 방법
1. 원하는 Jekyll Theme을 결정하기
    - 다양한 theme들이 있는데, 보통 minimal-mistakes를 자주 사용함
2.  해당 theme의 repository를 fork하기
    - minimal-mistakes의 repository는 [여기](https://github.com/mmistakes/minimal-mistakes)를 클릭하여 접근할수있음
4.  fork된 repository의 이름을 username.github.io 로 바꿔 주기
5.  이후, 아래에 적어둔 전반적인 사용법을 참고하여, 블로그를 자신이 원하는대로 customize해주면 됨

### 전반적인 사용법
- minimal-mistakes의 전반적인 사용법은 [이곳](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)을 참고하면 됩니다.
- config 파일을 수정할때, url과 baseurl값은 설정하지 않아야 config값이 적용됩니다.
- config 파일에서 layout값을 single로 설정해야 toc(table of contents)를 사용할수 있습니다.
  - toc_sticky 값을 true로 설정하면, scroll을 내려도 toc가 따라 내려오게 설정할 수 있습니다.
- 블로그 포스트는 [이곳](https://ansohxxn.github.io/blog/posting/)을 참고하여, markdown 파일로 작성하면 됩니다.
- markdown syntax는 [이곳](https://simhyejin.github.io/2016/06/30/Markdown-syntax/)을 참고하시면 됩니다.
- 다른 궁금증은 googling을 통해 해결할 수 도 있습니다.
- README.md 파일을 만들면, github.io repository를 들어왔을 때, 해당 repository에 대한 설명에 관한 내용을, README.md 파일을 통해 보여줄 수 있다
 
 
### jekyll-theme의 색상 변경하는 방법
1. \_sass -> minimal-mistakes -> skins -> 에서 알맞은 스킨 파일을 선택합니다.
2. 이후 변경하고자 하는 것의 hex값을 원하는 색의 hex값으로 변경하면 됩니다.
 
 
### 사이드바 커스터마이즈
- includes 폴더에 있는 nav_list_main 파일을 통해 커스터마이즈 할 수 있습니다.
- 이후, 해당 파일을 sidebar.html에 적용시켜야 하는데, 이러한 방법은 [사이드바 사용법](https://ansohxxn.github.io/blog/category/)을 읽어보면 쉽게 적용할 수 있습니다.
- 해당 사이트를 참고하여 naming을 동일하게 , sidebar_main: true 값을 적용해야 sidebar가 적용된다.


### comment 구현하기
- 가이드에 나와있는 내용인데, 간단하게 설명하자면 다음과 같습니다.
- 다양한 방식으로 comment를 구현할 수 있지만, 저는 utterances를 사용하는 방식을 정리하겠습니다.
1. \_config.yml 파일에서 만약 새로운 줄을 추가하지 않았다면, 26번째 줄에 repository값을 설정할 수 있는 코드가 있을 것입니다.
2. 이 값을 "username/repo-name"와 같은 형식으로 지정해 줍니다. 
3. 이후, utterances값에서 theme의 값을 "github-light" 이나 "github-dark" 중 원하시는 것으로 적용해줍니다.
4. issue_term의 값은 "pathname"을 적용해주시면 됩니다.
    - 이때. pathname은 따로 변경하지말고, 영어단어 있는 그대로 적으시면 됩니다.
6. 이렇게 하시면 comment를 사용할 수 있게 됩니다.


### 방문자수 구현하기
- [이곳](https://choiseonjae.github.io/jekyll/hits/)을 참고하시면 됩니다.
- HITS 페이지에서 링크를 생성할 때, 그냥 URL만 복사해서 사용하지말고, URL복사 후에 아이콘이나 TITLE같은 것들도 설정해준 후에 링크를 사용해주시는 것이 좋습니다.
- 방문자수를 출력하는 코드는 siderbar.html 파일에서 사이드바 출력하는 코드와 <\\div> 사이에 넣어주시면 됩니다.
- 방문자수를 사용자가 클릭하는 것을 원하지 않을 경우, a tag를 없애주시면 됩니다.
- 위에서 언급한 참고용 블로그를 보시면 img tag용 style을 위한 class를 만들었는데, 굳이 코드의 길이를 늘리고 싶지 않으시다면, style tag를 지우시고, style tag에 있던 값들을 img tag의 style attribute의 값에 넣어주시면 됩니다. 

### 폰트 바꾸기
- [이곳](https://oilmlio.com/blog/Change-the-GitHub-Blog-Font-RIDIBatang/)을 참고하시면 됩니다.


### index.html에 있는 Quick-Start Guide 버튼 지우는 방법
- \_data 폴더에 있는 navigation.yml 파일에서 지우면 됩니다.
- 이때, Quick-Start Guide 대신 다른 url과 title을 설정하여, 원하는 navigation button을 만드는 것도 가능합니다.


### index.html에 있는 Categories 버튼 추가하는 방법
- github에서는 자동적으로 category별로 포스트를 정렬해준 페이지를 제공합니다.
- 해당 페이지의 주소는 {사이트주소}/categories/ 인데, 이 주소를 추가하면 되는 것입니다.
- 즉, 위에서 나온 방법을 적용하여, data 폴더에 있는 navigation.yml 파일에 새로운 title을 Categories로하고, 새로운 url을 해당 주소로 설정해주시면 됩니다.


### Github 블로그를 clone 하는 방법
- 일반적인 git 사용법과 동일하게 `clone`을 해주면 됩니다.
- git을 사용하는 방법은 [이곳](https://sadoe3.github.io/etc/chatper2/)에 정리하였습니다.
    - 필요하실때 참고하세요.
- 즉, terminal에서 진행해도 되고, VS Code에서 진행해도 됩니다.
- 만약 terminal을 이용한다면, `cd`를 이용하여 cloning될 repository가 들어갈 folder의 위치로 이동하고,
- 이후 `git clone url`을 이용하여 cloning 해주면 됩니다.
    - 여기서 url은 GitHub에 있는 repository에서 code button을 누르면 확인할 수 있습니다.
    - 보통 cloning을 한뒤, `git remote add origin url`을 이용하여, origin을 해당 url로 설정하여, 이후에는 `git clone origin`으로 진행합니다.


### GitHub 블로그를 수정한 뒤, 변경사항을 적용시키는 방법
- 일반적인 git 사용법과 동일하게 `push`을 해주면 됩니다.
- 즉, terminal에서 진행해도 되고, VS Code에서 진행해도 됩니다.
- 만약 terminal을 이용한다면, `cd`를 이용하여 cloning될 repository가 들어갈 folder의 위치로 이동하고,
- 이후 `git add *` 및 `git commit -am "commit message"`를 이용하여 `commit`시키고, 이후 `git push`를 하여 설정된 origin으로 `push`를 진행합니다.


### VS Code에서 clone 된 repository 제거 방법
- 일반적인 git 사용법과 동일하게, local address에 저장된 폴더를 지우면 됩니다. 


### clone 된 repository를 수정된 최신 repository의 변경상태로 sync 를 맞추는 방법
- 일반적인 git 사용법과 동일하게 `pull`을 해주면 됩니다.
- terminal에서 `cd`으로 clone된 folder의 위치로 가서, 해당 command를 사용해도 되지만,
- 이게 귀찮은 경우, 그냥 VS Code에서 clone된 folder가 열려있는 경우, VS Code menu에서 `Terminal -> New Terminal`을 누르게 되면, VS Code에서 clone된 해당 folder로 `cd`가 되어있는 terminal이 열리게 됩니다.
- 그래서 여기서 바로 `git pull`을 해도 됩니다.


### Github 블로그에 코드를 올리는 방법
- 인라인 코드는 backtick(\`)문자 1개로 감싸면 됩니다.
    - ex: It is the `Inline Code Block`
1. backtick(\`)문자 3개를 연속으로 쓴 후, 원하는 프로그래밍 언어를 적은 다음 다음 줄부터 원하는 코드를 복사 및 붙여넣기 합니다.
2. 다 붙여넣었으면, 마지막 코드 다음 줄에 backtick(\`)문자 3개를 연속으로 적음으로써 code block를 끝내시면 됩니다.
- 예) c++ code block <br>
\`\`\`c++ <br>
~~ // this is the sample code <br>
\`\`\` <br>
- 주의할점: code block안에 new line을 넣으려고 enter를 이용하면 code block이 적용되지 않게됩니다. (즉, code block에는 빈공간이 없어야함)


### 공백을 포스트에 올리는 방법
- pre 태그를 이용하면 모든 공백을 포스트 할 수 있다.
- ex) \<pre> ~ \</pre> <br>


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
- 즉, `pull`을 진행한 뒤, `push`를 해주면 쉽게 해결할 수 있다.
- terminal에서 다음과 같은 codes를 작성해주면 된다.
 ```
git pull --rebase
git push
 ```
3. 그렇게 되면, 오류가 발생했던 변경사항이 성공적으로 `push`가 된다.
