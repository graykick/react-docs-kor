## State and Lifecyle
이전 섹션에서 다룬, 움직이는 시계 example을 생각해 봅시다.<br>
지금까지 우리는 UI를 업데이트하는 한가지 방법만을 배웠습니다.<br>
우리는 렌더링 결과를 변경하기 위해 ``` ReactDOM.render() ```를 호출하였습니다.
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

이번 섹션에서는, 재사용가능 하고, 캡슐화된 ``` Clock ``` component를 만드는 방법을 배울 것 입니다. 타이머를 스스로 설정하고, 매초마다 스스로 업데이트 할것입니다.

시계(clock)가 어떻게 보이는지 캡슐화하는 것으로 시작할 수 있습니다.
```
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock date={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```
<a href="http://codepen.io/gaearon/pen/dpdoYR?editors=0010">Try it on CodePen.</a>

하지만, 이 코드는 중요한 점을 놓쳤습니다. 사실, ``` Clock ```이 타이머를 설정하고, 매초마다 업데이트 하는것은 ``` Clock ```의 구현사항이어야 합니다.

이상적으로 우리는 한번만 작성하고 ``` Clock ``` 스스로 업데이트 하기를 원합니다.
```
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
이것을 구현하기 위해서, ``` Clock ``` component에 "state"를 추가해야 합니다.
State는 props와 비슷하지만, private하고 전적으로 component에 의해 컨트롤 됩니다.

전에 class로 정의된 component는 몇가지 부가적인 특징을 가진다고, 간단히 언급했습니다. Local state가 정확히 그것 입니다.(class에서만 사용가능한 특징)

## Function에서 Class로 변환하기
5단계를 거쳐, functional component를 ``` Clock ``` 처럼 class로 변환할 수 있습니다.

1. ``` React.Component ```를 extends한 ES6 class를 같은 이름으로 만드세요.

2. 빈 method를 하나 만드세요. 이를 ``` render() ```라고 하겠습니다.

3. function의 몸통(body)를 ``` render() ``` 안으로 옮기세요.

4. ``` render() ```의 몸통(body)안의 ``` props ```를 ``` this.props ```로 변환(replace)하세요.

5. 남은 빈 function declaration(선언)을 삭제하세요.
<a href="http://codepen.io/gaearon/pen/zKRGpo?editors=0010">Try it on CodePen.</a>

```
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```
이제 ``` Clock ```은 function이 아닌, class로 정의되었습니다.
이는 local state와 lifecyle hooks 같은 부가적인 특징들을 사용할 수 있게 해줍니다.

## Class에 Local State 추가하기
3단계를 통해, ``` date ```를 props에서 state로 이동시키겠습니다.
1) ``` render() ``` method의 ``` this.props.date ```를 ``` this.state.date ```로 변환하세요.
```
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```
2) inital(초기의 첫?) ``` this.state ```를 할당하는 class constructor를 추가하세요.
```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```
base constructor에 ``` props ```를 전달한 방법을 확인하세요.
```
constructor(props) {
   super(props);
   this.state = {date: new Date()};
 }
 ```
 Class component는 항상 base constructor를 ``` props ```와 함께 호출해야 합니다.

 3) ``` <Clock /> ``` element의 ``` date ``` prop을 제거하세요.
 ```
 ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
