# 프론트를 위한 JS Class 원리2

## 객체에 동작 추가하기(메소드)

지난 시간에 만들었던 `person`에 '먹는다'는 동작을 추가하고 싶다면

필드로 함수를 추가하면 됩니다.

```js
const person = {
	name: 'abc',
	age: 12,
	eat: function() {
		console.log('eating...');
	},
};
```

메소드 축약 표현을 사용해도 되겠죠.

```js
const person = {
	name: 'abc',
	age: 12,
	eat() {
		console.log('eating...');
	},
};
```

이렇게 말입니다.

그런데 앞서 봤었던 것처럼, 객체 리터럴( `{}` ) 방식으로 객체를 생성하는 것은 문제가 많았습니다.

필드의 개수가 늘어나고, 비슷한 필드를 가진 객체가 많아질수록 한 눈에 파악하기 어려웠죠.

여기에 행위까지 추가된다면 문제는 더욱 심각해집니다.

```js
const person = {
	name: 'abc',
	age: 12,
	eat() {
		// 100줄이라면.
	},
};
```

극단적인 예시지만, 만약 `eat`이 100줄짜리 함수라고 상상해봅시다.

그리고 이 객체가 100개의 파일에서 사용된다면.

```js
// file1.js
const sam = {
	name: 'sam',
	age: 13,
	eat() {
		// 100줄이라면.
	},
};

// file2.js
const ben = {
	name: 'ben',
	age: 55,
	eat() {
		// 100줄이라면.
	},
};

// file3.js
const huge = {
	name: 'huge',
	age: 41,
	eat() {
		// 100줄이라면.
	},
};

// ...

// file100.js
const ava = {
	name: 'ava',
	age: 33,
	eat() {
		// 100줄이라면.
	},
};
```

이전보다 훨씬 더 크게 문제가 됩니다.

만약 함수 내용이 바뀐다면 100개의 파일에서 함수를 전부 수정해야 할 것 입니다.

물론 이건 너무 극단적인 예시고, 함수를 만들어서 공유하는 방법도 있긴 합니다.

```js
// eat.js
export function eat() {
    console.log(this.name + 'is eating');
}

// file100.js
import { eat } from './methods/eat';

const ava = {
	name: 'ava',
	age: 33,
	eat,
};
```

이러면 100개의 파일에서 함수를 공유하면 되기 때문에 

함수가 수정될 경우, 100개의 변경 지점이 1개로 줄어들긴 했지만,

일반 함수에서 `this`를 쓰는 문법이 매우 낯설어 보일 것이며,

여러 파일에서 객체에 넣기 위한 함수를 가져와야 되서 `import`도 꼬이게 될 가능성이 있습니다.

**가장 큰 문제는 아까처럼 어떤 객체를 의미하는지 한 눈에 파악할 수 없다는 것과,**

필드가 너무 많아져서 실수로 누락한다면?
(필드 개수가 늘어날수록 실수할 확률은 늘어납니다.)

이 때도 역시 `class`로 문제를 해결할 수 있습니다.

```js
class Person {
	constructor(name, age) {
		this.name = name;
		this.age = age;
	}

	eat() {
		console.log(this.name + 'is eating');
	}
}

const person = new Person('abc', 1);
person.eat();
```

이렇게 객체가 들고 있는 함수를 메소드라고 부르는데,

메소드는 필드와 달리 `new`로 객체를 생성할 때 인자로 전달할 필요도 없습니다.

메소드도 결국 함수이기 때문에 함수의 '주소'를 담고 있으며,

하나의 함수를 `new Person` 으로 만든 모든 `Person` 객체가 공유하게 되기 때문입니다.

그래서 `walk`, `run` 등의 메소드를 추가하거나 `eat`을 제거하더라도 `Person`을 생성하는 100개의 파일에서 변경 사항이 일어날 필요가 애초에 없습니다.

```js
class Person {
	constructor(name, age) {
		this.name = name;
		this.age = age;
	}

	eat(how) {
		console.log(this.name + 'is eating ' + how);
	}
}

const person = new Person('abc', 1);
person.eat('quickly'); // 'abc is eating quickly'
```

물론 `eat`의 인자의 개수가 변하는 경우는 100개의 파일에서 인자에 값을 전달 해야겠죠.

