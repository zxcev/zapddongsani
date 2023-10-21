# 프론트를 위한 JS Class 원리4

## JS의 getter, setter 지원

지난 시간에 학습했던 `getter`, `setter`를 다시 코드로 구현해보겠습니다.

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

	setName(name) {
		// 검증 로직 생략
		this.#name = name;
	}

	setAge(age) {
		// 검증 로직 생략
		this.#age = age;
	}
}
```

`getter`, `setter`는 필드에 직접 접근하는 것이 아니라, 메소드를 통해 한 번 우회해서 읽고 쓰는 방법입니다.

객체 생성 시에는 `constructor`에서, 데이터 변경 시에는 `setter`에서 데이터가 유효한지 검증해서 객체가 생성부터 소멸 시점까지 유효함을 보장할 수 있었습니다.

```js
const person = new Person('abc', 1);

// 생략 되었지만 검증 로직 때문에 유효하지 않은 값(음수 나이)을 넣으면 예외 발생
person.setAge(-10);

// ok
person.setAge(10);
person.getAge(); // 10
```

JS를 비롯하여 C#, Kotlin 등의 객체지향 언어들은 `getter`, `setter`를 언어 차원에서 문법으로 제공하는데,

`getAge`, `setAge`를 언어 차원에서 지원하는 `getter`, `setter`로 변환하면 다음과 같습니다.

```js
class Person {
	#name
	#age

	constructor(name, age) {
		this.#name = name;
		this.#age = age;
	}

	// 함수명 앞에 get을 붙여주면 언어 차원에서 지원하는 getter 함수가 된다.
	get age() {
		return this.#age;
	}

	// 함수명 앞에 set을 붙여주면 언어 차원에서 지원하는 setter 함수가 된다.
	set age(age) {
		// 검증 로직 생략
		this.#age = age;
	}
}
```

`get`, `set`이 메소드명 앞에 붙었을 뿐, 뭐가 다른 것인지 의아할 것입니다.

원리는 동일하지만 사용법이 달라진다고 보면 됩니다.

```js
const person = new Person('a', 1);

// 마치 필드에 직접 접근하는 것처럼 사용
person.age = 10;

const age = person.age;

console.log(age); // 10
```

`get`, `set` 키워드로 생성한 `getter`, `setter`는 마치 필드에 직접 접근하는 것처럼 사용하면 됩니다.

필드에 접근하는 것처럼 보일 뿐 이전에 만들었던 `getter`, `setter` 함수 호출이라고 생각하시면 됩니다.

C#, Kotlin도 이런 식으로 `getter`, `setter`를 언어 차원에서 지원하며,

필드에 직접 접근하는 것처럼 보이지만 실은 내부적으로는 함수 호출입니다.

아래 작성한 C#의 getter, setter 코드를 한 번 살펴보세요.

JS와 조금 다르지만 거의 비슷합니다.

C#이 아마 getter, setter를 거의 처음으로 도입한 언어인 것으로 알고 있으며,

TypeScript를 이미 사용하고 있거나 이후 반드시 배우게 될 것인데,

C#도 같은 MS에서 만들었기 때문에 유사한 점이 많습니다.

```cs
// C#으로 작성된 코드
public class Person {
	// Person class의 필드
    private string name;
    private int age;

	// name의 getter, setter를 한 번에 선언
	// 필드명은 name, getter, setter은 Name으로 미묘하게 다름
	// 실제 필드는 name이지만 외부에서는 Name으로 우회하여 접근
    public string Name {
        get { return name; }
        set { name = value; }
    }

	// age의 getter, setter를 한 번에 선언
    public int Age {
	    // getter는 인자가 없다.
        get { return age; }
	    // setter는 인자가 필요.
        set {
	        // value라는 예약어가 인자로 주어진다고 보면 된다.
	        // value를 선언한 적이 없는데 잘 사용되고 있는 것을 볼 수 있다.
            if (value < 0) {
	            throw new ArgumentException("value는 음수일 수 없습니다.", nameof(value));
            }
            // 검증 성공 시 필드를 변경한다.
            age = value;
        }
    }
}

