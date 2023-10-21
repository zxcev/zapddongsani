# 프론트를 위한 JS Class 원리3

## class와 `__proto__` 파헤치기

지난 시간 마지막에 봤던 `__proto__`에 익숙하지 않은 분들에게 두려운 존재로 남을 것 같아서

`class`와 `__proto__`를 직관적으로 받아들일 수 있도록 보여드리겠습니다.

아래와 같이 클래스를 작성하겠습니다.

```js
class Person {
	constructor(name, age) {
		this.name = name;
		this.age = age;
	}

	eat() {
		console.log(`${this.name} is eating`);
	}

	sleep() {
		console.log(`${this.name} is sleeping`);
	}

	walk() {
		console.log(`${this.name} is walking`);
	}
}
```

위 코드를 그대로 복사하여 크롬 콘솔에 붙여넣어 실행하였습니다.

![](/imgs/front_oop3_1.png)

이제 `Person` 객체를 크롬 콘솔에서 만들 수 있게 되었겠지요.

![](/imgs/front_oop3_2.png)

객체를 만들고 살펴보면, `[[Prototype]]`이라는 속성이 보이실 겁니다.

지금은 이 `[[Prototype]]`이 바로 전 시간에 봤던 `__proto__`와 동일하다고 생각해주세요.
(약간 다르지만 이해를 위한 배경 지식들을 먼저 보고 이후에 좀 더 보충 설명하겠습니다.)

`new Person`으로 생성한 객체는 다음과 같이 표현할 수 있습니다.

```js
const person = new Person('a', 1);

// person
{
	name: 'a',
	age: 1,
	__proto__: {
		constructor: ...,
		eat: ...,
		sleep: ...,
		walk: ...,
	}
}
```

그렇다면 `new Person`으로 만든 객체의 `__proto__`는 다음과 같이 표현되겠지요.

```js
// person.__proto__
{
	constructor: ...,
	eat: ...,
	sleep: ...,
	walk: ...,
}
```

만약 `class Person`의 생성자 및 메소드의 내용을 생략한다면 다음과 같을 것인데,

```js
class Person {
	constructor(name, age) {}
	eat() {}
	sleep() {}
	walk() {}
}
```

`__proto__`와 `class`의 구조가 완전히 일치하는 것을 볼 수 있습니다.

이로 말미암아 `class`는 생성자, 메소드를 가지며,

`new <Class명>`으로 호출 시, `constructor`가 호출되며,

`__proto__`가 `constructor` 및 해당 클래스의 메소드를 모두 가진 객체를 만들어서

생성자에서 속성을 하나하나 넣어준다는 절차를 파악하실 수 있습니다.

지금까지 공부한 내용을 총정리한다고 생각하고 코드로 살펴보겠습니다.

```js
// 1. 원래 문법
const person = new Person('a', 1);


// ---------------------------------


// 2. 실제 일어나는 과정 및 결과
const person = {
	__proto__: {
		constructor: /* Person의 constructor */,
		eat: /* Person의 메소드 */,
		sleep: /* Person의 메소드 */,
		walk: /* Person의 메소드 */,
	},
};

// Person의 constructor를 호출하여 위에서 생성한 객체에 필드를 추가
// 첫 번째로 들어가는 person이 바로 this
Person.constructor(person, 'a', 1);

// 생성된 최종 person 객체
{
	name: 'a',
	age: 1,
	__proto__: {
		constructor: /* Person의 constructor */,
		eat: /* Person의 메소드 */,
		sleep: /* Person의 메소드 */,
		walk: /* Person의 메소드 */,
	},
}
```

위와 같이 객체 생성 과정 시 실제로 일어나는 일들을 정리해 보았습니다.

`class`를 좀 더 깊게 이해하는데 도움이 되었으면 좋겠네요.

---
## 접근 제어자(Access Modifier)

```js
class Person {
	constructor(name, age) {
		validateName(name);
		validateAge(age);
		this.name = name;
		this.age = age;
	}

	validateName(name) {
		if (typeof name !== 'string' || name.length < 1) {
			throw new Error('이름은 반드시 1자 이상의 문자여야 합니다.');
		}
	}

	validateAge(age) {
		if (age < 0) {
			throw new Error('나이는 음수가 될 수 없습니다.');
		}
	}
}
```

위와 같이 `Person` 클래스를 만들고, 이름과 나이를 생성 시 검증해주는 것으로

유효한 데이터를 갖는 객체를 만들 수 있었습니다.

만약 유효하지 않은 데이터(나이에 음수 등)를 넣는다면 예외가 발생합니다.

