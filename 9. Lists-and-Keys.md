# List와 Key
먼저, JavaScript에서 list를 어떻게 변형시켰는지 돌이켜봅시다.

주어진 아래의 코드에서, ``` number ```의 값을 얻고, 2배 하기위해, ``` map() ``` function을 사용했습니다. 2배 시키기 위한, ``` map() ```이 리턴한 새로운 배열을 할당하고, 그것을 출력했습니다.
```
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((number) => number * 2);
console.log(doubled);
```
이 코드는 ``` [2, 4, 6, 8, 10] ```을 console에 출력합니다.

리액트에서 array를 <a href="/react/docs/rendering-elements.html">elements</a>의 list로 변환시키는 것이 이와 거의 비슷합니다.

## 여러 Component 렌더링하기
중괄호({})로 감싸면, element의 collection을 만들고 그것을 JSX에 포함시킬 수 있습니다.

아래의 코드에서, ``` number ``` array를 JavaScript ``` map() ``` function를 이용해서, 순환했습니다. 각각의 item에서 ``` <li> ``` element를 리턴합니다. 마지막으로, element array의 결과를, ``` listsItem ```에 할당합니다.
```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);
```

``` listItems ``` array 전체를 ``` <ul> ```에 포함시키고, 그것을 <a href="/react/docs/rendering-elements.html#rendering-an-element-into-the-dom">DOM에 렌더링</a>하였습니다.
```
ReactDOM.render(
  <ul>{listItems}</ul>,
  document.getElementById('root')
);
```
<a href="https://codepen.io/gaearon/pen/GjPyQr?editors=0011">Try it on CodePen.</a>
이 코드는, 1 부터 5까지의 bullet list를 보여줍니다.

## 기본 List Component
보통은, <a href="/react/docs/components-and-props.html">component</a>안에서, list를 렌더링합니다.

이전의 예제를 component가 ``` numbers ``` array를 받아들이고, 그것을 element의 unordered list로 출력합니다.
```
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li>{number}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```
이 코드를 실행하면, list item에 key를 반드시 지정하야 한다는 warning을 보게됩니다. "key"는 element list를 생성할때, 포함시켜햐 하는 특별한 string attribute입니다. 다음 섹션에서 이것이 얼마나 중요한지, 논의 하겠습니다.

``` numbers.map() ```에서 list item에 key를 할당해서, missing key 이슈를 해결해 봅시다.
```
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```
<a href="https://codepen.io/gaearon/pen/jrXYRR?editors=0011">Try it on CodePen.</a>

## Keys
Key는 리액트가 item이 변경되었는지, 추가되었는지, 삭제되었는지를 판별하도록 돕습니다. element에 고정된, 변별성을 부여하기 위해서, array안의 element에는 key가 반드시 주어져야 합니다.
```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```
key를 선택하는 가장 좋은 방법은 형제 list item중 유일하게 식별가능한 string을 사용하는 것 입니다. 가장 흔한 방법은 key로 data의 id를 사용하는것 입니다.
```
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```
렌더링된, item에 대한 고정된 ID가 없다면, 마지막 보루로 item의 index를 key로 사용할 수 있습니다.
```
const todoItems = todos.map((todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
);
```
순서가 변할 가능성이 있는경우, index를 key로 사용하는 것은 좋은 방법이 아닙니다. 만약 더 관심이 있다면, <a href="/react/docs/reconciliation.html#recursing-on-children">key의 필요성에 대한 자세한 설명</a>을 읽어 보세요.

## Key로 component추출 하기
Key는 array 주변(surrounding array)의 context에서만 의미가 있습니다.

예를 들어, ``` ListItem ```을 추출한다면, ``` <li> ``` root element가 아니라, array안의 ``` <ListItem /> ``` element에서 key를 가지고 있어야 합니다.

** 예제: 잘못된, key 사용 **
```
function ListItem(props) {
  const value = props.value;
  return (
    // 땡! 여기서는 key를 사용할 필요가 없습니다.:
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // 땡! 여기서 key는 각각 달라야 합니다.:
    <ListItem value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

**예제: 올바른, key 사용**
```
function ListItem(props) {
  // 딩동댕! 여기서는 key를 사용할 필요가 없습니다.:
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // 딩동댕! array에서 key는 각각 달라야 합니다.
    <ListItem key={number.toString()}
              value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```
<a href="https://codepen.io/rthor/pen/QKzJKG?editors=0010">Try it on CodePen.</a>
경험적으로, 좋은 방법은 ``` map() ```안의 element들은 key가 필요하다는 것입니다.

## Key는 모든 형제중에서 고유해야 한다.
array에서 사용되는 key는 그 형제들 중에서 반드시 고유해야 합니다. 하지만, 전역적으로 고유해야 하는것은 아닙니다. 서로 다른 두 array에서는 같은 key를 사용할 수 있습니다.
```
function Blog(props) {
 const sidebar = (
   <ul>
     {props.posts.map((post) =>
       <li key={post.id}>
         {post.title}
       </li>
     )}
   </ul>
 );
 const content = props.posts.map((post) =>
   <div key={post.id}>
     <h3>{post.title}</h3>
     <p>{post.content}</p>
   </div>
 );
 return (
   <div>
     {sidebar}
     <hr />
     {content}
   </div>
 );
}

const posts = [
 {id: 1, title: 'Hello World', content: 'Welcome to learning React!'},
 {id: 2, title: 'Installation', content: 'You can install React from npm.'}
];
ReactDOM.render(
 <Blog posts={posts} />,
 document.getElementById('root')
);
```
<a href="https://codepen.io/gaearon/pen/NRZYGN?editors=0010">Try it on CodePen.</a>
key는 리액트의 hint역할을 합니다. 그러나, 그것을 component에 전달하지는 않습니다. 만일, key와 같은 값이 component에서 필요하다면, 다른 이름의 props로 전달해야 합니다.
```
const content = posts.map((post) =>
  <Post
    key={post.id}
    id={post.id}
    title={post.title} />
);
```

## JSX에 map() 삽입하기
위의 예제는 분리된 ``` listItems ```변수를 선언하고, 그것을 JSX에 포함시켰습니다.
```
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <ListItem key={number.toString()}
              value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```
중괄호({})로 감싸면, JSX는 <a href="/react/docs/introducing-jsx.html#embedding-expressions-in-jsx">모든 표현식</a> 삽입을 허용합니다. 따라서, inline ``` map() ```를 사용할 수 있습니다.
```
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map((number) =>
        <ListItem key={number.toString()}
                  value={number} />
      )}
    </ul>
  );
}
```
<a href="https://codepen.io/gaearon/pen/BLvYrB?editors=0010">Try it on CodePen.</a>

가끔 이런 코드가 더 깔끔하기도 합니다. 다만, 이 스타일의 경우도, 악용될 수 있습니다. JavaScript에서와 같이, 이것은 가독성에 따른 여러분의 선택에 달렸습니다. 만약 ``` map() ```이 너무 많다면, <a href="/react/docs/components-and-props.html#extracting-components">component 추출</a>이 도움이 될 수 있다는 것을 기억해두세요. 
