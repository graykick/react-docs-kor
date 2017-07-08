# Handling Events
리액트 element에서 event를 handling하는 것은 DOM에서 event를 handling하는것과 아주 비슷합니다. 여기에 문법적으로 다른점이 몇 개 있습니다.

- 리액트 event는 lowercase보다는 camelCase를 통해 네이밍된다.
- JSX에서는 event handler로 string이 아닌, function를 전달한다.

HTML의 예시
```
<button onclick="activateLasers()">
  Activate Lasers
</button>
```
리액트에서는 조금 다릅니다.
```
<button onClick={activateLasers}>
  Activate Lasers
</button>
```
다른 차이는 default behavior를 막기위해, false를 리턴할 수 없다는 것 입니다. 이를 위해서는 ``` preventDefault ```를 명시적으로 호출해야 합니다. 예를 들어, 순수 HTML에서, 새 페이지를 열때 기본 링크동작을 막기 위해서 아래와 같이 할 수 있습니다.
```
<a href="#" onclick="console.log('The link was clicked.'); return false">
  Click me
</a>
```
리액트에서는, 이렇게 합니다.
```
function ActionLink() {
 function handleClick(e) {
   e.preventDefault();
   console.log('The link was clicked.');
 }

 return (
   <a href="#" onClick={handleClick}>
     Click me
   </a>
 );
}
```
여기서 ``` e ```는 synthetic event입니다. 리액트는 이 synthetic event들을 <a href="https://www.w3.org/TR/DOM-Level-3-Events/">W3C spec</a>에 따라서 정의했기 때문에, 브라우저 호환성에 대해 걱정할 필요가 없습니다. 더 많은 정보는 <a href="/react/docs/events.html">``` SyntheticEvent ```</a>레퍼런스 가이드를 참고하세요.  

보통 리액트를 사용하는 경우에는 이미 생성된 DOM에 event를 추가하기위해, ``` addEventListener ```를 호출할 필요가 없습니다. 대신에, element가 처음 렌더링될때, listener를 제공해주면 됩니다.

<a href="https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes">ES6 class</a>를 사용해서 component를 정의할때, 흔한 패턴은 event handler를 class의 method로 두는것 입니다. 예를 들어, 이 ``` Toggle ``` component는 유저가 "ON"과 "USER"상태를 전활할 수 있게하는 버튼을 렌더링합니다.
```
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(prevState => ({
      isToggleOn: !prevState.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```
<a href="http://codepen.io/gaearon/pen/xEmzGg?editors=0010">Try it on CodePen.</a>

JSX callback에서 ``` this ```의 의미에 주의해야 합니다. JavaScript에서 class method는 기본적으로 bound되지 않습니다. 만일 ``` this.handleClick ```을 bind하지않고, ``` onClick ```을 전달하면 function(method)이 실제로 호출될을 때, ``` this ```은 undefined일 것입니다.

이것은 리액트만의 특별한 행동이 아닙니다. 이것은 <a href="https://www.smashingmagazine.com/2014/01/understanding-javascript-function-prototype-bind/">JavaScript function이 동작하는 방법</a>의 일부입니다. 일반적으로, method를 ``` () ```없이 조회한더면(``` onClick={this.handleClick} ``` 같이), 반드시 method를 bind해야 합니다.

만약 ``` bind ```을 호출하는 것이 귀찮다면, 이를 해결할 두가지 방법이 있습니다. 만약 experimental <a href="https://babeljs.io/docs/plugins/transform-class-properties/">property initializer syntax</a>를 사용한다면, property initializers를 이용하여 callback을 올바르게 bind할 수 있습니다.
```
class LoggingButton extends React.Component {
  // This syntax ensures `this` is bound within handleClick.
  // Warning: this is *experimental* syntax.
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```
이 문법은 <a href="https://github.com/facebookincubator/create-react-app">Create React App</a>에서 기본적으로 사용됩니다.

property initializers 문법을 사용하지 않는경우, <a href="https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions">arrow function</a>을 callback안에서 사용할 수 있습니다.
```
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // This syntax ensures `this` is bound within handleClick
    return (
      <button onClick={(e) => this.handleClick(e)}>
        Click me
      </button>
    );
  }
}
```
이러한 문법의 문제는 서로 다른 callback이 ``` LogginButton ```이 렌더링 될때마다, 생성된다는 것입니다. 대부분의 경우, 이는 큰 문제가 없습니다. 다만, 이 callback이 props로 하위 component에 props로 전달된다면, 이 component들은 특별히 더 많이 렌더링될 수 있습니다. 이러한 종류의 성능문제를 피하기 위해서 constructor에서 바인딩을 하거나, property initializer 문법을 사용하기를 권합니다.
