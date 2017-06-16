# Lifting State Up
가끔, 어떤 component는 같은 변화하는 data를 나타내야 할 때가 있습니다. 이런 경우 공유할 state를 가장 가까운 조상으로 올리는 것을 권합니다. 이것이 어떻게 동작하는지 알아봅시다.

이번 섹션에서는 주어진 온도에서 물이 끓는지의 여부를 계산기를 만들겠습니다.

우선, ``` BoilingVerdict ``` component를 만들겠습니다. 이 component는 ``` celsius ```라는 이름의 prop를 받고, 물이 끓기에 충분한지를 표시합니다.

```
function Boiling
Verdict(props) {
  if (props.celsius >= 100) {
    return <p>The water would boil.</p>;
  }
  return <p>The water would not boil.</p>;
}
```
다음으로, ``` Calculator ``` commponent를 만들것 입니다. 온도를 입력받고, ``` this.state.temperature ```로 가지는 ``` <input> ```을 렌더링합니다.

추가적으로, 현재 input value에 따라 ``` BoilingVerdict ```를 렌더링합니다.
```
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
    return (
      <fieldset>
        <legend>Enter temperature in Celsius:</legend>
        <input
          value={temperature}
          onChange={this.handleChange} />
        <BoilingVerdict
          celsius={parseFloat(temperature)} />
      </fieldset>
    );
  }
}
```
<a href="http://codepen.io/valscion/pen/VpZJRZ?editors=0010">Try it on CodePen.</a>

## 두번째 Input 추가하기
우리의 새로운 요구사항은, 섭씨 input을 추가하는 것 입니다. 화씨 input을 받고, 그것을 동기화 하여, 저장하는 것 입니다.

``` TemperatureInput ``` component를 ``` Calculator ```로 부터 추출하는 것으로 시작하겠습니다. ``` TemperatureInput ```에 ``` "c" ```나 ``` "f" ```가 될 수 있는 ``` scale ``` props를 추가하겠습니다.
```
const scaleNames = {
  c: 'Celsius',
  f: 'Fahrenheit'
};

class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
    const scale = this.props.scale;
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature}
               onChange={this.handleChange} />
      </fieldset>
    );
  }
}
```
이제, ``` Calculator ```를 서로 분리된 temperature input을 렌더링 하도록 바꿀 수 있습니다.
```
class Calculator extends React.Component {
 render() {
   return (
     <div>
       <TemperatureInput scale="c" />
       <TemperatureInput scale="f" />
     </div>
   );
 }
}
```
<a href="http://codepen.io/valscion/pen/GWKbao?editors=0010">Try it on CodePen.</a>

이제, 두개의 input을 가지게 되었습니다. 하지만, 하나의 input에 값을 입력하면, 다른 하나는 업데이트 되지 않습니다. 서로 동기화 되지 않기 때문에, 이는 우리의 요구사항과 다릅니다.

또한, ``` Calculator ```로 부터 값을 받아, ``` BoilingVerdict ```를 표시할 수 도 없습니다. ``` Calculator ```는 ``` TemperatureInput ```안에 숨겨져 있기 때문에 현재 온도를 알 수가 없습니다.

## Conversion(전환) Function 만들기
먼저, 섭씨를 화씨로 전환하는 혹은 그 반대의 기능을 하는 function을 만들겠습니다.
```
function toCelsius(fahrenheit) {
  return (fahrenheit - 32) * 5 / 9;
}

function toFahrenheit(celsius) {
  return (celsius * 9 / 5) + 32;
}
```
이 두 함수는 숫자를 변경합니다. 우리는 ``` temperature ```를 가지고, converter function을 인자로 받아 string을 리턴하는 함수를 만들겠습니다. 우리는 이것을 다른 input을 기반으로 나머지 input의 value를 산하는데 사용할 것 입니다.

이것은 잘못된 ``` temperature ```에 대해서는 빈 string을 리턴하고, 세번 째 소숫점에서 반올림합니다.
```
function tryConvert(temperature, convert) {
  const input = parseFloat(temperature);
  if (Number.isNaN(input)) {
    return '';
  }
  const output = convert(input);
  const rounded = Math.round(output * 1000) / 1000;
  return rounded.toString();
}
```
예를 들어, ``` tryConvert('abc', toCelsius) ```는 빈 string을 리턴하고, ``` try('10.22', toFahrenheit) ```는 ``` '50.396' ```을 리턴합니다.

## State 올리기
현재, 두개의 ``` TemperatureInput ```은 각각 독립적인 local state에서, value를 가지고 있습니다.
```
class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
  ```
그러나 우리는, 이 두 input을 서로 동기화 하려고 합니다. 섭씨 input이 업데이트 되면, 화씨 input또한 온도를 변환하여 반영해야 합니다. 반대의 경우도 마찬가지 입니다.

리액트에서 state를 공유하는 방법은, 공유가 필요한 component의 조상 중 가장 가까운 component로 이동시키는 것 입니다. 이것이 "Lifting state up"입니다. 이제 ``` TemperatureInput ```의 local state를 ``` Calculator ```로 이동시키겠습니다.

