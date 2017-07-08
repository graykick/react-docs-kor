# Components and Props
Components는 UI를 독립적이고, 재사용 가능하고, 각각의 부분을 개별적으로 생각할 수 있게 해줍니다.

개념적으로, commponente들은 JavaScript의 function과 같습니다. 이는 임의의 input(props)을 받고, 스크린에 보이는 것을 설명한 리액트 element를 리턴합니다.

## Class Components와 Functional
commponente를 정의하는 가장 쉬운 방법은 JavaScript function을 사용하는 것 입니다.
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```
이 function은 유효한 리액트 commponente입니다. 하나의 "props" object argument를 data로 받아들이고, 리액트 element를 리턴하기 때문입니다. 이들은 글자대로 JavaScript function이기 때문에 이것을 "Functional"하다고 합니다.

ES6 class로 commponent를 정의할수도 있습니다.
```
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```
위의 두 components는 리액트의 관점에서 같습니다.

Class는 몇가지 부가적인 특징들을 가지고 있습니다. 다음 섹션에서는 이 특징들에 대해서 다룰것 입니다. 그 전까지는 간결함을 위해 Functional components를 사용할 것 입니다.

## Component 렌더링
전에 리액트 elements를 나타내는 DOM tgas를 보았습니다.
```
const element = <div />;
```
그러나, elements는 사용자 정의 components로 나태낼 수도 있습니다.
```
const element = <Welcome name="Sara" />;
```
리액트가 element를 나타내는 볼 때, JSX attributes를 이 component에 하나의 object로써 전달 합니다. 이것을 "props"라고 부릅니다.

예를들어, 이 코드는 페이지에 "Hello, Sara"를 렌더링 합니다.
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```
<a href="http://codepen.io/gaearon/pen/YGYmEG?editors=0010">Try it on CodePen.</a>

이 예제에서 일어난 일을 요약해 보겠습니다.

1. ``` ReactDOM.render() ``` 를 ``` <Welcome name="Sara" /> ``` 와 함께 호출합니다.
2. 리액트는 ``` Welcome ``` component를 ``` {name: 'Sora'} ``` props와 함께 호출합니다.
3. ``` Welcome ``` component는 ``` <h1>Hello, Sara</h1> ```를 리턴합니다.
4. 리액트 DOM은 ``` <h1>Hello, Sara</h1> ```와 매치되도록 DOM을 업데이트 합니다.

>항상 component의 이름은 대문자로 시작하세요. 예를 들어 ``` <div /> ```은 DOM tag를 나타내지만, ``` <Welcome /> ``` 은 component를 나타내며 ``` Welcom ``` 이 scope에 있어야 합니다.

## Components 만들기
Components의 결과물은 다른 Components를 나타낼 수 있습니다. 이 덕분에 그 어떤 상세한 수준에도 같은 Components 추상화를 사용할 수 있습니다. 리액트에서 button, form, dialog, screen은 일반적인 Components로 표현됩니다.

예를 들어, ``` Welcome ```을 여러번 렌더링 하는 ``` App ``` component를 생성할 수 있습니다.
```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```
<a href="http://codepen.io/gaearon/pen/KgQKPr?editors=0010">Try it on CodePen.</a>

일반적으로, 리액트 앱은 최상단에 하나의 ``` app ``` component를 가집니다. 다만, 만일 기존의 앱에 통합한다면, ``` Button ``` 같은 작은 component부터 차즘, 상위 view 계층으로 이동할 수 있습니다.

> Components는 반드시 단일 루트 element를 리턴해야 합니다. 이것이 ``` <div> ``` container에 모든 ``` <Welcome /> ``` element를 추가한 이유입니다.

## Components 뽑아내기
Components를 작은 cmponents들로 분리하는것을 두려워하지 마세요.
예를 들어, 아래의 ``` Comment ``` component를 생각해 보세요.
```
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
<a href="http://codepen.io/gaearon/pen/VKQwEo?editors=0010">Try it on CodePen.</a>

이 component는 props로 ``` author ```(object), ``` text ```(string), ``` date ```(date)를 받아들입니다. 그리곤 comment를 소셜미디어 웹사이트에 표시합니다.

이 comment는 중첩때문에 변경하기 따다롭고,개별적인 부분으로 재사용하기도 힙듭니다. 여기서 몇개의 components를 뽑아내 봅시다.

먼저, ``` Avatar ```를 뽑아내겠습니다.
```
function Avatar(props) {
  return (
    <img className="Avatar"
      src={props.user.avatarUrl}
      alt={props.user.name}
    />
  );
}
```
``` Avaster ```는 ```Commnet```에서 렌더링 되었는지 아닌지 알 필요가 없습니다. 이것이 props의 이름로, ```author```를 사용하지 않고, 더 일반적인 ```user```를 사용한 이유입니다.

component의 prop이름을 네이밍할때, 그 component가 사용되는 시점보다는, 그 컴포넌트의 관점에서 생각하기를 권합니다.

``` Comment ```를 조금이나마 간소화 하였습니다.
```
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <Avatar user={props.author} />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
다음은, user의 name 옆에 ``` Avatar ```를 렌더링하는 ``` UserInfo``` component를 뽑아내 보겠습니다.
```
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">
        {props.user.name}
      </div>
    </div>
  );
}
```
``` Comment ```를 더 간소화 하였습니다.
```
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
<a href="http://codepen.io/gaearon/pen/rrJNJY?editors=0010">Try it on CodePen.</a>

commponente를 뽑아내는 일은 처음에는 쓸데 없는 일 처럼 보일수 있습니다. 하지만, 재사용 가능한 components 팔레트는 큰 규모의 app에서 성과를 거 둘 수 있습니다. 경험적으로, 반복해서 사용되는 UI의 일부분(Button, Panel, Avatar)이나 그 자체로 복잡한 component(App, FeedStory, Comment)는 재사용 가능한 component의 좋은 후보입니다.

## Props는 read-only(읽기 전용)이다.
함수형(Functional) component와 class형 component모두, 절대로 props를 스스로 변경하면 안됩니다.

아래의 ``` sum ``` function을 봅시다.
```
function sum(a, b) {
  return a + b;
}
```
위와같은 function들을 ** "pure"(순수) ** 하다고 합니다. function의 input을 변경하지 않고 동일한 input일 경우, 항상 동일한 값을 리턴하기 때문입니다.

반대로 아래의 function은 impure(순수하지 않은)합니다. function의 input을 변경하기 때문입니다.
```
function withdraw(account, amount) {
  account.total -= amount;
}
```
리액트는 유연합니다. 그치만, 한가지 엄격한 규칙이 있습니다.

** "모든 리액트 component들은 props와 관련해선 pure function처럼 동작해야 합니다." **

물론, 앱의 UI는 시간이 지남에 따라 동적으로 변합니다. 다음 섹션에서는 새로운 컨셉의 "state"를 소개합니다. State는 시간이 지남에 따라, user action이나 network response이나 어떤것들에 반응해서 리액트 component의 결과가 바뀌는것을 허용합니다. 이 규칙을 위배하지 않고 말이죠.
