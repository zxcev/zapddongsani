# 프론트엔드를 위한 class 사용법 및 객체지향 1

대부분의 프론트엔드 개발자 분들은 JS로 시작해서 TS에 정착했거나 아직 TS를 만나기 전일 것입니다.

특별한 경우가 아니라면 React를 메인 기술 스택으로 잡으셨을텐데,

웬만하면 `class`를 사용할 일이 아예 없을 것이라고 생각합니다.

React가 `class` 컴포넌트에서 함수형 컴포넌트로 전환한지 오래 되었고,

여전히 `class` 컴포넌트를 사용할 수야 있다지만 거의 사장되었기 때문입니다.

React를 배우다보면 `class` 자체를 사용하지 않다보니 개념만 잡고 넘어가는 경우가 많은 것 같고,

공부할 필요성도 크게 못 느꼈을 것 같습니다.

저도 백엔드 개발자로 굳히긴 했지만 처음에 JS로 프로그래밍 공부를 시작했었는데,

`class`를 거의 사용하지 않다보니 존재만 알 뿐, 이해도도 낮고 심지어 문법도 사용할 때마다 헷갈리는 상태였습니다.

심지어 JS의 `class`는 사실 다른 언어의 `class`와 전혀 다른, `class`를 흉내낸 것이라는 설명을 보니 더욱 정이 떨어졌습니다.

다른 언어를 알지도 못하는데, 그걸 이해하려면 다른 언어의 `class`를 공부한 뒤 JS의 `class`를 공부 하란건가 라는 생각이 들었으니까요.

그런데 React에서는 실제로 사용하지도 않기 때문에 시간 낭비라는 생각이 들었습니다.

이런 여러 프론트의 내부 사정 때문에 처음 프로그래밍을 시작하던 시절 `class` 학습은 거의 포기했었는데,

저와 비슷한 분들이 분명 있을 것 같아서 JS의 `class` 원리와 사용법에 대한 글을 작성해보려고 합니다.

어쨌든 객체지향은 현재 프로그래밍 세계에서 가장 최전선에 있는 주류 메타이고 nextjs가 ssr, ssg, isr 등을 도입하면서 프론트가 프론트가 아닌 html 렌더링 서버가 되어 어느 정도 고급 개발자가 되기 위해서 분명 백엔드 학습도 필요할텐데, 그 때 nestjs 등을 학습하기 위해서는 객체지향과 JS의 `class`는 반드시 넘어야 할 벽입니다.

게다가 지금 당장 눈 앞에 있는 우아한테크코스를 통과하기 위해서도 반드시 학습해야 하는 존재라고 생각해주세요.

이렇게 쓸데없이 머릿말을 길게 쓰는 이유는 쓸모없다고 생각하면 애초에 시간 낭비라는 부정적인 생각과 대충 쓰고 버리겠다는 마인드로 접근하게 되고 깊은 지식을 쌓는데 심적 장벽이 된다는 것을 스스로 겪었기 때문입니다.

글이 너무 길면 집중도가 떨어지기 때문에 짧게 나눠서 여러 편으로 작성해보겠습니다.

# JS의 객체

```js
const person = {};

person['name'] = 'abc';
person['age'] = 10;
```

JS의 객체는 매우 자유롭습니다.
빈 객체를 만든 뒤, 원하는 `key-value`를 원하는만큼 추가하면 됩니다.

```js
const person = {
	name: 'abc',
	age: 10,
};
```

또는 객체를 생성할 때, 모든 필드를 명시해도 됩니다.

대부분 이렇게 사용하실겁니다.

그런데 만약 똑같은 객체를 여러 파일에서 사용해야 한다면 이 방법은 좋지 않을 것입니다.

```js
// file1.js
const person1 = {
	name: 'x',
	age: 32,
};

// file2.js
const person2 = {
	name: 'ox',
	age: 44,
};

// file3.js
const person3 = {
	name: 'xxx',
	age: 13,
};

// ...

// file100.js
const person100 = {
	name: 'xox',
	age: 54,
};
```