``` Calculator ```가 공유 state를 가질 경우, 두 input의 현재 온도의 "진실의 근원"이 됩니다. 이는 서로 같은 값을 가지도록 합니다. 두 ``` TemperatureInput ``` component의 props가 같은 부모 ``` Calculator ``` component에서 나오므로, 두 input은 항상 동기화 됩니다.

이것이 어떻게 동작하는지 차례로 살펴봅시다.

먼저, ```TemperatureInput ``` component의  ``` thia.state.temperature ```를 ``` this.props.temperature ```로 변경하였습니다. 나중에 ``` Calculator ```에서 props를 전달해야 하지만, 우선은 ``` this.props.temperature ```가 이미 존재한다고 가정하겠습니다.
```
render() {
  // Before: const temperature = this.state.temperature;
  const temperature = this.props.temperature;
```
우리가 아는대로, <a href="/react/docs/components-and-props.html#props-are-read-only">props는 읽기전용 입니다.</a> ``` temperature ```가 local state였을 때는, 이를 변경하려면 단순시 ``` TemperatureInput ```에서 ``` this.setState() ```를 호출하기만 하면 되었습니다. 하지만, 지금은 ``` temperature ```는 부모의 props로써 전달되어오기 때문에 이것을 컨트롤 할 수 없습니다.  

보통 리액트에서는 이 문제를 component를 "controlled"하게 만들어서 해결합니다. ``` <input> ``` DOM이 ``` value ```와 ``` onChange ``` 두개의 props를 받은것 같이, 사용자 정의 ``` TemperatureInput ``` component 또한 ``` temperature ```와 ``` onTemperatureChange ``` 두개의 props를 부모 ``` Calculator ```로 부터 받으면 됩니다.

이제, ``` TemperatureInput ```가 temperature를 업데이트 하려면, ``` onTemperatureChange ```를 호출하면 됩니다.
```
handleChange(e) {
   // Before: this.setState({temperature: e.target.value});
   this.props.onTemperatureChange(e.target.value);
```

> 사용자 정의 component에서 prop 이름인  `` temperature ```나 ``` onTemperatureChange ```에 특별한 의미는 없습니다. 일반적인 컨벤션을 따른 이름(``` value ```, ``` onChange ```)은 무엇이든 호출 할 수 있습니다.

``` onTemperatureChange ``` props는 ``` temperature ``` props와 함께, 부모 ``` Calculator ``` component에서 제공됩니다. 이는 자신의 local state를 수정 하므로써, 변경을 처리하기 때문에 두 input이 새 값으로 재 렌더링 됩니다. 잠시 뒤, 새롭게 구현된 ``` Calculator ```를 보겠습니다.

``` Calculator ```를 수정하기 전에, ``` TemperatureInput ```의 변경을 요약해 보겠습니다. 먼저, local state를 지웠고, ``` this.state.temperature ``` 대신에 ``` this.props.temperature ```를 읽도록 하였습니다. 또 변경할 때, ``` this.setState() ``` 대신에 ``` Calculator ```에서 제공된,  ``` this.props.onTemperatureChange ```를 호출하도록 하였습니다.
```
class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(e) {
    this.props.onTemperatureChange(e.target.value);
  }

  render() {
    const temperature = this.props.temperature;
    const scale = this.props.scale;
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature}
               onChange={this.handleChange} />
      </fieldset>
    );
  }
}
```
이제 ``` Calculator ``` component를 갈아 업읍시다.

먼저, 현재 input의 ``` temperature ```와 scale을 ``` Calculator ```의 local state에 저장하겠습니다. 이 state는 우리가  input 에서"lifted up" 한것이고 두 input을 위한 "진실의 근원"이 될것 입니다. 이는 두 input을 렌더링 하기위해 알아야할 모든 데이터를 최소한으로 나타냅니다.

예를 들어, 섭씨 input에 37을 입력하면, ``` Celsius ``` component의 state는 아래와 같이 됩니다.
```
{
  temperature: '37',
  scale: 'c'
}
```
그 뒤에, 화씨 field를 212로 수정하면, ``` Calculator ```의 state는 아래와 같이 됩니다.
```
{
  temperature: '212',
  scale: 'f'
}
```
두 input의 value를 저장할 수는 있지만, 불필요하다고 보여집니다. 가장 최근에 변경된 값과 나타내기위한 scale을 저장하는 것으로 충분합니다. 그러면 이제 다른 input의 값을 현재 ``` temperature ```와 ``` scale ```을 통해 추론 할 수 있습니다.

값들은 같은 state에서 처리되기 때문에, 항상 동기상태로 유지됩니다.
```
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleCelsiusChange = this.handleCelsiusChange.bind(this);
    this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this);
    this.state = {temperature: '', scale: 'c'};
  }

  handleCelsiusChange(temperature) {
    this.setState({scale: 'c', temperature});
  }

  handleFahrenheitChange(temperature) {
    this.setState({scale: 'f', temperature});
  }

  render() {
    const scale = this.state.scale;
    const temperature = this.state.temperature;
    const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
    const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;

    return (
      <div>
        <TemperatureInput
          scale="c"
          temperature={celsius}
          onTemperatureChange={this.handleCelsiusChange} />
        <TemperatureInput
          scale="f"
          temperature={fahrenheit}
          onTemperatureChange={this.handleFahrenheitChange} />
        <BoilingVerdict
          celsius={parseFloat(celsius)} />
      </div>
    );
  }
}
```
<a href="http://codepen.io/valscion/pen/jBNjja?editors=0010">Try it on CodePen.</a>

이제, 어떤 input을 변경하든 ``` Calculator ```의 `` this.state.temperature ```와 ``` this.state.scale ```이 업데이트되기 때문에 상관없습니다. input중 하나가 값을 그대로 가져오므로 모든 user input은 보호되고 다른 input값은 항상 그것을 기반으로 다시 계산합니다.

