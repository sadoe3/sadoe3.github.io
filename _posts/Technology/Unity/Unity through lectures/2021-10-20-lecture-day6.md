---
title: "Unity 강의 - Day 6"

categories:
    - unity

tags:
    - [Game Engine, Unity, Lecture]

toc: true
toc_label: "목차"
toc_sticky: true

date: 2021-10-20 
---

전주정보문화산업진흥원에서 진행하는 **게임콘텐츠(Unity3D) 제작 실무자 양성과정** 강의를 듣고 정리한 필기입니다.
{: .notice--warning}

## Animation in Unity
- Unity에서 애니메이션을 구현할 때에는 크게 2가지가 필요합니다.
    1. Animation clips: 이 파일에는 실제 애니메이션에 필요한 sprite등의 데이터들이 들어있습니다.
    - 또한 이 파일에서 해당 애니메이션의 전반적인 설정도 변경할 수 있습니다.
    2. Animator controller: 이것은 어떤 상황에서 어떤 방식으로 Animation clips를 보여주는지를 설정해줍니다.
- Animation clips와 Animator controller가 적용된 object를 한번에 만드는 방법
    - Assets window에서 애니메이션으로 만들 sprite들을 모두 선택하기 -> 해당 파일들을 hierarchy window로 drag -> 이렇게 하면, 자동적으로 animation clips와 animator controller를 갖고있는 새로운 오브젝트가 만들어짐
    - 이때, 해당 animation clips와 animator controller는 assets folder에 자동적으로 만들어지게 됩니다.
- Animation clips 파일의 inspector window에서 해당  다양한 설정을 해줄 수 있음
    - 대표적으로 loop time을 설정해줘야, animation이 반복 실행됨
- Animation clips 파일의 애니메이션을 볼 때에는 해당 animation 파일을 선택하면, inspector window에서 preview를 볼 수 있다.
    - 만약에 animation preview가 inspector window에서 사라지게 되면, inspector window의 우측 하단에 있는 점 3개 있는 버튼을 눌러 restore in Inspector를 선택하면 다시 animation preview를 불러올 수 있다.
- 애니메이션 세부 설정 -> 해당 animation 선택 -> ctrl + 6
    - 이후, 나오게되는 window에서 필요없는 animation event를 지우고, 필요한 animation event를 적절히 추가해 주면 됨


## How to use custom Animator
- Assets folder -> right click -> create -> animator controller를 통해 animator 만들기
- 이후, 해당 animator를 더블클릭 하여 animator window를 띄우고, animation clips 파일을 drag하여 해당 animation clips를 animator에 추가할 수 있음
- animator에서 다양한 설정을 통해, animation이 어떤식으로 보여지는지 설정할 수 있음
    - speed값을 올려주면, 좀더 빨리 animation을 보여줄 수 있음

## 특정 오브젝트에 애니메이션 추가하는 방법
1. 특정 오브젝트 선택 -> inspector window
2. sprite renderer component 추가하기
3. 처음 출력될 sprite를 해당 component에 drag & drop 하기
4. animator component 추가하기
5. 알맞은 animator를 해당 component에 drag & drop 하기


## Layer in Unity
- layer는 포토샵 layer와 같은 개념으로, 게임 내의 가상의 층을 말한다.
- 이를 통해 특정 layer만 raycast가 되도록하거나, camera에 보이도록 하는 식 등으로 사용할 수 있음
- Layer는 unity의 우측 상단에서 설정할 수 있다.
- 특정 object의 layer를 설정하는 것도 해당 object의 inspector window의 우측 상단에서 확인할 수 있음

## Sorting Layer
- hierarchy의 위치를 기반으로 rendering order를 설정할 수도 있지만, 세부적인 rendering order를 설정하기 위해선 다음과 같은 설정을 해야한다.
- sprite renderer 같은 renderer component의 sorting layer값을 통해 renderering order를 설정할 수 있다.
- 만약 sorting layer의 값이 같다면, order in layer값으로 세부적인 order를 설정할 수 있다.
- sorting layer는 layer를 추가하는 window와 같은 window를 사용한다.
    - 그래서 sorting layer를 추가할 때에는 layer를 추가하는 방법과 동일하게 진행하면 된다.

## How to handle click event of buttons
1. click를 처리할 button object 선택 -> inspector window
2. \+ symbol click
3. click event를 처리할 method가 들어있는 script 파일이 적용된 object를 추가된 공간에 drag & drop
    - 그래서 보통 UIManager script가 적용된 UIManager object를 넣게됨