추후에 timer 코드를 해당 component의 뒤에(back) 추가할것 입니다.
결과는 아래와 같습니다.
```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
<a href="http://codepen.io/gaearon/pen/KgQpJd?editors=0010">Try it on CodePen.</a>

## Class에 Lifecyle Method 추가하기
많은 component를 가진 어플리케이션에서는 component가 파괴(destroyed)되었을 때, 사용했던 리소스를 해제하는것이 아주 중요합니다.

``` Clock ```이 처음 DOM에 렌더링될 때는 ** timer를 설정 **하려고 합니다. 리액트에서는 이를 "mounting"이라고 합니다.

``` Clock ```이 생성한 DOM이 삭제될 때는 ** timer를 해제 **하려고 합니다. 리액트에서는 이를 "unmounting"이라고 합니다.

component class에 component가 mount되거나 unmount될 때, 실행되는 특별한 method들을 선언할 수 있습니다.
```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {

  }

  componentWillUnmount() {

  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```
이러한 method들을 "lifecyle hooks"라고 합니다.
``` componentDidMount() ```hook는 component의 결과가 DOM에 렌더링된 후 실행됩니다. timer를 설정하기 좋은 곳입니다.
```
componentDidMount() {
   this.timerID = setInterval(
     () => this.tick(),
     1000
   );
 }
 ```
 ``` this ```의 오른쪽에서 timer ID를 어떻게 저장했는지 확인하세요.
 ``` this.props ```가 리액트에 스스로에 의해 설정되고 ``` this.state ```가 특별한 의미를 가지고 있다고 해도, 무엇을 저장해야하는데 그것이 시각적인 결과에 사용되지 않는다면, 자유롭게 class에 손수 fields를 추가할 수 있습니다.

만약 무언가가 ``` render() ```안에서 사용되지 않는다면, 그것은 state일 필요가 없습니다.

이제 ```  componentWillUnmount() ```lifecycle hook에서 timer를 해제하겠습니다.
```
componentWillUnmount() {
  clearInterval(this.timerID);
}
```
마지막으로, 매초마다 실행되는 ``` tick() ```method를 구현하겠습니다.

local state의 업데이트를 예약하는 ``` this.setState() ```를 사용하겠습니다.
```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
<a href="http://codepen.io/gaearon/pen/amqdNA?editors=0010">Try it on CodePen.</a>

이제 clock은 매초마다 똑딱입니다.

어떤일이 일어났고, method들이 호출된 순서를 빠르게 요약하겠습니다.

1) ``` <Clock />이 ``` ReactDOM.render() ```에 주어지면, 리액트는 ``` Clock ```component의 constructor를 호출합니다. ``` Clock ```이 실시간으로 보여져야 하므로, 현재 시각을 포함한 object로 ``` this.state ```를 초기화 합니다. 추후, 이 state를 업데이트 할 것입니다.

2) 리액트는 다음으로, ``` Clock ```component의 ``` render() ``` mothod를 호출합니다. 이것이 리액트가 스크린에 무엇을 표시할지 아는 방법입니다. 리액트는 다음으로, DOM을 업데이트해서 ``` Clock ```의 렌더링 결과와 매치시킵니다.

3) ``` Clock ```의 결과가 DOM에 삽입될 때, 리액트는  ``` componentDidMount() ``` lifecycle hook를 호출합니다. 내부에서는 ``` Clock ```component가 매 초마다 ``` tick() ```을 호출하는 timer를 설정하기 위해 브라우저에 요청합니다.

4) 브라우저는 매초마다 ``` tick() ```을 호출합니다. 내부에서는 ``` Clock ```component가 현재 시각을 포함한 객체를 가지고 ``` setState() ```를 호출하므로써 UI 업데이트를 예약합니다. ``` setState() ```를 호출한 덕분에 리액트는 state가 변경된것을 알았고 스크린에 무엇을 표시할지 알기위해 다시한번 ``` render() ```를 호출합니다. 이 시간, ``` render() ```method안의 ``` this.state.date ```는 변결될것 입니다. 그리고 렌더링 결과는 업데이트된 시간을 포함할것 입니다. 리액트는 그에따라 DOM을 업데이트 합니다.

5) ``` Clock ```component가 DOM에서 제거되면, 리액트는 ``` componentWillUnmount() ``` lifecycle hook를 호출하여 timer을 중지합니다.

## State 올바르게 사용하기
``` setState() ```에대해 세 가지 알아야 하는 것이 있습니다.

### state를 직접 수정하지 말것
예를 들어, 이 경우, component를 다시 렌더링하지 않습니다.
```
// Wrong
this.state.comment = 'Hello';
```

대신에 ``` setState() ```를 사용하세요.
```
// Correct
this.setState({comment: 'Hello'});
```
``` this.state ```는 오직 constructor에서만 할당가능합니다.

### State의 업데이트는 비동기 일것이다.
리액트는 성능을 위해 여러 ``` setState() ```호출을 한번의 업데이트에 배치합니다.

때문에 ``` this.props ```와 ``` this.state ```는 비동기로 업데이트 될 수 있습니다. 따라서, 다음 state의 계산을 state의 값에 의존해서는 안됩니다.

예를 들어, 이 코드는 counter를 업데이트에 실패할것 입니다.
```
this.setState({
  counter: this.state.counter + this.props.increment,

```
이를 해결하기 위해, object가 아닌 function을 인자로 받는 두번째 형태의 ``` setState() ```를 사용해야 합니다. 이 function은 첫번째 인자로 이전 state를 받고 업데이트가 적용된 시점의 props가 두번째 인자입니다.
```
// Correct
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
```

위에서는 arrow  function을 사용하였습니다. 물론 이것은 regular function으로도 동작합니다.
```
// Correct
this.setState(function(prevState, props) {
  return {
    counter: prevState.counter + props.increment
  };
})
```

### State 업데이트는 병합된다.
``` setState() ```를 호출할 때, 리액트는 제공된 object를 현재 상태에 병합합니다.

예를 들어, state는 독립적인 변수들을 포함할 수 있습니다.
```
constructor(props) {
    super(props);
    this.state = {
      posts: [],
      comments: []
    };
  }
```
다음으로, 이것을 분리된 ``` setState() ```를 호출하여 독립적으로 업데이트 할 수 있습니다.
```
componentDidMount() {
   fetchPosts().then(response => {
     this.setState({
       posts: response.posts
     });
   });

   fetchComments().then(response => {
     this.setState({
       comments: response.comments
     });
   });
 }
 ```
이 병합은 얕습니다. 그래서 ``` this.setState({comment}) ```는 ```this.state.posts ```를 온전히 보존합니다. 그러나, ``` this.state.comments ```를 완전히 대체합니다.


## Data는 아래로 흐른다.
parent component이던 child component이던 특정 component가 stateful한지 혹은 stateless한지 알지 못합니다. 그리고 이들은 function으로 정의되었든 class로 정의되었든 상관하지 상관할 필요가 없습니다.

이것이 state가 local혹은 encapsulated(캡슐화)로 불리는 이유입니다. 소유한 component아닌 다른 component에서는 접근 불가능하며, 소유한 component는 state를 설정할 수 있습니다.

component는 state를 child component의 props로 전달 할 수 있습니다.
```
<h2>It is {this.state.date.toLocaleTimeString()}.</h2>
```
이는 사용자 정의 component에서도 동작합니다.
```
<FormattedDate date={this.state.date} />
```
``` FormattedDate ``` component는 props로 ``` date ```를 받을 수 있고, 그것이 ``` Clock ```의 state인지 ``` Clock ```의 props인지 아니면, 손으로 타이핑되었는지 알 수 없습니다.
```
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```
<a href="http://codepen.io/gaearon/pen/zKRqNB?editors=0010">Try it on CodePen.</a>

이것은 보통 "top-down"이나 "unidirectional" data flow라고 불립니다. 어떤 state이든 항상 특정 component에 속하고, 어떤 data나 UI에서 파생된 state는 tree의 "아래"에 있는 component에만 영향을 미칩니다.

여러분이 component 트리는 props의 폭포(waterfall)로 생각했다면, 각각의 state는 아래로 흐르고 임의의 지점에 합류하는 추가적인 수원지라고 볼 수 있습니다.

모든 component가 실제로 분리되어 있다는 것을 보이기 위해, 3개의 ``` <Clock> ```들을 렌더링하는 ``` App ``component를 만들겠습니다.
```
function App() {
  return (
    <div>
      <Clock />
      <Clock />
      <Clock />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```
<a href="http://codepen.io/gaearon/pen/vXdGmd?editors=0010">Try it on CodePen.</a>
각각의 ``` Clock ```은 자신의 timer를 설정하고 독립적으로 업데이트 합니다.

리액트 앱에서는 component가 stateful한지 stateless한지 고려하는 것은 시간이 지남에 따라 변할 수 있는 component의 세부 구현사항입니다. stateless component안에 stateful component를 사용할 수도 있고, 그 반대의 경우도 가능합니다.