input을 수정할때 일어나는 일들을 요약해보겠습니다.
- 리액트는 ``` <input> ``` DOM에 ``` onChange ```로 지정된 function을 호출합니다. 이 경우, ``` TemperatureInput ```의 ``` handleChange ``` method입니다.

- ``` TemperatureInput ```의 ``` handleChange ``` method는 ``` this.props.onTemperatureChange() ```를 새롭게 변경하기를 원하는 값을 가지고 호출합니다. ``` onTemperatureChange ```를 포함한 props는 그것의 부모 component인 ``` Calculator ```에서 제공됩니다.

- 이전에 렌더링할때 ``` Calculator ```가 넘겨줬던 섭씨 ``` TemperatureInput ```의 ``` onTemperatureChange ```는 이제 ``` Calculator ```의 ``` handleCelsiusChange ``` method이고 화씨 ``` TemperatureInput ```의 ``` onTemperatureChange ```는 이제 ``` Calculator ```의 ``` handleFahrenheitChange ```입니다. 그래서, 이 두 ``` Calculator ```의 method는 수정한 input에 따라서 호출 됩니다.

- 이 method들 안에서 ``` Calculator ``` component는 리액트에게 ``` this.setState() ```를 새로운 input 값과 현재 방금 수정한 input의 scale 함께 호출하여, 스스로 재 렌더링 요청을 합니다.

- 리액트는 UI가 어떻게 보여져야 하는지 알기위해 ``` Calculator ``` component의 ``` render ```를 호출합니다. 두 input의 값은 현재 온도와 활성화된 scale에 기반하여 재계산 됩니다. 온도의 전환은 여기서 이루어 집니다.

- 리액트는 각각의 ``` TemperatureInput ```의 ``` render ```를 ``` Calculator ```에서 지정한 새로운 props와 함께 호출합니다. 이를 통해 UI가 어떻게 생겼는지 알게 됩니다.

- 리액트 DOM은 바라는 input 값과 일치하도록 DOM을 업데이트 합니다. 방금 수정한 inpur은 현재의 value를 받고, 다른 input은 온도 전환후에, 온도가 업데이트 됩니다.

모든 업데이트는 이와 같은 과정으로 이루어 집니다. 따라서 input은 항시 동기상태로 유지됩니다.

## 교훈

리액트 어플리케이션에서는 변경되는 모든 data에 하나의 "진실의 근원"이 반드시 있어야 합니다. 보통 state는 렌더링에 component에 먼저 추가됩니다. 그리고 다른 component또 한 state가 필요하다면, state를 가장 가까운 조상 component로 들어올리면 됩니다. 다른 component간에 state를 동시화 시키려는 노력 대신에, <a href="/react/docs/state-and-lifecycle.html#the-data-flows-down">top-down data flow</a>를 사용 할 수 있습니다.

state를 들어올리는 것은 양방향 바인딩 접근법 보다 더 많은 "boilerplate"코드를 작성해야하지만, 버그를 찾고 고립시키는데 드는 수고를 덜어줍니다. 어떤 state든 component안에 존재하고 그 component만 state를 변경 할 수 있다면, 표면적은 버그는 굉장히 줄어듭니다. 추가적으로 유저 input을 거부하거나 변환하는 어떤 커스텀 로직이든 구현할 수 있습니다.

만약 무언가가 props나 state로 부터 파생된다면, 대부분 이는 state안에 있으면 안됩니다. 예를 들어, ``` celsiusValue ```와 ``` fahrenheitValue ``` 대신에 우리는 오직 마지막으로 수정된 ``` temperature ```나 그의  ``` scale ```을 저장합니다. 다른 input의 값은 항상 ``` render() ``` method에서 계산됩니다. 이를 통해 유저 input을 손실없이 반올림을 하거나, 반올림을 제거할 수 있습니다.

UI에서 잘못된 것을 봤을때, <a href="https://github.com/facebook/react-devtools">React Developer Tools</a>를 이용해서 props를 분석하고, state를 업데이트라는 component까지 tree를 따라 올라갈 수 있습니다. 이는 여러분의 코드에서 버그를 추적하도록 도와줍니다.
