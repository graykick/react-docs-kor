# 조건부(Conditional) 렌더링
리액트에서는 필요한 동작을 캡슐화하는 별개의 component를 만들 수 있습니다. 그 후, 어플리케이션의 상태에 따라서 그중 일부만 렌더링 할 수 있습니다.  

리액트의 조건부 렌더링은 JavaScript의 condition과 똑같이 동작합니다. 현재의 상태를 나타내는 element를 생성하기 위해, JavaScript 연산자 ``` if ```나 <a href="https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Conditional_Operator">conditional operator</a>를 사용하세요. 그러면, 리액트가 UI에 맞게 업데이트합니다.

아래의 두 component를 생각해보세요.
```
function UserGreeting(props) {
  return <h1>Welcome back!</h1>;
}

function GuestGreeting(props) {
  return <h1>Please sign up.</h1>;
}
```
유저의 로그인 여부에 따라서, 이 commponent들 중 하나를 보여주는 ``` Greeting ``` commponent를 만들겠습니다.
```
function Greeting(props) {
  const isLoggedIn = props.isLoggedIn;
  if (isLoggedIn) {
    return <UserGreeting />;
  }
  return <GuestGreeting />;
}

ReactDOM.render(
  // Try changing to isLoggedIn={true}:
  <Greeting isLoggedIn={false} />,
  document.getElementById('root')
);
```
<a href="https://codepen.io/gaearon/pen/ZpVxNq?editors=0011">Try it on CodePen.</a>

이 예제는 ``` isLoggedIn ``` prop의 값에 따라서 다른 greeting을 렌더링 합니다.

## Element 변수
element를 변수에 저장할 수 있습니다. 이는 결과의 나머지 부분이 변하지 않는 동안, component의 일부분을 조건적으로 렌더링 할 수 있게 해줍니다.

로그아웃과 로그인 버튼을 나타내는 아래의 새로운 component를 보세요.
```
function LoginButton(props) {
  return (
    <button onClick={props.onClick}>
      Login
    </button>
  );
}

function LogoutButton(props) {
  return (
    <button onClick={props.onClick}>
      Logout
    </button>
  );
}
```
아래의 예제에서, ```LoginControle ```이라는 이름의 <a href="/react/docs/state-and-lifecycle.html#adding-local-state-to-a-class">stateful component</a>를 생성할것 입니다.

이는 현재의 state에 따라서 ``` <LoginButton/> ```이나 ``` <LooutButton/> ```을 렌더링 합니다. 이전 예제의  ``` <Greeting /> ```도 같이 렌더링 합니다.
```
class LoginControl extends React.Component {
  constructor(props) {
    super(props);
    this.handleLoginClick = this.handleLoginClick.bind(this);
    this.handleLogoutClick = this.handleLogoutClick.bind(this);
    this.state = {isLoggedIn: false};
  }

  handleLoginClick() {
    this.setState({isLoggedIn: true});
  }

  handleLogoutClick() {
    this.setState({isLoggedIn: false});
  }

  render() {
    const isLoggedIn = this.state.isLoggedIn;

    let button = null;
    if (isLoggedIn) {
      button = <LogoutButton onClick={this.handleLogoutClick} />;
    } else {
      button = <LoginButton onClick={this.handleLoginClick} />;
    }

    return (
      <div>
        <Greeting isLoggedIn={isLoggedIn} />
        {button}
      </div>
    );
  }
}

ReactDOM.render(
  <LoginControl />,
  document.getElementById('root')
);
```
<a href="https://codepen.io/gaearon/pen/QKzAgB?editors=0010">Try it on CodePen.</a>

변수를 선언하고, ``` if ```구문을 사용하는 것은 조건부 렌더링의 좋은 방법입니다. 더 간단한 문법을 원하실 수 도 있습니다. JSX에서 inline condition를 사용하는 몇가지 방법이 있습니다. 아래에서 설명하겠습니다.

