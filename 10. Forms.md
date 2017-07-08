# Forms
HTML form element는 리액트의 DOM element와는 조금 다르게 동작합니다. form element는 자연히 내부의 상태를 가지게 됩니다. 예를 들어, 이 순수 HTML로 이루어진 form은 하나의 name을 받습니다.
```
<form>
  <label>
    Name:
    <input type="text" name="name" />
  </label>
  <input type="submit" value="Submit" />
</form>
```
이 HTML form은 유저가 form을 submit했을때, 새 페이지를 브라우징 하는 기본 행동을 가지고 있습니다. 리액트에서 이러한 행동을 원한다면, 그냥 동작합니다. 다만, 대부분의 경우 form의 submit을 처리하고, 유저가 입력한 data에 접근하는 function을 만드는 것이 더 편리합니다. 이를 위한 표준적인 방법은 "controlled component"라는 기술을 사용하는 것입니다.

## controlled Component
HTML에서 ``` <input> ```, ``` <texrarea> ```, ``` <select> ```같은 form element들은 보통, 그들만의 state를 가지고 있고, 그것은 유저의 input에 기반하여 업데이트 합니다. 리액트에선 잘 변경되는 state를 보통, commponent의 state로 가지고 있고, 오직 ``` setState() ```만을 사용해서 업데이트합니다.

리액트 state를 "single source of truth"(하나의 진실의 근원)으로 만들면, 위의 두 가지를 결합할 수 있습니다. 그러면, form을 렌더링하는 리액트 commponent는 다음 유저 input에 따라, form에서 무슨일이 일어나는 일을 제어합니다. input form element의 값은 리액트에 의해 제어되고 이를 "controlled commponent"라고 합니다.

예를 들어, 이전 예제에서 submit되었을때, name을 출력하려고 한다면, form을 "controlled component"로 작성할 수 있습다.
```
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```
<a href="https://codepen.io/gaearon/pen/VmmPgp?editors=0010">Try it on CodePen.</a>

form element에 value attribute가 설정되었기 때문에, 보여지는 value는 항상 ``` this.state.value ```가 되고, 리액트 state는 진실의 근원이 됩니다. 리액트 state를 업데이트하기 위해, ``` handleChange ```가 모든 kestroke에서 실행됩니다. 따라서, 보여지는 value는 유저의 타이핑으로 업데이트 됩니다.

controlled component를 사용하면, 모든 state 변경마다, 연관된 handler function을 가지게 됩니다. 이를 통해, 유저의 input수정하거나, 변경할 수 있습니다. 예를 들어, name을 모두 대문자로 입력받고 싶다면, ```handleChange ```를 이렇게 사용할 수 있습니다.
```
handleChange(event) {
  this.setState({value: event.target.value.toUpperCase()});
}
```

## textarea target
HTML에서는 ``` <textarea> ```element는 children에 의해 text가 정의됩니다.
```
<textarea>
  Hello there, this is some text in a text area
</textarea>
```
리액트에서 ``` <textarea> ```는 ``` value ``` attribute를 대신 사용합니다. 이 방법은 ``` <textarea> ```를 사용하는 form이 single-line input과 매우 유사하게 입력받을 수 있게 해줍니다.
```
class EssayForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: 'Please write an essay about your favorite DOM element.'
    };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('An essay was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <textarea value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

## select tag
HTML의 ``` <select> ``` tag는 drop-down list를 만듭니다. 예를 들어, 이 HTML은 맛에 대한 drop-down list를 만듭니다.
```
<select>
  <option value="grapefruit">Grapefruit</option>
  <option value="lime">Lime</option>
  <option selected value="coconut">Coconut</option>
  <option value="mango">Mango</option>
</select>
```
``` selected ``` attribute으로 인해, 위의 Coconut option은 초기 선택 항목입니다. 하지만, 리액트에서는 이보다는 root ``` <select> ``` tag에 ``` value ``` attribute를 사용합니다. 한 곳에서만 업데이트가 필요하기 때문에, controlled component에서는 이런 방법이 더 편합니다. 예제 입니다.
```
class FlavorForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: 'coconut'};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('Your favorite flavor is: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Pick your favorite La Croix flavor:
          <select value={this.state.value} onChange={this.handleChange}>
            <option value="grapefruit">Grapefruit</option>
            <option value="lime">Lime</option>
            <option value="coconut">Coconut</option>
            <option value="mango">Mango</option>
          </select>
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```
<a href="https://codepen.io/gaearon/pen/JbbEzX?editors=0010">Try it on CodePen.</a>

이는 전체적으로, ``` value ``` attribute를 받고, controlled component를 구현하여 사용할 수 있다는 점에서, ``` <input type="text"> ```, ``` <textarea> ```, ``` <select> ```모두가 비슷하게 동작하게 합니다.

## 다수의 input handling
다수의 controlled ``` input ``` element를 handling 해야할 경우, 각각의 ``` element ```에 ``` name ``` attribute를 추가하고, handler function이 ``` event.targer.name ```에 따라 어떤 행동을 할지 성택하게 하면 됩니다.
예제입니다.
```
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }

  render() {
    return (
      <form>
        <label>
          Is going:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange} />
        </label>
        <br />
        <label>
          Number of guests:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```
<a href="https://codepen.io/gaearon/pen/wgedvV?editors=0010">Try it on CodePen.</a>

주어진 input name과 같은 key의 state를 업데이트 하기위해 ES6의 <a href="https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names">computed property name</a>을 사용한것을 살펴보세요.

```
this.setState({
  [name]: value
});
```
이것은 아래의 ES5 code와 같습니다.
```
var partialState = {};
partialState[name] = value;
this.setState(partialState);
```
또한, ``` setState() ```는 현재 state에 일부 state를 자동적으로 병합하기 때문에, 우리는 변경되는 부분과 함께 호출하기만 하면 됩니다.

## Controlled Component의 대안
변경되는 모든 state에 대한 handler event를 작성해야 하고, 모든 input state를 리액트 component를 통해 수송해야 하기 때문에, controlled component를 사용하는 것이 귀찮게 느껴질 수 도 있습니다. 이런 문제는 특히, 기존의 코드를 리액트로 변경할때나 리액트 코드와 리액트 코드가 아닌것을 통합할때, 특히 두드러집니다. 이런 상황의 경우, input form을 구현하기 위한 대안 기술로 <a href="/react/docs/uncontrolled-components.html">uncontrolled components</a>를 생각해 볼 수 있습니다.