이 부분은 어쩔 수 없기 때문에 처음부터 함수 설계를 잘 해서 해결해야 할 부분일듯 싶습니다.

---

## 객체의 유효성

객체는 항상 생성 시부터 소멸 시까지 유효한 상태를 가져야 합니다.

이에 대한 자세한 설명은 [객체 지향과 4대 특성](https://www.mainfn.dev/184fb1c9-5cf1-4fff-a692-5dce63408146) 포스트에 작성해두었습니다.

Java로 작성되었으나, 현재 Java를 모르더라도 이후 작성할 TypeScript의 `class`까지 읽고 나면 어렵지 않게 읽을 수 있을 것이므로 나중에 한 번 읽어보시는 것을 추천드립니다.

간단하게 요약하면,

객체지향에서 '객체'라는 것은 현실 세계의 사물을 코드로 옮긴 것이며,

'사물'이라고 하면 무생물을 뜻하는 것 같지만, 객체지향에서는 '생물'조차도 객체에 포함됩니다.

그래서 `Person`이라는 객체를 만들었던 것입니다.

현실 세계의 객체는 속성과 행동을 가집니다.

예를 들어, 사람은 이름, 나이, 키 등의 속성이 존재하고,

먹다, 자다, 걷다 등의 행동을 할 수 있습니다.

이를 코드로 옮기면 아래와 같습니다.

```js
class Person {
	constructor(name, age, height) {
		this.name = name;
		this.age = age;
		this.height = height;
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

// 객체 생성
const a = new Person('a', 10, 150);

a.eat(); //     'a is eating'
a.sleep(); //   'a is sleeping'
a.walk(); //    'a is walking'
```

지금까지 배운 바로는 `Person`이라는 `class`는 결국 함수에 불과했으며,

`new`라는 키워드를 붙이면 다음과 같이 빈 객체를 만들어주고,

첫 번째 인자로 빈 객체를 넣어서 `constructor` 내에서 초기화를 진행했습니다.

```js
// 실제 문법
const a = new Person(a, 'a', 10, 150);

// 내부적인 동작
const a = {};
Person(a, 'a', 10, 150);
```

다른 언어에서 이미 `class`로 객체를 생성하는 문법이 너무 대중화 되었기 때문에,

JS에서는 함수로 객체를 생성하던 방식을 '문법'만 `class`처럼 사용할 수 있게 만든 것이었습니다.

ES5까지는 아래와 같은 방식이 `class`와 완전히 동일한 객체 생성 함수라고 보면 됩니다.

```js
function Person(name, age, height) {
	this.name = name;
	this.age = age;
	this.height = height;
	
	this.eat = function() {
		console.log(`${this.name} is eating`);
	}

	this.sleep = function() {
		console.log(`${this.name} is sleeping`);
	}

	this.walk = function() {
		console.log(`${this.name} is walking`);
	}
}

// 객체 생성
const a = new Person('a', 10, 150);

a.eat(); //     'a is eating'
a.sleep(); //   'a is sleeping'
a.walk(); //    'a is walking'
```

중요한 내용이기 때문에 한 번 다시 정리를 해봤습니다.

그럼 이제 본격적으로 객체의 유효성에 대해 알아볼텐데,

다음과 같은 `Person` 객체는 유효한 데이터를 가질까요?

```js
const person = new Person('a', -100, -150);
```

현실 세계에서 `age`와 `height`는 음수가 될 수 없습니다.

고로 위 객체는 유효하지 않은 데이터를 가진다고 볼 수 있습니다.

```js
class Person {
	constructor(name, age, height) {
		// 유효성 검사
		if (age < 0) {
			throw new Error('나이는 반드시 양수가 되어야 합니다.');
		}
		if (height < 0) {
			throw new Error('키는 반드시 양수가 되어야 합니다.');
		}

		if (name !== typeof 'string' && name.length < 1) {
			throw new Error('유효하지 않은 이름입니다.');
		}

		this.name = name;
		this.age = age;
		this.height = height;
	}
}
```

이렇게 유효하지 않은 데이터를 갖는 객체가 생성되는 것을 막기 위해 생성자 내에서 검증 로직을 작성해주면 됩니다.

![[프론트를 위한 JS Class 원리2-1.png]]

이제 유효하지 않은 값을 입력할 경우, 검사가 이루어진 뒤 예외를 던지게 됩니다.

`new Person`을 호출한 쪽에서 예외를 `catch`하여 예외가 발생할 때마다 경고 메세지를 출력하고 재입력을 받도록 하는 등의 로직을 작성하면 되겠지요.

덕분에 `Person`이라는 객체는 항상 유효한 데이터를 가진 채로 생성됩니다.

그런데 필드가 많아진다면 생성자(constructor)에 너무 많은 로직이 들어가게 될 것입니다.

```js
class Person {
	constructor(name, age, height) {
		// 유효성 검사 함수 호출
		this.validate(name, age, height);
		this.name = name;
		this.age = age;
		this.height = height;
	}

	// 유효성 검사를 위한 로직 분리
	validate(name, age, height) {
		if (age < 0) {
			throw new Error('나이는 반드시 양수가 되어야 합니다.');
		}
		if (height < 0) {
			throw new Error('키는 반드시 양수가 되어야 합니다.');
		}

		if (name !== typeof 'string' && name.length < 1) {
			throw new Error('유효하지 않은 이름입니다.');
		}
	}
}
```

그래서 위와 같이 유효성 검사를 위한 로직을 또 다른 메소드로 뺄 수 있습니다.

위 코드를 실행해보면 잘 돌아가는 것을 볼 수 있으며,

`constructor` 는 생성만, `validate` 는 검증만, 각각 하나의 책임만 갖도록 분리되었습니다.

```js
class Person {
	constructor(name, age, height) {
		// this는 생성된 빈 객체가 첫 번째 인자로 들어오는 것이라고 설명했음.
		// 그런데 this.validate 안에 함수를 할당한 적이 없는데?
		this.validate(name, age, height);
	
		// ... 생략
	}

	// ... 생략
}
```

그런데 주석에 써있는대로 `this.validate`에 따로 함수를 할당한 적도 없는데 잘 돌아가는 것이 의아할 수 있습니다.

`constructor` 의 첫 번째 인자로 받는 `this`는 빈 객체라고 했지만,

사실은 `__proto__`라는 속성을 갖고 있습니다.

`a`라는 `Person`의 객체를 크롬 콘솔에서 생성한 결과는 다음과 같습니다.

![](/imgs/front_oop2_1.png)

`new Person`으로 생성했기 때문에 `a.__proto__`의 `constructor`는 `class Person`이라 적혀 있습니다.

그리고 밑에서 2번째 속성을 보면 `validate`가 들어가 있는 것을 볼 수 있습니다.

이전에 배웠던 지식을 좀 더 확장하면 아래와 같다고 볼 수 있습니다.

```js
// 객체 생성
const a = new Person('a', 1 ,1);



// 실제 일어나는 일(아래 모든 코드)
const a = {
	__proto__: {
		constructor: /* 이전에 만든 Person의 constructor */,
		validate: /* 이전에 만든 validate 함수 */,
	},
};
// 이제 Person의 constructor 함수 호출하여 a에 name, age, height 등의 데이터 넣어줌
Person(a, 'a', 1, 1);
```

이래서 메소드를 생성자에서 할당하지 않아도, 바로 사용할 수 있었던 것입니다.

`__proto__` 속성은 모든 객체에 존재하는 속성입니다.

지금은 이 객체가 어떤 `class`로 생성되었는지,(정확히는 특정 `class`의 `constructor`, 특정 객체 생성용 함수를 의미했었음)

어떤 메소드를 가지고 있는지에 대한 정보를 가지고 있는 속성이라고 생각해주세요.

그래서 우리 입장에서는 `person`라는 객체를 만들었으면 `person.eat()`라고 사용하는 것처럼 보이겠지만,

내부적으로는 `person.__proto__.eat()`과 같이 메소드를 호출해주는 것입니다.

그래야 우리가 일일이 `person`을 만들 때마다 할당할 필요 없이 모든 객체에서 같은 메소드를 공유할 수 있으며,

매번 이렇게 쓰기는 너무 장황해져서 편의를 위해 축약 표현을 사용하고 있는 것입니다.