## inline if와 논리연산자 &&
표현식을 중괄호({})로 감싸면, <a href="/react/docs/introducing-jsx.html#embedding-expressions-in-jsx">표현식을 JSX에 삽입</a>할 수 있습니다. 여기에는 JavaScript 논리연산자 &&도 포함되어 있는데요. 이 것은  조건부 element에 유용합니다.
```
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 &&
        <h2>
          You have {unreadMessages.length} unread messages.
        </h2>
      }
    </div>
  );
}

const messages = ['React', 'Re: React', 'Re:Re: React'];
ReactDOM.render(
  <Mailbox unreadMessages={messages} />,
  document.getElementById('root')
);
```
<a href="https://codepen.io/gaearon/pen/ozJddz?editors=0010">Try it on CodePen.</a>
이것은 JavaScript이기 때문에 동작합니다. ``` true && expression(표현식) ```은 항상 ``` expression ```을 평가합니다. 반대로 ``` false && expression(표현식) ```은 항상 false를 평가합니다.

그래서, condition이 ``` ture ```이면 ``` && ```오른쪽에 있는 element는 출력 될것이고, ``` false ```라면 리액트는 무시할것 입니다.

## inline If-Else와 조건 연산자
inline 조건부 렌더링을 하는 또 다른방법은 JavaScript의 조건 연산자 <a href="https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Conditional_Operator"><code>condition ? true : false</code></a>를 사용하는것 입니다.

아래의 예제에서는 text의 일부분을 조건부 렌더링 합니다.
```
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      The user is <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.
    </div>
  );
}
```
이 방법은 더 큰 표현식에서도 사용할 수 있지만, 비교적 덜 명확합니다.
```
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      {isLoggedIn ? (
        <LogoutButton onClick={this.handleLogoutClick} />
      ) : (
        <LoginButton onClick={this.handleLoginClick} />
      )}
    </div>
  );
}
```
JavaScript에서 처럼, 이 문제는 여러분이나 여러분의 팀이 더 읽기 쉽다고 생각하는 것을 기반으로, 적당한 스타일을 선택하는것은 여러분에게 달려있습니다. 또 조건이 너무 복잡할 경우에는, <a href="/react/docs/components-and-props.html#extracting-components">component를 추출하는 것</a>이 좋은 방안이 될 수있다는 것을 기억하세요.

## Component렌더링 막기
드문 경우지만, 다른 component에서 렌더되더라도, component를 스스로 숨기고 싶을 수 있습니다. 이를 위해서는 렌더링 결과대신에, ``` null ```을 리턴하면 됩니다.

아래의 예제에서, ``` <WarningBanner /> ```는 ``` warn``` props에 따라서, 렌더링됩니다. 만약 props의 값이 ``` false ```라면 component는 렌더링 되지 않습니다.
```
function WarningBanner(props) {
  if (!props.warn) {
    return null;
  }

  return (
    <div className="warning">
      Warning!
    </div>
  );
}

class Page extends React.Component {
  constructor(props) {
    super(props);
    this.state = {showWarning: true}
    this.handleToggleClick = this.handleToggleClick.bind(this);
  }

  handleToggleClick() {
    this.setState(prevState => ({
      showWarning: !prevState.showWarning
    }));
  }

  render() {
    return (
      <div>
        <WarningBanner warn={this.state.showWarning} />
        <button onClick={this.handleToggleClick}>
          {this.state.showWarning ? 'Hide' : 'Show'}
        </button>
      </div>
    );
  }
}

ReactDOM.render(
  <Page />,
  document.getElementById('root')
);
```
<a href="https://codepen.io/gaearon/pen/Xjoqwm?editors=0010">Try it on CodePen.</a>
``` null ```을 리턴하는 component의 ``` render ``` method는 component의 lifecycle이 실행되는데 영향을 미치지 않습니다. 예를 들어, ``` componentWillUpdate ```와 ``` componentDidUpdate  ```는 여전히 호출됩니다.
