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
<a href="codepen.io/lacker/embed/vXpAgj?height=600&amp;theme-id=0&amp;slug-hash=vXpAgj&amp;default-tab=js&amp;user=lacker&amp;embed-version=2">Try it on CodePen.</a>

이제, component의 계층구조가 생겼습니다. 이제는 앱은 구현할 차례입니다. 가장 간단한 방법은 data model을 취하지만, 통신은 하지 않는 version을 빌드하는것 입니다. 정적 version을 빌드하는 것은 많은 많이 생각할 필요는 없지만, 많은 타이핑이 필요한 반면, 통신을 하는 version을 빌드하는 것은 많은 생각이 필요지만, 타이핑은 많이 필요하지 않아서, 이 두 version을 분리하는 것이 가장 좋은 방법입니다. 이제 이러한 이유를 살펴보겠습니다.

원하는 data model을 렌더링 하는 앱을 빌드하기 위해서는 다른 component를 재사용하고 <em>props</em>로 data를 전달하는 component를 만들어야 합니다. <em>props</em>는 부모가 자식에게 data를 전달하는 방법입니다. <em>state</em>의 개념에 익숙하다면, 정적 version을 빌드하는데에는 ** state가 전혀 사용하지 마세요 **. State는 통신 version에서만 예약됩니다. 즉, 시간에 따라 data가 변경됩니다. 때문에, 정적 version에서는 state가 필요하지 않습니다.

하향식(top-down)이나 상향식(bottom-up)으로 앱을 빌드할 수 있습니다. 즉, 상위 계층의 component 부터 만들기 시작하거나(ex. ``` FilterableProductTable ```) 낮은 계층의 component(``` ProductRow ```)부터 시작 할 수 있습니다. 간단한 경우에는 하향식이 더 쉽고 크고 복잡한 경우에는 상향식으로 작업하면서 테스트를 작성하는 것이 더 쉽습니다.

이번 단계를 마치면, data model을 렌더링하는 재사용 가능한 component 라이브러리를 같데 될것 입니다. 정적 version의 앱이기 때문에 이 component는 ``` render() ``` method만을 가집니다. 최상위 component(``` FilterableProductTable ```)는 data model을 props으로 받습니다. 만약 data model을 변경하고 ``` ReactDOM.render() ```를 다시 호출하면, UI는 업데이트 됩니다. 복잡한 일이 없기 때문에, UI가 어떻게 업데이트 되고, 언제 변경해야 하는지 알기 쉽습니다. 리액트의 ** 단방향 data flow **는 모든 것을 모듈화 하고 빠르게 만듭니다.

이 실행 단계에 대해 더 알고 싶다면, <a href="/react/docs/">React docs</a>를 참고하세요.

### 막간 설명: Props vs State
리액트에는 두 가지의 "model" data가 있습니다. props와 state입니다. 이 둘의 차이와 특징을 이해하는 것은 아주 중요합니다. 이를 확실히 알지 못한다면, <a href="/react/docs/interactivity-and-dynamic-uis.html">리액트 공식문서</a>를 훑어 보세요.

## 3 단계: UI표시에 최소한의 state를 확인해라 (Identify The Minimal (but complete) Representation Of UI State)
UI를 인터렉티브하게 만드려면, 기본 data model의 변경을 트리거 할 수 있어야 합니다. 리액트는 이를 state를 사용해, 쉽게 해결합니다.

앱을 올바르게 만드려면, 먼저 앱에 필요한 변경하능한 state를 최소화 하는것을 생각해야 합니다. 중요한 점은 DRY: <em>Don't Repeat Yourself</em>입니다. 앱에 필요한 state의 표시를 절대적으로 최소화 하고, 다른 모든것은 필요에 따라 계산해야 합니다. 예를 들어, 만약 TODO list를 만든다면 TODO item들을 array로 가지고 계시고, 갯수를 별도의 state변수에 저장하지 마세요. 대신에 TODO의 갯수를 렌더링 할때는 간단히, TODO item array에서 length를 얻으세요.

이 예제 앱의 모든 data 조각들을 생각해 보세요. 이제 아래의 것들을 가지고 있습니다.
 - product의 원본 리스트
 - 유저가 입력한 검색 text
 - checkbox의 값
 - 여과된 product 리스트
이제 위의 data 하나하나를 살펴보고 어느것이 state인지 알아봅시다. 각각의 data에 대해서 3가지 간단한 질문에 답해보면 됩니다.
1. 부모의 props에서 전달되는가? 그렇다면, state가 아니다.
2. 시간이 지나도 변하지 않는가? 그렇다면, state가 아니다.
3. 해당 component의 props이나 state를 연산하여 얻을 수 있는 값인가? 그렇다면, state가 아니다.
원본 product 리스트는 props로 전달됩니다. 때문에 state가 아닙니다. 검색 text와 checkbox 값은 시간에따라 변하고, 다른 값으로 부터 얻을 수도 없기 때문에 state입니다. 마지막으로 여과된 product 리스트는 state가 아닙니다. 원본 product 리스트와 검색 text, checkbox 값을 조합하여 얻을 수 있기 때문입니다.

따라서, state는 아래와 같습니다.
  - 유저가 입력한 검색 text
  - checkbox의 값

