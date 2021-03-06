# React 완벽 가이드 with Redux, Next.js, TypeScript

- [링크](https://www.udemy.com/course/best-react/)

## 섹션 1: 시작하기

- react.js: 자바스크립트 라이브러리로 사용자 인터페이스를 만드는 데 사용된다. 기존 사이트는 요청을 보내고 html 페이지로 응답이 오기까지 기다려야했다. 자바스크립트 DOM을 통해 새로운 html페이지를 요청하지 않고 화면을 조작할 수 있다. 복잡하고 길게 써야했던 기존 코드를 리액트를 사용해 컴포넌트화해 재활용하고 저수준의 지침은 리액트가 작성해주고 우리는 고수준에서 작업하며 리액트와 연계해 복잡한 사용자 인터페이스를 더 쉽게 구축하도록 한다.
- angular: 많은 기능이 내장된 프레임워크로 타입스크립트를 사용한다. 컴포넌트 기반이다.
- vue.js: angular와 react.js의 중간. 컴포넌트 기반인 것은 다른 두 개와 같다.
- 강의 소개 및 개요: 리액트 기본 핵심 기능(컴포넌트로 사용자 이벤트 구축, events, props, state, 스타일링, hooks), 심화 개념(side effects, refs, more hooks, context api, redux - 가장 중요한 서드 파티 라이브러리 중 하나, forms, http requests, custom hooks, routing, 배포, next.js), 요약 & 정리(javascript, reactjs 요약)
- 확장 플러그인 prettier 설치 -> 파일 - 기본 설정 - 바로가기 키(ctrl + k+ s) -> format document (shift + alt + f)로 포맷팅 사용 / 설정에서 editor formatter 설정 prettier
- 확장 플러그인 material icon theme



## 섹션 2: 자바스크립트 새로고침

- 화살표 함수로 this keyword 문제 해결. 화살표 함수에서는 늘 정의한 객체를 나타낸다.
- export default는 해당 파일에서 하나인 default export로 지정된 것을 가져오며 가져올 때 이름을 마음대로 지정 가능하다. default 키워드를 사용하지 않았을 때는 중괄호 안에 이름을 정확히 명시해서 어떤 것을 가져올 지 알려줘야 한다. 하지만 as를 통해 이름을 바꿀 수 있다.
- ES7 클래스와 속성 메서드: 클래스안에서 생성자와 상속받는 클래스 생성자의 super() 없고 프로퍼티에 this 키워드도 없이 지정한다. 함수 역시 화살표 함수로 만들 수 있다. 화살표 함수에서는 this키워드를 사용해 프로퍼티에 접근한다.

```javascript
// 이전
class Human {
    constructor() {
        this.gender = 'male';
    }
    
    printGender() {
        console.log(this.gender);
    }
}

// 현재
class Human {
    gender = 'male';
    
    printGender = () => {
        console.log(this.gender);
    }
}
```

- 스프레드 및 나머지 연산자: ... / 스프레드는 모든 원소와 프로퍼티를 가져와 새 배열이나 객체 등에 전달하지만 디스트럭쳐링(구조분해할당)은 원소나 프로퍼티를 하나만 가져와서 변수에 저장. / 참조형 타입을 복사하기위해 재할당 하지말고 스프레드 연산자를 사용하자.



## 섹션 3: 리액트 기초 및 실습 컴포넌트

- 컴포넌트 중심의 사용자 인터페이스 구축하기. 리액트는 컴포넌트가 전부다. 모든 사용자 인터페이스는 컴포넌트로 구축된다.

- 컴포넌트: 사용자 인터페이스에서 재사용할 수 있는 빌딩 블럭. html, css, 로직을 위한 js의 결합. 리액트가 각각의 빌딩 블럭을 결합해 최종 사용자 인터페이스에서 어떻게 구성될 것인지 명령한다. 재사용 가능해 반복을 피할 수 있고 우려사항을 분리해 코드를 작고 관리 가능하게 분리한다. 
- 선언적 접근 방식: 바닐라 자바스크립처럼 html요소를 생성하고 사용자 인터페이스에서 어떤 위치에 삽입되어야 하는지, 직접 구체적인 DOM을 업데이트 하도록  명령하지 않는다. 대신 항상 원하는 목표 상태 또는 다양한 상황에 따른 다양한 목표 상태를 정의하고 실제 페이지에서 어떤 요소가 추가, 삭제, 업데이트 되어야 하는지 해결해야 하며 어떤 상태가 사용되어야 하는지 정의하면 된다. 
- index.js: 프로젝트를 실행할 때 처음으로 실행된다. react, react-dom을 import할 때 react 기능을 사용하기 위해 하는 것이다. 
- `npm start`를 통해 개발 서버를 구동하고 코드를 감시를 하며 브라우저에 전달하기 전에 먼저 변환한다. 그래서 특정 기능들이 작동할 수 있게 한다.(예를 들어 import css같은 것)
- index.html: spa에서 유일하게 브라우저에서 전달되고 관리되어 렌더링되는 html파일이다. index.js에서 index.html에 있는 root라는 div를 선택하고 App을 임포트해 사용한다. App은 컴포넌트로 root id와 함께 그 요소가 있는 자리에서 렌더링한다.
- App.js: html구문을 return하고 있다. 이 파일을 JSX라고 한다.
- JSX: 자바스크립트 xml. js코드를 브라우저 친화적으로 변환.
- 기존 자바스크립트 명령적인 접근을 따른다. 자바스크립트와 브라우저가 어떤 단계에서 무엇을 해야하는지 단게별로 정확히 지시하고 있다. 이런 지침을 작성하는건 지루하고 반복적이므로 리액트를 통해 최종 상태만 정의할 수 있게 만든다.
- 사용자 지정 컴포넌트 만들기: 컴포넌트 안에 jsx코드에서 반환하는 문장마다 반드시 한 개의 루트 요소를 갖는다. 전체 html코드를 하나의 div코드로 감싸는 방법이 있다.

ExpenseItem.js

- css파일을 사용하기 위해 import한 후 class가 아닌 `className`으로 적용했다. html처럼 보이지만 react에서 제공하는 jsx로 class는 예약어기 때문에 className을 사용한다. 
- 날짜 객체는 string으로 변형해줘야 한다.

```javascript
import './ExpenseItem.css';

function ExpenseItem() {
    const expenseDate = new Date(2022, 7, 1);
    const expenseTitle = 'Car Insurance';
    const expenseAmount = 294.67;

  return (
    <div className="expense-item">
      <div>{expenseDate.toDateString()}</div>
      <div className="expense-item__description"> 
        <h2>{expenseTitle}</h2>
        <div className="expense-item__price">${expenseAmount}</div>
      </div>
    </div>
  );
}

export default ExpenseItem;
```

- props: properties. 속성을 추가해 사용자 지정 컴포넌트에 데이터를 전달한다. 컴포넌트 안에서 속성에 접근할 수 있다.

App.js

```javascript
import ExpenseItem from "./components/ExpenseItem";

function App() {
  const expenses = [
    {
      id: "e1",
      title: "Toilet Paper",
      amount: 94.12,
      date: new Date(2020, 7, 14),
    },
  ];
  return (
    <div>
      <h2>Let's get started!</h2>
      <ExpenseItem
        title={expenses[0].title}
        amount={expenses[0].amount}
        date={expenses[0].date}
      ></ExpenseItem>
    </div>
  );
}

export default App;
```



ExpenseItem.js

```javascript
import ExpenseDate from "./ExpenseDate";
import "./ExpenseItem.css";

function ExpenseItem(props) {
  return (
    <div className="expense-item">
        <ExpenseDate date={props.date}/>
      <div className="expense-item__description">
        <h2>{props.title}</h2>
        <div className="expense-item__price">${props.amount}</div>
      </div>
    </div>
  );
}

export default ExpenseItem;
```



ExpenseDate.js

```javascript
import './ExpenseDate.css';

function ExpenseDate(props) {
  const month = props.date.toLocaleString("en-US", { month: "long" });
  const day = props.date.toLocaleString("en-US", { month: "2-digit" });
  const year = props.date.getFullYear();

  return (
    <div>
      <div className="expense-date__month">{month}</div>
      <div className="expense-date__year">{year}</div>
      <div className="expense-date__day">{day}</div>
    </div>
  );
}

export default ExpenseDate;
```



- 컴포지션: 작은 빌딩으로부터 유저 인터페이스를 구축하는 접근 과정. 

- 현재 아이템들이 들어가는 각각의 카드와 그 카드들을 전부 가지고 있는 카드는 그림자, 둘레 등이 상당히 유사하게 생겼다. 이런 경우 각각의 css에서 공통되는 부분을 꺼내서 하나의 컴포넌트를 만들어 적용시킬 수 있다.

ExpenseItem.js

- 아이템이 들어간 개별 컴포넌트
- return 의 div를 공통 css를 적용한 Card 컴포넌트로 변경해 컨텐츠를 감싸는 래퍼로 사용한다. 하지만 사용자 정의 컴포넌트에서는 안의 컨텐츠 내용이 보이지 않는다.

```react
import Card from "./Card";
import ExpenseDate from "./ExpenseDate";
import "./ExpenseItem.css";

function ExpenseItem(props) {
  return (
    <Card className="expense-item">  
        <ExpenseDate date={props.date}/>
      <div className="expense-item__description">
        <h2>{props.title}</h2>
        <div className="expense-item__price">${props.amount}</div>
      </div>
    </Card>
  );
}

export default ExpenseItem;
```

Expense.js

- 모든 아이템이 들어간 컴포넌트
- 마찬가지로 div를 Card 컴포넌트로 변경했는데 div같은 디폴트 html에서는 className이 제대로 작동하지만 사용자 정의 컴포넌트에서는 적용되지 않는다.

```react
import Card from "./Card";
import ExpenseItem from "./ExpenseItem";
import "./Expenses.css"

function Expense(props) {
  return (
    <Card className="expenses">
      <ExpenseItem
        title={props.expenses[0].title}
        amount={props.expenses[0].amount}
        date={props.expenses[0].date}
      />
		...
    </Card>
  );
}

export default Expense;
```

Card.js

- 1번 문제를 위해서는 `모든 컴포넌트가 가지는 리액트 내장 props를 사용한다.` props를 매개변수로 받고 그 안의 children으로 컨텐츠를 보이게 한다.
- 그리고 2번 문제를 위해 className역시 props에서 가져와 적용해주도록 한다.

```react
function Card(props) { 
    const classes = 'card ' + props.className;
    return < div className={classes}>{props.children}</div>
}

export default Card;
```



- 브라우저의 개발자 도구에서는 jsx가 아닌 변환된 코드만 볼 수 있다. 
- package.json은 대부분 화면 뒷단의 변환을 다루지만 react, react-dom은 리액트와 관련있는 요소다. 그런데 react-dom은 index.js에서 사용되지만 react는 아무 곳에서도 import되지 않고 있다. 과거 리액트 프로젝트는 JSX파일을 사용하는 모든 파일에 react를 임포트했지만 최신 리액트 셋업에서는 생략 가능하다. 
- react를 import해서 React의 createElement요소를 return 하도록 코드를 변경하면 루트 JSX요소에 왜 루트 엘리먼트가 필요한지 명확해진다. createElement에서는 하나 이상의 나란히 있는 요소들이기 때문에 return이 불가능해 래퍼가 필요하다.

App.js

```react
...
return React.createElement(
    "div",
    {},
    React.createElement("h2", {}, "Let's get started!"),
    React.createElement(Expense, { expenses: expenses })
  );

  // return (
  //   <div>
  //     <h2>Let's get started!</h2>
  //       <Expense expenses = {expenses} />
  //   </div>
  // );
```



- 컴포넌트 정리하기: src/components에 모두 넣지 말고 변경해줬다.
  - 앱의 특정 기능과 관련 없는 일반적인 사용자 인터페이스 컴포넌트: UI
  - 특정 기능을 가진 컴포넌트: Expenses



- function대신 화살표 함수 사용하기

```react
function App() {}

const App = () => {}
```



## 섹션 4: 리액트 State 및 이벤트 다루기

```react
const ExpenseItem = (props) => {
  const clickHandler = () => {
    console.log("clicked");
  }

// return 
<button onClick={() => {console.log('Clicked@')}}>Change Title</button>

<button onClick={clickHandler}>Change Title</button>
```

- onClick에서 바로 화살표 함수를 넣을 수 있지만 소스를 깔끔하게 유지하고 JSX에 너무 많은 로직이 들어가지 않게 분리하면 좋다. 
- ClickHander에서 ()를 사용하지 않아 지정만하고 실행은 하지 않도록 했다. 실행되도록 하면 코드를 분석할 때 js가 실행되고 jsx를 반환할 때 전체 코드를 분석한다. 그래서 클릭했을 때가 아닌 코드 분석시 실행이 된다.
- 지원되는 이벤트 리스너를 추가되면 일부는 특정 요소에서만 가능하지만 모두 기본적으로 DOM 동작에 기반한다.
- 컴포넌트는 함수형으로 jsx를 반환하는데 우리가 컴포넌트를 사용할 때 react가 인식해 호출하는 것이다. 그런데 리액트는 처음 호출한 후 반복하지 않아 데이터를 업데이트 하고싶을 때 재평가를 위해 `state`라는 개념을 도입했다.

ExpenseItem.js

- React에서 useState라는 특정 import를 하는데 이것을 named import라고 한다.
- 리액트 훅: 버튼을 클릭하면 데이터를 바꾸고 싶을 때 **리액트 훅**을 사용한다. 훅은 보통 use로 시작하며 리액트 컴포넌트 함수 안에서 직접 호출되어야 한다. 예외도 존재하지만 일단 그렇다. 
- useState: 특별한 값을 반환해 변수에 새로운 값을 할당하기 위해 사용한다. 변수를 넘겨주면 배열에서 첫 번째로 변수 그 자체인 관리되는 값, 현재 상태값을 반환하고 두 번째는 그걸 업데이트 하는 함수를 반환한다. 관례상 첫 값이 변수고 두 번째가 그걸 세팅하는 것이기 때문에 set변수로 이름짓는다.
- set함수를 호출하면 바로 새 값을 할당하는게 아니라 useState로 상태를 초기화한 곳에서 다시 실행된다. state가 변할 때 업데이트 함수를 호출하고 새 값을 할당하고 싶어  react에게 useState로 재평가하라고 한다. 그러면 react가 재실행, 재평가하고 감지된 변화가 화면에 보인다.
- 그런데 console.log를 통해 확인하면 title은 예전 값이 그대로다. 함수 호출 시 값을 바꾸는 게 아니라 state의 업데이트를 예약해 새로운 값이 적용되지 않았다.
- 현재 ExpenseItem은 Expense에서 4번 호출되었고 각각 다른 상태와 분리되어 있다. 4번 호출되며` 각각의 state가 별도 생성되며 리액트에 의해 독립적으로 관리된다. `
- setTitle로 title에 새로운 값이 할당될텐데 const를 사용하는 이유는 `우리가 등호로 값을 할당하는게 아니라는 것`이다. state업데이트하는 함수를 호출해 구체적인 값은 리액트에 의해 관리된다. 변수 자체는 우리에게 보이지 않고 함수만 호출한다.

```react
import React, { useState } from 'react';
const ExpenseItem = (props) => { 

  const [title, setTitle] = useState(props.title);  

  const clickHandler = () => {
    setTitle('Updated!');
      console.log(title);
  }
  return (
      <div className="expense-item__description">
        <h2>{title}</h2>
      	<button onClick={clickHandler}>Change Title</button> 
      </div>
  );
}

export default ExpenseItem;
```