그런데 현재는 데이터 유효성을 오직 생성할 때만 검사하고 있을 뿐,

변경할 때는 전혀 검사하고 있지 않습니다.

```js
const person = new Person('abc', 1);
// 변경할 때는 유효성 검사가 일어나지 않기 때문에
// age를 음수로 변경할 수 있음
person.age = -1;
```

객체는 생성부터 소멸 시점까지 유효한 데이터를 가져야 합니다.

현실 세계에서도 사람이 태어날 때 이미 키가 수십cm이며, 몸무게도 3kg이고,

죽을 때까지 키와 몸무게가 음수가 되는 일은 결코 없습니다.

`Person` 객체도 이와 마찬가지로 중간에 `age`가 음수가 되는 일은 없어야 할 것입니다.

이를 위해서는 `age`를 변경할 때도 검증을 해줘야 합니다.

## private access modifier

```js
// Person Class 외부에서는 #name, #age에 접근할 수 없음
class Person {
	// private 필드는 명시 필요
	#name
	#age

	constructor(name, age) {
		// constructor는 Person Class 내부에 존재하므로 접근 가능
		this.#name = name;
		this.#age = age;
	}
}
```

ES2019부터 도입된 `#` 접근 제어자를 사용하면, 클래스 외부에서 필드에 접근할 수 없게 됩니다.
![](/imgs/front_oop3_3.png)
실제로 VSCode에서 실행해보면 접근 및 수정이 불가능한 것을 확인할 수 있습니다.

읽고 쓰기가 모두 불가능한 상태가 된 것인데,

그럼 `name`과 `age`가 뭔지 볼 수가 없는데 무슨 의미가 있나 싶을 수도 있습니다.

```js
class Person {
	#name
	#age

	constructor(name, age) {
		this.#name = name;
		this.#age = age;
	}

	getName() {
		return this.#name;
	}

	getAge() {
		return this.#age;
	}
}
```

그래서 나온 것이 `getter`입니다.

전혀 특별할 것 없는 `getXXX`라는 네이밍 컨벤션을 가진 메소드를 만들어서

`class` 외부에서 접근할 수 없는 필드를 그대로 반환하는 것입니다.

즉, `getter`만 생성한 `private` 필드의 경우 필드가 아닌 메소드를 통해 필드의 값을 읽을 수 있으며,

변경은 불가능한, 조회만 가능한 필드가 됩니다.

```js
const person = new Person("abc", 1);
const name = person.getName(); // 'abc'
const age = person.getAge();   // 1
```

이렇게 사용하면 되겠습니다.

`getter`가 조회라면, `setter`는 변경을 위한 메소드입니다.

역시 전혀 특별할 것 없는 일반적인 메소드일 뿐이며,

아래와 같이 작성할 수 있습니다.

```js
  class Person {
	#name
	#age

	// ...
	// constructor, getter, validate 메소드 생략

	setName(name) {
		// name을 변경할 때마다 유효한 데이터인지 검증
		validateName(name);
	    this.#name = name;
	}
	
	setAge(age) {
		// age를 변경할 때마다 유효한 데이터인지 검증
		validateAge(age);
	    this.#age = age;
	}
}
```

그러나 `setter`를 호출해서 변경을 하게 되면 변경 전에 유효성 검증 로직을 삽입할 수 있습니다.

```js
const person = new Person("abc", 1);
person.setAge(-1);
```

`setAge`로 음수를 삽입해 보겠습니다.

![](/imgs/front_oop3_4.png)

예외가 던져져서 유효하지 않은 데이터가 입력 되었음을 알려줍니다.

`constructor`, `access modifier`를 사용하여 이제 `Person` 객체는 생성 시부터 소멸 시까지 유효한 데이터를 가질 수 있게 된 것입니다.

![](/imgs/front_oop3_5.png)

`private` 필드는 `class` 최상단에 명시하지 않으면 VSCode가 오류를 알려줍니다.

`#`(private access modifier)을 붙이지 않은 필드는 기본적으로 `public` 필드가 되는데,

`public`이란 것은 `class` 블록 외부에서도 접근할 수 있음을 뜻합니다.

그리고 `public` 필드 역시 `private` 필드처럼 명시해줄 수 있습니다.

안해도 오류가 나지 않을 뿐입니다.

```js
class Person {
	name;
	age;

	constructor(name, age) {
		this.name = name;
		this.age = age;
	}
}
```

다음 시간에는 `getter`와 `setter`에 대한 JS의 문법 지원과 이를 좀 더 자세히 알아보도록 하겠습니다.
