# Element 렌더링

Element는 React앱의 최소 구성요소 입니다. <br>
Element는 화면에 표시할 내용을 기술합니다.
```
const element = <h1>Hello, world</h1>;
```
브라우저의 DOM element와는 달리, 리액트의 Elements는 일반 객체이고 생성비용(create cost)이 적습니다. 또한, 리액트 DOM은 리액트 Element와 일치하도록 DOM을 업데이트 합니다.

>어떤 사람들은 Element를 더 유명한 "component"의 컨셉과 혼동할 수 있습니다. 다음 섹션에서 components를 설명합니다. Elements는  component의 구성요소이기 때문에 이 섹션을 건너뛰지 말것을 권합니다.

## Element를 DOM에 렌더링하기
``` div ``` 가 HTML file 어딘가에 있다고 합시다. :
```
<div id="root"></div>
```
내부의 모든것이 리액트 DOM에 의해 관리되기 때문에, 이것을 "root" DOM node라고 부릅니다.

리액트로 빌드된 어플리케이션은 보통 하나의 root DOM node를 가집니다. 만약 기존의 앱에 리액트를 통합한다면 원하는 만큼의 독립된 root DOM node들이 있을 수 있습니다.

리액트 Element를 root DOM node에 렌더링하기 위해서, 2가지를 ``` ReactDOM.render() ``` 에 전달해야 합니다.
```
const element = <h1>Hello, world</h1>;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```
<a href="http://codepen.io/gaearon/pen/rrpgNB?editors=1010">Try it on CodePen.</a>

"Hello World"를 페이지에 표시합니다.

## 렌더링된 Element를 업데이트하기
리액트 elements는 불변합니다. 때문에 생성된 element의 children이나 attributes를 변경 할 수 없습니다.
Element는 영화의 한 장면과 같습니다(한 순간의 UI를 나타냅니다)

지금까지 지식으로는, UI를 업데이트할 방법은 새로운 element를 생성한 다음 2가지를 ``` ReactDOM.render() ``` 에 전달하는것 입니다.

움직이는 시계의 example을 잘 생각해 보세요.
```
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```
<a href="http://codepen.io/gaearon/pen/gwoJZk?editors=0010">Try it on CodePen.</a>

``` setInterval() ```의 callback에서 매 초마다 ``` ReactDOM.render() ```를 호출합니다.

>실제로, 대부분의 리액트 앱들은 ``` ReactDOM.render() ```을 한 번만 호출합니다. 이 다음 섹션에서는 이 코드가 어떻게 **stateful components**로 캡슐화 되는지 배웁니다. <br> 다양한 토픽 서로가 서로를 기반 지식으로하기 때문에, 토픽은 생략하지 않는 것을 권합니다.

## 리액트는 필요한 것만 렌더링 한다
리액트 DOM은 element와 children을 전의 것과 비교하고 DOM을 원하는 상태로 만드는데 필요한 DOM 만을 업데이트 합니다.

브라우저 도구를 사용해서 <a href="http://codepen.io/gaearon/pen/gwoJZk?editors=0010">last example</a> 를 검사하면 확인할 수 있습니다. <br>
<img src="https://facebook.github.io/react/img/docs/granular-dom-updates.gif"> <br>
매 초마다 전체 UI 트리를 그리는 element를 만들었지만, 변경된 text node만 ReactDOM에 의해 업데이트 됩니다.

우리의 경험에서, UI가 주어진 시간에 어떻게 보여야 하는지 생각하는 것은 시간이 지남에 따라 어떻게 변하는지 생각하는 것 보다, 모든 버그가 제거됩니다.