4. 이후 UIManager -> 알맞는 method 선택
    - 만약 특정 method가 enum를 parameter로 갖는 경우, 해당 method는 editor에서 뜨지 않아 선택할 수 없게 된다.
    - 이럴 경우에 대한 해결방법은 다양하게 있으니, 인터넷으로 검색하면 쉽게 해결할 수 있다.
5. 해당 method가 call이 될때 pass할 argument들 적절하게 입력
- 이 방식은 editor에서 event listener를 추가하는 방식인데, event listener를 script에서 동적으로 추가하는 방식도 존재한다.

## How to handle mouse input
- Unity에서 mouse input에 대한 처리는 보통 Raycast를 통해 진행된다.
- 즉, Unity에서 특정 오브젝트를 클릭하거나 선택하는 처리는 raycast로 구현할 수 있다는 의미이다.
- Raycast는 쉽게 말하면, 가상의 광선을 쏘는 행위를 담당하는 method이다.
- Unity에서는 mouse 버튼이 눌렸을때, 커서의 위치에서 scene쪽으로 가상의 광선을 쏘아 mosue input에 대한 event를 처리하게 된다.
- 아래는 마우스 좌클릭이 되었을 때, LayerName에 있는 object들을 대상으로, 마우스 커서의 위치에서 scene쪽으로 무한한 광선을 쏘고, 해당 광선이 무언가에 hit이 된 경우, hit이 된 오브젝트의 ClassName component를 얻고, 해당 component를 기반으로 임의의 event handling을 하는 코드입니다.
- 해당 component가 ClassName이라는 class의 instance이기 때문에, 해당 class의 특정 method를 call하는 식으로 event handling이 진행된다고 생각하시면 됩니다.
```c#
// example of handling mosue input by using raycast
// if GetMouseButtonDown(0) returns true, then it means that left mouse is just pressed down 
if(Input.GetMouseButtonDown(0)){
    RaycastHit hit;
    Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
    if(Physics.Raycast(ray, out hit, Mathf.Infinity, LayerMask.GetMask("LayerName"))){
        ClassName objectName = hit.transform.GetComponent<ClassName>();
        if(objectName != null)
            ... // some codes for handling event
    }   
}
```
- 만약에 특정 layer만 제외하고 싶은 경우에는 "~LayerName"으로 argument를 pass해주시면 됩니다.
- 또한 Raycast는 최초로 hit된 물체에 대한 정보를 제공하지만, RaycastAll이라는 method는 광선이 물체들을 뚫고 나아갔을때 만나는 모든 물체들의 정보를 배열로 제공합니다.
- 주의할 점은 raycast는 collider component가 있는 object만 확인한다는 점입니다.
    - 3d collider를 사용하면 Physics에 있는 raycast method를 그냥 사용하면 되고, 2d collider를 사용하면 Physics2D에 있는 raycast method를 사용해야 한다.
- 2D 처리는 다음과 같이 할 수 있습니다.
```c#
// example of handling mosue input by using raycast in 2D world
if(Input.GetMouseButtonDown(0)) {
    Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
    RaycastHit2D hit = Physics2D.Raycast(ray.origin, ray.direction, Mathf.Infinity, LayerMask.GetMask("LayerName"));
    if(hit && hit.transform == tranfrom) {
        // if this code block is processed, this means that the hit has happened and the object which was hit by the ray is this object
        ... // some codes
    }
}
```
- 2D처리를 하다보면 가끔씩 objectA를 click했는데 objectB가 click되는 경우가 발생한다.
- 이 현상의 원인은 Collider가 너무 커서 그런 것인데, 큰 collider를 사용하면서, 이 현상을 해결하고 싶은 경우 다음과 같은 방식으로 해결할 수 있다.
1. 해당 object에 child object를 만들고, child object에 적절한 collider를 만든뒤, size와 offset을 알맞게 지정한다.
2. 해당 child object의 layer를 기존 object의 layer와 다르게 설정한뒤, raycast를 할때, child object의 layer로 진행해 주면된다.
3. 이후, script에서 다시 child object의 parent object 즉, 기존 object의 component들을 접근할 때에는 hit object의 collider에서 GetComponentInParent method를 사용해주면 된다.
```c#
// exmaple
if(Input.GetMouseButtonDown(0)) {
    Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
    RaycastHit2D hit = Physics2D.Raycast(ray.origin, ray.direction, Mathf.Infinity, LayerMask.GetMask("Popup"));
    if(hit)
        hit.collider.GetComponentInParent<ParentScriptName>().OnClickOpenRemovePopup();
}
``` 

## Resources folder (10월 4주차 수요일 수업내용 - prefab 정리)
- Unity에 있는 Assets window에 Resources folder를 만들 수가 있느

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}