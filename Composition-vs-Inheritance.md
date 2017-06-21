# 조합(composition)과 상속
리액트는 강력한 조합 모델을 가지고 있습니다. 때문에, component간의 코드 재사용을 위해서는 상속보다는 조합을 사용하는것을 권합니다.

In this section, we will consider a few problems where developers new to React often reach for inheritance, and show how we can solve them with composition.

## 방지(Containment)
어떤 component는 그들의 자식요소를 미리 알지 못합니다. 이는 특히, ``` Sidebar ```나 ``` Dialog ```같이 일반적인 "box"를 나타내는 component에서 흔합니다.

이런 component에는 ``` children ```이라는 특별한 props를 사용하여, 직접 자식 element를 전달하는 것을 추천합니다.
```
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
```
이렇게 JSX를 중첩해서 다른 commponent가 임의의 자식을 전달 할 수 있습니다.
```
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```
<a href="http://codepen.io/gaearon/pen/ozqNOV?editors=0010">Try it on CodePen.</a>

``` <FancyBorder> ``` JSX tag 안에 있는 모든것은 ``` FancyBorder ``` component의 ``` child ``` props가 됩니다. ``` FancyBorder ```가 ``` {props.children} ```을 ``` <div> ```안에서 렌더링 했기 때문에, 전달된 element는 최종결과에 나타납니다.

이는 보다 드물지만, 가끔 여러 "hole"이 필요할 때가 있습니다. 이런 경우, ``` children ```대신에 다른 자신만의 컨벤션을 찾아야 합니다.
```
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={
        <Contacts />
      }
      right={
        <Chat />
      } />
  );
}
```
<a href="http://codepen.io/gaearon/pen/gwZOJp?editors=0010">Try it on CodePen.</a>
``` <Contacts /> ```와 ```<Chat /> ```같은 리액트 element는 단지 object입니다. 따라서, 이들을 다른 data처럼 props로 전달할 수 있습니다.

## 특수화

가끔, component가 다른 component의 "특수 경우"라고 생각합니다. 예를 들어, ``` WelcomeDialog ```는 ``` Dialog ```의 특수 경우 입니다.

리액트에서는 이를 조합(composition)으로 해결 할 수 있습니다. "특수한" component가 "일반적인" component를 렌더링 하고, props를 설정합니다. (where a more "specific" component renders a more "generic" one and configures it with props)
```
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
    </FancyBorder>
  );
}

function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!" />
  );
}
```
<a href="http://codepen.io/gaearon/pen/kkEaOZ?editors=0010">Try it on CodePen.</a>
조합은 class로 선언된 component에서도 똑같이 작동합니다.
```
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
      {props.children}
    </FancyBorder>
  );
}

class SignUpDialog extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.handleSignUp = this.handleSignUp.bind(this);
    this.state = {login: ''};
  }

  render() {
    return (
      <Dialog title="Mars Exploration Program"
              message="How should we refer to you?">
        <input value={this.state.login}
               onChange={this.handleChange} />
        <button onClick={this.handleSignUp}>
          Sign Me Up!
        </button>
      </Dialog>
    );
  }

  handleChange(e) {
    this.setState({login: e.target.value});
  }

  handleSignUp() {
    alert(`Welcome aboard, ${this.state.login}!`);
  }
}
```
<a href="http://codepen.io/gaearon/pen/gwZbYa?editors=0010">Try it on CodePen.</a>

## 그래서, 상속은 어떤가
페이스북에서는 수 천개의 리액트 component를 사용하지만, component 상속을 추천드릴만한 그 어떤 경우도, 없었습니다.

Props와 composition은 component의 생김새와 동작을 명확하고 안전하게 커스터마이즈 할 수 있는 유연성을 제공합니다. component는 원시 값(primitive value), 리액트 element, function을 props로 받을 수 있다는 사실을 기억하세요.

component간에 UI외적인 기능을 재 사용하고싶은 경우, 이를 별도의 JavaScript module로 추출하는것을 권해드립니다. component는 이를 import하면, 해당 function, object, class등을 상속 없이 사용할 수 있습니다. 