// main.cs
class Program {
    static void Main(string[] args) {
        // Person 클래스의 객체 생성
        Person person = new Person();

        // Name setter를 통해 name 필드 설정
        person.Name = "abc";

        // Age setter를 통해 age 필드 설정
        person.Age = 1;

		// getter를 통해 Name 출력
        Console.WriteLine("Name: " + person.Name);
        // getter를 통해 Age 출력
        Console.WriteLine("Age: " + person.Age);
    }
}

```

문법 상의 차이가 약간 있을 뿐, 역할과 사용법은 같습니다.

getter, setter는 함수처럼 만들되, 필드처럼 사용하면 됩니다.

C#은 상당히 선구적인 언어로, C#에 도입되는 기능은 다른 많은 언어들에 영감을 주어,

다른 언어에서도 이런 선구적인 기능들을 많이 도입했습니다.

예를 들어, JS의 async, await는 ES2017에서 도입 되었는데,

C#에서는 이미 2012년에 도입된 기능입니다.

이번에는 Kotlin 코드도 살짝 살펴보겠습니다.

getter, setter가 다양한 언어에서 사용되는 범용적인 문법이며,

언어가 달라도 유사한 부분이 많다는 것을 보여드리기 위함이니 가볍게 살펴만 봐주세요.

```kotlin
// Kotlin
class Person {  
    
	// age 필드 선언
    var age: Int 
	    // getter, field가 예약어임.
	    get() {
		    return field
	    }
	    // setter
	    // 역시 field가 예약어
        set(age: Int) {  
            if (age < 0) {  
                throw IllegalArgumentException("?")  
            }
            field = age  
        }  

	// name은 getter, setter를 설정하지 않았음
    val name: String
    
	constructor(name: String, age: Int) {  
        this.name = name  
        this.age = age  
    }

}
```

역시 거의 비슷합니다.

주석이 많아서 코드가 길어졌네요.

마지막으로 TypeScript도 한 번 살펴보겠습니다.

```ts
// TypeScript
class Person {
	// 필드
	// JS와 다르게 변수명에 #를 붙이지 않고
	// private이라는 키워드를 사용
	private name: string;
	private age: number;

	// 생성자는 타입이 붙는다는 것만 다름
	constructor(name: string, age: number) {
		this.name = name;
		this.age = age;
	}

	// getter도 리턴 타입이 붙는다는 것만 다름
	get name(): string {
		return this.name;
	}

	// getter도 리턴 타입이 붙는다는 것만 다름
	get age(): number {
		return this.age;
	}
}
```

`private` , 타입이 조금 추가된 것 빼고는 JS와 비슷합니다.

setter를 안 만든 이유는 다음과 같습니다.

![](/imgs/front_oop4_1.png)

필드명과 `getter`, `setter` 명이 같아버리면 필드명이 중복된다고 인식됩니다.

```ts
class Person {
	// 필드명에 _를 붙임
	private _name: string;
	private _age: number;

	constructor(name: string, age: number) {
		this._name = name;
	    this._age = age;
	}

	// getter, setter명은 _를 안붙임
	get name(): string {
		return this._name;
	}

	set name(name: string) {
	    this._name = name;
	}
}
```

위와 같이 필드명 앞에 `_`를 붙이고 `getter`, `setter`는 안 붙이는 것으로 해결됩니다.

JS 방식의 `private`은 필드명 앞에 `#`를 붙여서 getter, setter와 충돌이 생기지 않았는데,

TS를 쓰게 되면 나중에 이런 식으로 해결하시면 됩니다.

## Property VS Field

Java에서는 클래스 내부의 필드들을 필드, 혹은 멤버 변수라고 부릅니다.

그러나 TypeScript, C#, Kotlin에서는 Property(속성)이라고 부릅니다.

이러한 명칭의 차이가 발생하는 이유를 생각해보면

Java는 getter, setter를 순수 메소드로 구현하고,

TS, C#, Kotlin은 언어 차원에서 getter, setter 문법을 지원합니다.

그래서 getter, setter, 필드를 하나의 요소로 묶어서 Property라고 명명한 듯 싶습니다.