## 4 단계: State가 어디에 존재해야 하는지 확인해라
<a href="codepen.io/lacker/embed/ORzEkG?height=600&amp;theme-id=0&amp;slug-hash=ORzEkG&amp;default-tab=js&amp;user=lacker&amp;embed-version=2">Try it on CodePen.</a>

좋습니다. 이제 최소한의 app state를 알아냈습니다. 다음은 어떤 component가 이 state를 변경하고 소유할지 알아보아야 합니다.

기억하세요: 리액트는 항상 한 방향으로 계층구조를 따라 아래로 흐릅니다. 어떤 component가 어떤 state를 소유해야 하는지, 바로 명확히 보이지 않을 수 있습니다. ** 종종 이는 초심자가 이해하는데 있어서 가장 큰 산 **이므로, 아래의 단계에 따라 알아보겠습니다.

앱의 state들 입니다.
  - state에따라 렌더링되는 모든 component들을 판별하세요.
  - 공통 소유(common owner) component를 찾으세요.(계층구조에서 state가 필요한 모든 component위의 single component)
  - common owner나 더 높은 계층의 다른 component는 state를 소유해야 합니다.
  - 어디서 state를 소유해야 할지 찾지 못했다면, state를 소유하는 간단한 component를 만든뒤, 이를 계층구조의 공통 소유 component위 어딘가에 추가하세요.

이 방법을 통해 우리의 앱을 돌려봅시다.
  - ``` ProductTable ```는 state에 따라 product를 여과해야 하고, ``` SearchBar ```는 검색 text와 check state를 표시해야 합니다.
  - 공통 소유 component는 ``` FilterableProductTable ```입니다.
  - 이는 개념적으로 filter text와 checked value는 ``` FilterableProductTable ```에 존재함을 의미합니다.

  좋습니다. 우리의 state는 ``` FilterableProductTable ```에 존재한다고 결정했습니다. 먼저, 앱에 초기 state를 반영하기 위해, instance property ``` this.state = {filterText: '', inStockOnly: false} ``` to ``` FilterableProductTable ```의 ``` constructor ```에 추가 하세요. 그런 뒤, ``` filterText ```와 ``` inStockOnly ```를 ``` ProductTable ```에 props로 전달하세요. 마지막으로 ``` ProductTable ```에서 props를 사용해서 row들을 여과하고  ``` SearchBar ```에서 form field의 값을 설정하세요.

  이제 앱이 어떻게 동작할지 볼 수 있습니다. ``` filterText ```응 ``` "ball" ```로 설정하고 앱을 리프레시해보세요. 정상적으로 업데이트된 table이 나올것 입니다.

## 5 단계: 역방향 data flow 추가하기
<a href="codepen.io/lacker/embed/ORzEkG?height=600&theme-id=0&slug-hash=ORzEkG&default-tab=js&user=lacker&embed-version=2">Try it on CodePen.</a>

지금까지, 계층구조를 따라 props와 state의 함수처럼 올바르게 렌더링 하는 앱을 만들었습니다. 이제 다른 방식의 데이터 흐름을 지원해 보겠습니다. 계층구조 깊숙한 곳의 form component가 ``` FilterableProductTable ```의 state를 업데이트 해야 합니다.

리액트는 프로그램이 어떻게 동작하는지 이해하기 쉽도록 데이터 흐름을 명확하게 만듭니다. 하지만, 기존의 양방향 바인딩 보다는 많은 타이핑이 필요합니다.

만약 현재 버전의 예제이서는 타이핑을 하거나, check box를 조작해도 리액트가 이를 무시한다는 것을 알수 있습니다. 이는 의도적인 것으로, ``` input ```의 ``` value ```를 ``` FilterableProductTable ```에서 전달된 ``` state ````와 항상 같도록 하였기 때문입니다.

우리가 원하는 것이 무엇인지 생각해 봅시다. 우리는 언제든 유저가 form을 변경하면, user input을 반영하여 state를 업데이트 하기를 원합니다. component 자신만이 state를 업데이트 할 수 있기때문에, ``` FilterableProductTable ```은 ``` SearchBar ```에 state가 업데이트 되어야 할때 실행될 callback을 전달합니다. 변경을 알리기 위해, input에 ``` onChange ```이벤트를 사용할 수 있습니다. ``` FilterableProductTable ```에서 전달된 callbac은 ``` setState() ```를 호출하고, 앱이 업데이트 되게 됩니다.

이것이 복잡해 보일 수 있지만, 단지 몇줄의 코드일 뿐입니다. 또한 앱 곳곳에서 데이터가 어떻게 흐르는지가 매우 명확합니다.

## 이것이 전부다
다행이도, 이를 통해 리액트로 어플리케이션과 component를 만드는 방법에 대해 생각해 볼 수 있습니다. 전 보다 타이핑을 덜 하더라도, 코드는 작성한 시간보다 더 많이 읽혀지고 모듈화 되고 명확한 코드가 읽기 쉽다는 것을 을 기억하세요. 큰 규모의 component 라이브러리를 만들기 시작했다면, 모듈화와 명확성에 감사하게 되고,  코드는 재사용 되고, 여러분의 코드는 줄어들기 시작할 것 입니다. :)