아직 별로 문제가 느껴지지 않으신다면,
필드의 개수를 15개로 늘려보겠습니다.

```js
// file1.js
const person1 = {
	name: 'x',
	age: 32,
	birth: ...,
	gender: ...,
	locale: ...,
	email: ...,
	id: ...,
	phone: ...,
	school: ...,
	address: ...,
	friends: ...,
	posts: ...,
	home: ...,
	firstName: ...,
	lastName: ...,
};

// file2.js
const person2 = {
	name: 'ox',
	age: 44,
	birth: ...,
	gender: ...,
	locale: ...,
	email: ...,
	id: ...,
	phone: ...,
	school: ...,
	address: ...,
	friends: ...,
	posts: ...,
	home: ...,
	firstName: ...,
	lastName: ...,
};

// file3.js
const person3 = {
	name: 'xxx',
	age: 13,
	birth: ...,
	gender: ...,
	locale: ...,
	email: ...,
	id: ...,
	phone: ...,
	school: ...,
	address: ...,
	friends: ...,
	posts: ...,
	home: ...,
	firstName: ...,
	lastName: ...,
};

// ...

// file100.js
const person100 = {
	name: 'xox',
	age: 54,
	birth: ...,
	gender: ...,
	locale: ...,
	email: ...,
	id: ...,
	phone: ...,
	school: ...,
	address: ...,
	friends: ...,
	posts: ...,
	home: ...,
	firstName: ...,
	lastName: ...,
};
```

복붙으로 필드명을 정확히 일치하도록 만들었습니다.

100개의 파일에요.

여러 가지 문제가 있는데,

우선 객체가 무엇을 의미하는지 **정확히 추론하기가** 어렵습니다.

우리는 필드명만 가지고 있기 때문에 이 객체가 의미하는 '대상'을 모릅니다.
이게 `Person`인지 `Student`인지 정확히 구분하려면 문서를 뒤져보거나 해야 할 것입니다.
변수명이 `person`이니 변수명으로 구분하면 된다구요?

```js
// ...

// file100.js
const sam = {
	name: 'xox',
	age: 54,
	birth: ...,
	gender: ...,
	locale: ...,
	email: ...,
	id: ...,
	phone: ...,
	school: ...,
	address: ...,
	friends: ...,
	posts: ...,
	home: ...,
	firstName: ...,
	lastName: ...,
};
```

만약 변수명이 위와 같다면 어떻게 하시겠습니까.

우리가 만드는 서비스가 `Student`, `Teacher`, `Parent` 등 여러 가지 사람을 의미하는 객체가 공존한다면요.

`Student`는 `school`, `Teacher`는 `job`, `Parent`는 `child`라는 미묘한 필드의 차이가 있을 뿐, 나머지 14개의 필드가 같다면 구분하기 매우 어려울 것입니다.

객체 하나를 보더라도 필드를 세세히 살펴야 할 것이며, 결코 한 눈에 알아볼 수 없겠죠.

사소한 것에 큰 에너지를 소모해야 하여 비효율적입니다.

객체가 아닌, 클래스를 사용하면 이를 어느 정도 해소할 수 있습니다.

```js
class Teacher {
	constructor(name, age, job) {
		this.name = name;
		this.age = age;
		this.job = job;
	}
}
// -객체 생성 방법
// const teacher = new Teacher('abc', 32, 'teacher');

// - 생성되는 객체
// { name: 'abc', age: 32, job: 'teacher' }

class Student {
	constructor(name, age, school) {
		this.name = name;
		this.age = age;
		this.school = school;
	}
}
// -객체 생성 방법
// const student = new Student('xxx', 13, 'Gangnam School');

// - 생성되는 객체
// { name: 'xxx', age: 13, school: 'Gangnam School' }

class Parent {
	constructor(name, age, child) {
		this.name = name;
		this.age = age;
		this.child = child;
	}
}
// -객체 생성 방법
// const parent = new Parent('xyz', 44, 'xxx');

// - 생성되는 객체
// { name: 'xyz', age: 44, child: 'xxx' }
```

이제 `Teacher` 객체를 만들 때는 `new Teacher`로,

