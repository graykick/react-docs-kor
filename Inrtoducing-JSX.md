# Introducing JSX
이 변수선언에 대해 생각해 봅시다.
```
const element = <h1>Hello, world!</h1>;
```
이 웃긴 tag 문법은 string도 아니며 HTML도 아닙니다.

이것은 JavaScript 문법을 확장한 JSX입니다. UI가 어떻게 생겼는지 나타내기 위해서 리액트와 함께 JSX를 사용하는 것을 추천합니다. JSX는 템플릿 언어를 떠올리게 하지만 JSX는 JavaScript의 모든 것이 딸려옵니다.

JSX는 리액트 "elements"를 만듭니다. 다음 섹션에서는 이것을 DOM에 렌더링하는 것을 다룹니다. 아래의 내용에서 JSX를 시작하는데 필요한 기본적인 사항들을 알 수 있습니다.  

## Embedding Expressions in JSX
JavaScript 표현식을 중괄호({})로 감싸므로써, JSX에 삽입할 수 있습니다.

예를 들어, ``` 2+2 ```, ``` user.firstName ```, 과 ``` formatName(user) ``` 는 모두 사용 가능한 표현식 입니다.
```
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = (
  <h1>
    Hello, {formatName(user)}!
  </h1>
);

ReactDOM.render(
  element,
  document.getElementById('root')
);
```
<a href="http://codepen.io/gaearon/pen/PGEjdG?editors=0010">Try it on CodePen.</a>

JSX는 JSX를 가독성을 위해 여러 lines으로 나누었습니다. 필수사항은 아니지만, 이 작업을 할 때는 뜻하지 않은 자동 세미콜론 삽입을 피하기 위해, 괄호(())로 감싸는 것을 추천합니다.

## JSX도 표현식이다.
컴파일 후에 JSX는 정상적인 JavaScript Object가 됩니다.

이것은 JSX안에 ``` if ``` 구문이나 ``` for ``` 루프를 사용하고 변수를 할당하고 인자를 받아들여 function으로 return할 수 있다는 것을 의미합니다.
```
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

## JSX attributes 지정
string literals을 이용해서 attributes속성을 지정할 수 있습니다.
```
const element = <div tabIndex="0"></div>;
```
중괄호를 이용해서 JavaScript 표현식을 attribute에 포함시킬 수도 있습니다.
```
const element = <img src={user.avatarUrl}></img>;
```
JavaScript 표현식을 삽입할 때, 중괄호를 따옴표로 둘러싸지 마세요. 그렇지 않으면, JSX는 attribute를 JavaScript 표현식이 아닌, string literal로 취급합니다. attribute를 지정하기 위해, 따옴표나 중괄호를 사용해야 하지만 만드시 둘중 하나만 사용해야합니다.

## JSX children 지정
빈 tag는 ``` /> ```를 사용하여 즉시 닫을 수 있습니다.
```
const element = <img src={user.avatarUrl} />;
```
JSX는 children을 포함할 수 있습니다.
```
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```
>JSX는 HTML보다 JavaScript에 더 가깝기 때문에, HTML attribute 이름대신에 리액트 DOM은 property이름에 ``` camelCase ```를 사용합니다.
예를 들어 JSX에서는 ``` class ```는 ``` className ``` 이고 ``` tabindex ```는 ``` tabIndex ``` 입니다.

## Injection 공격을 막는 JSX
JSX에서 아래의 유저 input은 안전합니다.
```
const title = response.potentiallyMaliciousInput;
// This is safe:
const element = <h1>{title}</h1>;
```
기본적으로, 리액트 DOM은 렌더링 전에 JSX에 삽입된 value들을 escape 시킵니다. 그래서 명확하게 작성되지 않은 것은 앱에 삽입될 수 없습니다.
모든 것은 렌더링 되기 전에 string으로 변환됩니다. 따라서 XSS 공격을 막는데 유용합니다.

## JSX는 Objects를 나타낸다
Babel은 ``` React.createElement() ```를 호출해서 JSX를 컴파일 한다. 아래의 두 예제는 동일하다.
```
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```
```
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```
``` React.createElement() ```는 버그없는 코드를 작성하기 위해 몇 가지 체크를 합니다. 그러나 본질적으로 아래와 같은 object를 만듭니다.
```
// Note: this structure is simplified
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world'
  }
};
```
이것들이 "React elements"입니다. 이것들은 화면에서 보고 싶은 것에대한 명세서로 생각할 수 있습니다. 리액트는 이 object 들을 읽고 이를 이용해서 DOM을 생성하고 최신상태로 유지합니다. 
