# 리액트스럽게 생각하기
리액트는 크고, 빠른 웹앱을 만드는 최고의 방법이라고 생각합니다. 리액트는 페이스북과 인스타그램에서 아주 잘 확장되어 왔습니다.

리액트의 많은 장점중 하나는, 리액트가 앱을 만들때, "어떻게 생각하게 만드는 가"입니다. 이번 문서에서는 리액트로 검색가능한 데이터 테이블을 만드는 과정의 생각을 쭉 살펴보겠습니다.

## 목업으로 시작하기

 JSON API와 디자이너에게 받은 목업을 이미 가지고 있다고 가정하겠습니다. 목업은 아래와 같이 생겼습니다.
 <img src="https://facebook.github.io/react/img/blog/thinking-in-react-mock.png" alt="Mockup">

 우리의 JSON API는 아래와 같은 data들을 반환합니다.
 ```
 [
  {category: "Sporting Goods", price: "$49.99", stocked: true, name: "Football"},
  {category: "Sporting Goods", price: "$9.99", stocked: true, name: "Baseball"},
  {category: "Sporting Goods", price: "$29.99", stocked: false, name: "Basketball"},
  {category: "Electronics", price: "$99.99", stocked: true, name: "iPod Touch"},
  {category: "Electronics", price: "$399.99", stocked: false, name: "iPhone 5"},
  {category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7"}
];
```

## 1 단계: UI를 component 계층구조로 분해하라
가장먼저 할 일은, 모든 component(subCompoenet 포함)주위에 상자를 그리고 이름을 붙이는 것 입니다. 디자이너와 함께 작업하는 경우라면, 그 분들이 이미 해놓았을 것이므로 그 분들께 이야기 하면 됩니다. 그들의 포토샵 레이어의 이름이 결국, 리액트 component의 이름이 될것 입니다!

하지만 이것이 자체 component(이름을 가진 독립적인 component)가 되어야하는지 어떻게 알 수 있을까요? 단순히 새 function이나 object를 만들지를 결정하는 기법을 사용하면 됩니다. 이 중 하나는 <a href="https://en.wikipedia.org/wiki/Single_responsibility_principle">single responsibility principle</a>입니다. 다시말해, 이상적으로는 한 omponent가  한가지 일만 합니다. 이것이 보장되지 않으면, 더 작은 subCompoenet로 분리해야 합니다.

종종 JSON data model을 사용자에게 보여주기 때문에, modele이 정확하다면, UI(component 구조)가 잘 매핑된다는 것을 느꼈을 것입니다. UI와 data modele이  <em>정보의 구조(architecture)</em>에 의존하는 경향이 있기때문입니다. 이는 UI를 component로 분리하는 것이 가끔은 별것 아니라는 것을 의미합니다. 단순히 data model에서 하나의 부분만 명확이 보여주도록 component를 분리하세요.

<img src="https://facebook.github.io/react/img/blog/thinking-in-react-components.png" alt="Component diagram">

이 간단한 앱에 5개의 component가 있다는 것을 볼수 있습니다. 각 component가 나타내는것을 이태릭체로 강조하였습니다.
1. ``` FilterableProductTable ```(orange): 예제 전체를 포함합니다.
2. ``` SearchBar ```(blue): <em>user input</em>을 받습니다.
3. ``` ProductTable ```(green): <em>user input</em>에 따라, <em>data collection</em>을 필터링하고 표시합니다.
4. ``` ProductCategoryRow ```(turquoise): heading한 <em>category</em>를 표시합니다.
5. ``` ProductRow ```(red): 각각의 <em>product</em>를 표시합니다.
``` ProductTable ```의 table header("Name"과 "Price"를 포함한)는 자체 component가 아닙니다. 이는 선호의 문제이고, 어느 방법이든 논쟁할 여지가 있습니다. 예를 들어, 여기서는 table header를 ``` ProductTable ```의 일부분으로 남겨두었습니다. ``` ProductTable ```의 역할인 <em>data collection</em>렌더링의 일부이기 때문입니다. 그러나, header가 복잡해 질 경우(ex. 정렬기능 제공), 자체 ``` ProductTableHeader  ``` component로 분리하는것이 확실히 더 좋습니다.

이제, 목업에서 component들을 분리하였습니다. 이제 이들을 계층구조로 배열해 보겠습니다. 무척 쉽습니다. 목업내부의 component에서 나타나는 component들은 계층구조의 child로 나타납니다.

- ``` FilterableProductTable ```
    - ``` SearchBar ```
    - ``` ProductTable ```
        - ``` ProductCategoryRow ```
        - ``` ProductRow ```

## 2 단계: 리액트로 정적 버전을 빌드해라

<iframe id="cp_embed_vXpAgj" src="//codepen.io/lacker/embed/vXpAgj?height=600&amp;theme-id=0&amp;slug-hash=vXpAgj&amp;default-tab=js&amp;user=lacker&amp;embed-version=2" scrolling="no" frameborder="0" height="600" allowtransparency="true" allowfullscreen="true" name="CodePen Embed" title="CodePen Embed 2" class="cp_embed_iframe " style="width: 100%; overflow: hidden;"></iframe>