`Parent`는 `new Parent`,

`Student`는 `new Student`를 호출해서 객체를 생성하면 되기 때문에

어떤 객체가 만들어지는지 한 눈에 볼 수 있습니다.

즉, 클래스는 같은 종류의 객체를 여럿 찍어내기 위한 '설계도'입니다.

`constructor`는 생성자라고 하며,

객체를 생성할 때 호출되는 함수입니다.

사실 JS의 클래스는 다음과 같이 함수로 표현할 수도 있습니다.

```js
// class는 결국 객체를 생성하는 함수
function Student(name, age, school) {
	this.name = name;
	this.age = age;
	this.school = school;
}

// 객체 생성
const student = new Student('xxx', 13, 'Gangnam School');
```

JS의 class는 결국 정해진 틀대로 객체를 생성하는 함수에 불과한 것이죠.

`class`를 사용할 때와 사용법도 똑같으며, 만들어지는 객체도 똑같습니다.

그런데 반환값이 없어서 의아하실 겁니다.

```js
// 첫 번째 인자로 this가 숨겨져있다.
function Student(this, name, age, school) {
	this.name = name;
	this.age = age;
	this.school = school;
	// 그런데 반환값이 없다.
}

// 이게 원래 문법이다.
const student = new Student(name, age, school);

// 사실 빈 객체를 생성하고 이를 첫 번째 인자로 넘긴 뒤,
// 함수 내에서 해당 빈 객체에 필드를 넣어주는 것이라고 생각해도 무방하다.
const student = {};
Student(student, name, age, school);
```

주석에 쓰여진 그대로입니다.

```js
const student = new Student(name, age, school);
```

지금까지는 문법상 객체 생성을 이렇게 했기 때문에 모르셨을 수도 있지만,

빈 객체를 먼저 생성하고 첫 번째 인자로 그 객체를 전달해서

원본을 변경하는 것이 숨겨진 동작이고,

그 때문에 `this`를 만들지도 않았는데 함수 안에 `this`가 존재했던 것이고,

그 때문에 반환이 필요 없었던 것입니다.

대부분의 언어에서 저런 식으로 동작하는데 JS를 자세히는 모르기 때문에

내부 동작이 좀 다를 수도 있지만 비슷할 것이라 예상하여

`this`와 반환이 없는 것을 이해하는데는 도움이 될 것이라 생각합니다.

JS의 함수 호출은 2가지 방식이 있으며,

`new`를 붙인 함수 호출은 앞서 설명했듯 빈 객체를 생성하고 첫 번째 인자로 넘겨서 필드를 채워 넣는다.

`new`를 붙이지 않으면 알고 있으신대로 일반적인 함수 호출이 된다.

JS의 `class`는 함수를 대부분의 사람들이 여타 Java, C# 등의 객체 지향 언어의 문법에 너무 익숙해졌기 때문에

편의를 위해 `new` 키워드로 함수 호출 시, 객체 생성하는 함수를 `class` 처럼 보이는 문법으로 만들었고,

`class`의 생성자(`constructor`)가 바로 객체 생성하는 함수의 바디가 들어가는 부분이다. 라는 것입니다.

ES5까지는 `class` 문법 자체가 존재하지 않았기에 함수를 사용해서 객체를 생성할 수 있으며,

`class`는 ES6(ES2015)부터 지원됩니다.

```js
class Person1 {
	// 클래스명이 결국 함수명이 되고, 생성자의 인자와 내부 로직이 그대로 함수 인자 및 바디가 된다.
	constructor(name, age) {
		this.name = name;
		this.age = age;
	}
}


function Person2(name, age) {
	this.name = name;
	this.age = age;
}

const person1 = new Person1('abc', 1);
const person2 = new Person2('xxx', 2);

// 둘 다 사용법, 동작이 동일하며 반환되는 객체는
// person1 -> { name: 'abc', age: 1 };
// person2 -> { name: 'xxx', age: 2 };
```

피드백이나 잘못 알고 있는 부분이 있다면 지적 환영합니다.

시간 날 때마다 천천히 추가 내용을 작성해보겠습니다.
