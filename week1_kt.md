# 개요

우아한테크코스 6기 프리코스 1주차 미션으로 '숫자 야구 게임'을 진행하며 여러 번의 리팩토링을 진행해보는 중인데, 그 과정을 복기할겸 기록해보려고 한다.

백엔드로 지원했지만 코틀린에 흥미가 있기도 하고 지원자 수가 거의 10배 차이가 나서 자바보다는 코틀린 자료가 훨씬 부족할 것이라 생각하여 글은 코틀린으로 작성해보기로 했다.

# 1. 설계

```md
## Domain

### BallNumbers

data: 입력한 3자리 수

behaviours: 다른 BallNumbers와 비교

### BaseballGame

data: 정답 3자리 수를 들고 있는 `BallNumbers`

behaviours: 입력한 숫자와 정답을 비교 후 결과를 담은 `BallResult` 반환

### BallResult

data: 볼, 스트라이크 갯수 behaviours: 3스트라이크인지 확인
```

기능 요구 사항을 매우 간단하게 설계했다.

처음부터 설계를 완벽하게 하려고 하면 시간 소모가 너무 크고,

미션을 진행하면서 코드를 짜면서 요구 사항에 대한 이해도가 늘어나는 경험을 많이 했기 때문에 간단하게 핵심만 설계한 뒤, 일단 만들고 후에 고도화 하는 방식으로 진행 해보겠다.

# 1. BallNumbers

숫자 야구는 자동으로 생성된 3자리 숫자를 정답으로 사용한다.

그리고 사용자가 추측한 3자리 숫자와 대조하여 모든 숫자의 자리와 값이 일치할 경우

게임이 클리어되는데, `BallNumbers`는 3자리 숫자를 담는 일급 컬렉션으로,

정답, 추측을 모두 표현할 수 있는 범용 객체로 사용할 것이다.

```kotlin
// domain/BallNumbers.kt
package baseball.domain  
  
class BallNumbers(private val numbers: List<Int>) {

    init {  
		// 객체 생성 시, 만들어둔 검증 로직 실행
        validateNumbersLength(numbers)  
        validateNumbers(numbers)  
    }  

	// 생성 전에 검증 로직 수행을 위한 private 정적 메소드 생성
    companion object {  

		// 생성 시 인자로 들어온 숫자 리스트 크기가 3인지 검증
        private fun validateNumbersLength(numbers: List<Int>) {  
            if (numbers.size != 3) {  
                throw IllegalArgumentException("BallNumbers는 3자리로 이루어진 숫자여야 합니다.")  
            }  
        }  
		
		// 생성 시 인자로 들어온 숫자 리스트 내의 요소가 유효한지 검증
        private fun validateNumbers(numbers: List<Int>) {  
            for (number in numbers) {  
                if (number < 1 || number > 9) {  
                    throw IllegalArgumentException("BallNumber는 1~9 사이의 3자리 수여야 합니다.")  
                }  
            }  
        }
        
    }  
  
}
```

객체 생성 및 검증 로직만 작성하였다.

메소드의 depth를 2 이상 넘지 말라는 우아한 테크코스의 미션 제한 사항을 이미 어겨 버렸지만,

나중에 고쳐 보도록 하고, 일단 계속 만들어보겠다.

```kotlin
// domain/BallNumbers.kt

// ...

// BallNumbers 2개를 비교.
// 정답을 answer, 사용자가 입력한 추측을 guess라고 생각.

// answer, guess에서 동일한 index에 위치한 요소를 비교하여
// 값이 일치하면 true, 아니면 false 반환
//
// 즉, index에 위치한 요소가 strike인지 구하는 로직
fun isStrikeAt(index: Int, other: BallNumbers) =
    numbers[index] == other.numbers[index]

// guess의 index에 위치한 값을
// answer가 포함하고 있는지,
// strike가 아닌지 구하는 로직
//
// 즉, index에 위치한 요소가 ball인지 구하는 로직
fun isBallAt(index: Int, other: BallNumbers) =  
        !isStrikeAt(index, other) &&  
                numbers.any { it == other.numbers[index] }
```

이제 외부에서 `BallNumbers` 인스턴스를 생성하면,

다른 `BallNumbers` 인스턴스와 비교하여

n번째 요소가 스트라이크인지 볼인지 여부를 구할 수 있게 되었다.

---

# 2. BallResult

앞서 만든 메소드로 두 개의 `BallNumbers`를 비교하여 볼과 스트라이크의 개수를 구한 뒤,

결과를 저장할 클래스다.

출력(응답)에 사용될 데이터를 담고 있기 때문에 DTO라고 봐도 무방하다.

실제 Spring에서 응답을 보낼 때도 DTO가 사용되는데,

그 때는 객체를 JSON으로 변환하는 작업이 필요하다.

그러려면 각 필드에 getter를 작성해야 필드에 저장된 값을 읽어서 직렬화 해줄 수 있다.

고로 DTO는 getter를 오픈해도 캡슐화가 깨질 걱정을 하지 않아도 되는 것이다.

데이터를 전송하기 위한 말단이기 때문에 이미 완성된 데이터일 것이고,

다른 객체가 의존하여 꼬일 일도 없을 것이며,

무엇보다도 getter가 없으면 데이터를 읽어서 JSON으로 변환한 뒤,

클라이언트로 전송할 수 없게 되어 존재 의의가 사라지므로.

```kotlin
// domain/BallResult.kt
package baseball.domain;  
  
data class BallResult(    
        val strikeCount: Int,  
        val ballCount: Int,
)
```

코드는 매우 간단하게 응답에 필요한 정보인 볼과 스트라이크의 개수만을 담은 데이터 클래스로 만들었다.

결과 객체는 `ball`, `strike`의 개수를 담은 불변 인스턴스로 생성될 것이며,

외부에서도 필드 조회는 가능하기 때문에 출력할 때 사용하면 된다.

# BaseballGame

게임을 진행하는 클래스라고 생각하면 된다.

더 정확히는 사용자의 추측값을 받아서 정답과 비교하여

그 결과를 `BallResult`에 담아 반환하는 기능을 구현했다.

```kotlin
// domain/BaseballGame.kt
package baseball.domain;  
  
class BaseballGame(private val answer: BallNumbers) {

	// 사용자가 입력한 추측 값을 넣으면,
	// 스트라이크와 볼 갯수를 비교 후 그 결과를 BallResult에 담아 반환한다.
    fun play(guess: BallNumbers) =  
        BallResult(
	        strikeCount(guess), 
	        ballCount(guess),
	    )

	// isStrikeAt 메소드로 answer, guess의 모든 요소를 비교하여
	// 총 strike 갯수를 구함
	private fun strikeCount(guess: BallNumbers): Int {  
	    var count = 0  
	  
	    for (i in 1..3) {  
	        val isStrike = answer.isStrikeAt(i, guess)  
	        if (isStrike) {  
	            count++  
	        }  
	    }  
  
	    return count  
	}
	
	// isBallAt 메소드로 answer, guess의 모든 요소를 비교하여
	// 총 ball 갯수를 구함
	private fun ballCount(guess: BallNumbers): Int {  
	    var count = 0  
  
	    for (i in 1..3) {  
	        val isBall = answer.isBallAt(i, guess)  
	        if (isBall) {  
	            count++  
	        }  
	    }  
  
		return count  
	}
}
```

다음 구문을 자세히 보자.

`val isBall = answer.isBallAt(i, guess)`

`if` 문의 조건 안에 바로 `answer.isBallAt(i, guess)`를 넣어줘도 무방하지만,

코드 컴플리트2에서 가독성을 위해 조건을 보다 명시인 변수에 담는 기법이 생각나서

해당 방식으로 작성해보았다.

책에서 봤던 내용들은 금방 휘발되기 때문에 이런 식으로 기회가 될 때마다 사용해서 체화하는 것이 장기 기억에 도움이 되는 듯해서다.

`isBall`이나 `answer.isBallAt(i, guess)`나 둘 다 가독성이 괜찮기 때문에,

위와 같이 변수로 조건을 빼는 것은 좀 더 여러 개의 논리 연산이 섞였을 때, 큰 가독성 향상이 일어난다.

위와 같은 상황에선 큰 이점은 없지만 간단하지만 괜찮은 기법을 기억하기 위해서라고 생각하자.


# 매직 넘버 제거

숫자 리터럴을 그대로 코드에 포함시키는 것을 매직 넘버라고 한다.

매직 넘버는 의미를 파악하기 어려우며,

특히 개수가 증가하며 여기저기 흩어지게 되면,

변경이 생길 때 추적이 불가능하여 실수할 가능성이 높아진다.

그래서 사용을 지양해야 한다.

숫자 야구 게임은 간단하기 때문에 큰 문제가 되진 않겠지만,

여러 사람이 협업하는 큰 코드 베이스라고 생각하고 매직 넘버를 제거해보겠다.

`BallNumbers`는 항상 3자리 숫자를 갖기 때문에,

생성 시에도 길이 체크를 3으로 해줬었다.

`BALL_COUNT`라는 전역 정적 상수로 만들어서 관리하도록 하겠다.

매직 넘버 3을 사용하는 모든 코드를 `BALL_COUNT`로 바꿔주면 된다.

`MIN_BALL_NUMBER`, `MAX_BALL_NUMBER`도 마찬가지다.

```kotlin
// domain/BallNumbers.kt
class BallNumbers {

	// ...

    companion object {  

		// 정적 상수 생성
		const val BALL_COUNT = 3  
		private const val MIN_BALL_NUMBER = 1  
		private const val MAX_BALL_NUMBER = 9

        private fun validateNumbersLength(numbers: List<Int>) {  
			// 매직 넘버 제거
            if (numbers.size != BALL_COUNT) {  
                throw IllegalArgumentException("BallNumbers는 3자리로 이루어진 숫자여야 합니다.")  
            }  
        }  
		
        private fun validateNumbers(numbers: List<Int>) {  
            for (number in numbers) {  
				// 매직 넘버 제거
                if (number < MIN_BALL_NUMBER || number > MAX_BALL_NUMBER) {  
                    throw IllegalArgumentException("BallNumber는 1~9 사이의 3자리 수여야 합니다.")  
                }  
            }  
        }
        
    }  
  
}
```

Kotlin의 정적 변수는 `companion object` 블록에 위치시키면 된다.

Java의 `static` 변수는 클래스 상단에 위치시키지만,

Kotlin에선 반대로 클래스 하단에 위치시킨다.

생성 이후로 절대 변경하지 않을 것이기 때문에 `const`를 붙여 상수로 선언했다.

`val`과 `const val`는 런타임, 컴파일 타임에 값이 결정된다는 차이가 있다.

그리고 매직 넘버를 모두 정적 상수로 대체했다.

이제 숫자 야구 게임이 3자리가 아닌 4자리, 5자리가 되더라도

모든 매직 넘버를 변경할 필요가 없고,

`BallNumbers`의 `BALL_COUNT`만 바꿔주면 된다.

변경 지점이 3 -> 1로 줄어 들었으며,

코드 베이스가 커져서 매직 넘버 사용 시 변경 지점이 N개가 된다 해도,

상수화 한 `BALL_COUNT`는 항상 변경 지점이 1임을 보장할 것이다.

매직 넘버를 상수화 하는 간단한 기법만으로 실수도 줄이고 가독성도 향상되며 유지 보수도 편해졌다.

어려운 일이 아니기 때문에 설령 간단한 프로그램을 작성하더라도 매직 넘버는 처음부터 배제하는 것이 좋을듯하다.

```kotlin
// domain/BaseballGame.kt

// ...

private fun strikeCount(guess: BallNumbers): Int {  
    var count = 0  

	// 매직 넘버 -> 상수
    for (i in 0..<BallNumbers.BALL_COUNT) {  
        val isStrike = answer.strikeAt(i, guess)  
        if (isStrike) {  
            count++  
        }  
    }  
  
    return count  
}  
  
private fun ballCount(guess: BallNumbers): Int {  
    var count = 0  

	// 매직 넘버 -> 상수
    for (i in 0..<BallNumbers.BALL_COUNT) {  
        val isBall = answer.isBallAt(i, guess)  
        if (isBall) {  
            count++  
        }  
    }  
  
    return count  
}

// domain/AnswerCreator.kt

// ...

fun create(): Answer {  
    val set = mutableSetOf<BallNumber>()  

	// 매직 넘버 제거
    while (set.size < BallNumbers.BALL_COUNT) {  
        set.add(BallNumber(pickRandomNumber()))  
    }  
  
    return Answer(set.toList())  
}
```

매직 넘버를 사용하던 부분을 모두 상수로 대체하였다.

# InputView

```kotlin
// view/InputView.kt
package baseball.view  
  
import camp.nextstep.edu.missionutils.Console  
  
class InputView {  
  
    fun inputBallNumbers(): List<Int> {  
        if (executionCount++ == 0) {  
            println("숫자 야구 게임을 시작합니다.")  
        }  
        print("숫자를 입력해주세요 : ")  
  
        return readLine()  
                .map { it.toString().toInt() }
    }  
  
    fun inputWillReplay(): Boolean {
	    executionCount = 0
	    
        print("\n3개의 숫자를 모두 맞히셨습니다! 게임 종료")  
        println("게임을 새로 시작하려면 1, 종료하려면 2를 입력하세요.")  
  
        return when (readLine()) {  
            "1" -> true  
            "2" -> false  
            else -> throw IllegalArgumentException("1, 2 중 하나를 입력해주세요.")  
        }  
    }  
  
    private fun readLine() =  
            Console.readLine().trim()  
  
    companion object {  
        private var executionCount = 0  
    }  
  
}
```

코드가 간단하기 때문에 설명할 것이 별로 없다.

`"숫자 야구 게임을 시작합니다"`라는 메세지는 처음 한 번만 출력되기 때문에,

`executionCount`을 통해 메세지 실행 횟수에 따른 출력 여부를 결정한다.

정답을 맞출 경우, 새 게임을 시작해야 하기 때문에 0으로 초기화된다.

`when` expression을 사용해서 게임 재시작/종료 여부를 `"1"` 또는 `"2"`로 입력 받고 있는데,

패턴이 둘 다 일치하지 않는 경우는 예외를 던지도록 처리했다.

# OutputView

```kotlin
// view/OutputView.kt
package baseball.view  
  
import baseball.domain.BallResult  
  
class OutputView {  
    fun printResult(result: BallResult) {  
        if (result.ballCount + result.strikeCount == 0) {  
            print("낫싱")  
        }  
		if (result.ballCount > 0) {  
		    print("${result.ballCount}볼 ")  
		}  
		if (result.strikeCount > 0) {  
		    print("${result.strikeCount}스트라이크")  
		} 
        println()  
    }  
}
```

# AnswerCreator

하나 빠뜨린게 있어서 추가하였다.

숫자 야구 게임을 시작하면 정답으로 사용될 3자리 숫자를 랜덤으로 생성하라는 요구 사항이 있기 때문에,

간단하게 `AnswerCreator`라는 이름으로 작성하였다.

중복 요소를 여러 개 넣어도 무시되는 Set에 랜덤으로 뽑은 숫자를 길이가 3이 될 때까지 넣어준 뒤,

정적 팩토리 메소드로 `BallNumbers`를 만들어 정답 3자리 숫자를 반환해준다. 

```kotlin
// domain/AnswerCreator.kt
package baseball.domain  
  
import camp.nextstep.edu.missionutils.Randoms  
  
class AnswerCreator {  
    fun create(): BallNumbers {  
        // 자동으로 중복 제거해주는 Set        
        val set = mutableSetOf<Int>()  
  
        // Set에 숫자 3개가 들어갈 때까지  
        while (set.size < 3) {  
            set.add(pickRandomNumber())  
        }  
  
        return BallNumbers(set.toList())  
    }  
  
    // 1~9 사이의 랜덤 생성 숫자 생성  
    private fun pickRandomNumber() = Randoms.pickNumberInRange(1, 9)  
  
}
```

# GameController

전체적인 흐름을 한 눈에 볼 수 있도록 주석으로 정리해두었다.

```kotlin
// controller/GameController.kt
import baseball.domain.BaseballGame  
import baseball.view.InputView  
import baseball.view.OutputView  
  
class GameController(  
        private val inputView: InputView,  
        private val outputView: OutputView,  
        private val answerCreator: AnswerCreator,  
) {  
    fun run() {  
        // 1. 게임 생성  
        val game = createGame()  
  
        // 2. 게임 실행(정답 맞출 때까지)  
        playRecursive(game)  
  
        // 3. 재실행 여부 입력  
        val willReplay = inputView.inputWillReplay()  
  
        // 4. "1"을 입력 받았다면 재실행, 아니면 그대로 종료  
        if (willReplay) {  
            run()  
        }  
    }  
  
    private fun playRecursive(game: BaseballGame) {  
        // 사용자가 추측한 숫자 입력  
        val ballNumbers = BallNumbers(inputView.inputBallNumbers())  
        // 정답과 비교 후 결과 반환  
        val result = game.play(ballNumbers)  
  
        // 결과 출력  
        outputView.printResult(result)  
  
        // 3스트라이크가 아닌 경우 재귀 호출  
        if (result.strikeCount != 3) {  
            playRecursive(game)  
        }  
    }  
  
    private fun createGame(): BaseballGame {  
        // 정답으로 사용할 3자리 랜덤 숫자 생성 및 반환  
        val answer = answerCreator.create()  
        return BaseballGame(answer)  
    }  
}
```

# Application

DI 및 `run` 메소드를 호출하여 프로그램을 실행한다.

자세한 설명은 생략한다.

```kotlin
// Application.kt
package baseball  
  
import baseball.controller.GameController  
import baseball.domain.AnswerCreator  
import baseball.view.InputView  
import baseball.view.OutputView  
  
fun main() {  
    val inputView = InputView()  
    val outputView = OutputView()  
    val answerCreator = AnswerCreator()  
    val gameController = GameController(inputView, outputView, answerCreator)  
  
    gameController.run()  
}
```

여기까지 프로토타입이 완성되었는데,

작은 프로젝트일수록 다양한 시도와
이제부터는 부족한 부분을 조금씩 개선해보도록 하겠다.

# Refactoring

```kotlin
// view/InputView.kt

// ...

fun inputWillReplay(): Boolean {  
    print("\n3개의 숫자를 모두 맞히셨습니다! 게임 종료")  
    println("게임을 새로 시작하려면 1, 종료하려면 2를 입력하세요.")  
  
    val input = readLine()  
  
    return when (input) {  
        "1" -> true  
        "2" -> false  
        else -> throw IllegalArgumentException("1, 2 중 하나를 입력해주세요.")  
    }  
}
```

`InputView`에서 게임을 클리어 했을 때 재실행 여부를 입력 받는 로직을 보면

반환 타입이 `Boolean`으로 되어있다.

이것도 역시 코드 컴플리트에서 본 내용인데,

2가지 상태를 담는 `flag` 변수를 `Boolean`으로 나타낼 수도 있지만,

`Boolean`이라는 타입만으로는 `true`, `false`가 각각 어떤 정확히 상태를 나타내는지 파악하기 어려울 수 있기 때문에,

`enum`으로 만들어 주는 것이 가독성 향상에 도움이 된다.

게다가 이후에 상태의 개수가 늘어날 가능성이 있을 때 손쉽게 확장이 가능한데,

`Boolean`으로는 2가지 상태밖에 나타낼 수 없기 때지만

`enum`은 기본적으로 DB의 serial처럼 요소가 늘어날수록 1씩 증가하는 정수값이기 때문에, 원하는 만큼 상태를 표현할 수 있다.

그러니까 현재 상태가 2개더라도 늘어날 가능성이 높다고 판단되면 `enum`으로 만드는 것이 추후 유지 보수 관점에서 이점을 가질 것이다.

```kotlin
// domain/GameStatus.kt
package baseball.domain  
  
enum class GameStatus(private val input: String) {  
    REPLAY("1"),  
    EXIT("2");  
  
    companion object {  
		fun of(input: String) =  
		        when (input) {  
		            "1" -> REPLAY  
		            "2" -> EXIT  
		            else -> throw IllegalArgumentException("GameStatus는 반드시 1, 2 중 하나를 입력해서 만들 수 있습니다.")  
		        }
    }  
}
```

게임을 클리어 한 뒤 사용자의 입력 `"1"`, `"2"` 중 하나를 받은 다음

재시작, 종료 중에 하나의 상태를 `GameStatus`으로 표현하였다.

이제 새로운 '일시정지' 등의 게임 상태가 얼마든지 추가되어도 상태를 늘려서 대응 가능하고,

가독성도 좀 더 좋아졌다.

거기다 검증도 `GameStatus.of` 에서 담당하기 때문에 여러 군데에서 사용하더라도

검증 포인트는 단 한 곳만 존재하게 되었다.

```kotlin
// view/InputView.kt

// ...

fun inputNextGameStatus(): GameStatus {  
    println("\n3개의 숫자를 모두 맞히셨습니다! 게임 종료")  
    println("게임을 새로 시작하려면 1, 종료하려면 2를 입력하세요.")  

	// ✅
    return GameStatus.of(readLine())
}
```

`inputWillReplay`의 메소드명과 로직을 수정하였다.

변경 이전의 코드와 비교해보면 한층 간결해진 것을 볼 수 있다.

여기선 한 번밖에 사용되지 않았지만, 여러 번 재사용 될 수록 위력이 더욱 크게 체감될 것이다.

```kotlin
// controller/GameController.kt

	// ...
	
	fun run() {  
	    // 1. 게임 생성  
	    val game = createGame()  
	  
	    // 2. 게임 실행(정답 맞출 때까지)  
	    playRecursive(game)  
	  
	    // 3. 재실행 여부 입력  
	    val status = inputView.inputNextGameStatus()  
	  
	    // 4. "1"을 입력 받았다면 재실행, 아니면 그대로 종료  
	    if (GameStatus.REPLAY == status) {  
	        run()  
	    } 
	}
```

`GameController`도 이에 맞춰서 변경하였다.

여긴 오히려 코드량은 늘어났지만, IDE를 사용한다면 `GameStatus`에 들어가서 어떤 상태가 존재하는지 바로 확인할 수 있다는 장점도 취할 수 있다.

```kotlin
// controller/GameController.kt

	// ...
	
	fun run() {  
	    // 1. 게임 생성  
	    val game = createGame()  
	  
	    // 2. 게임 실행(정답 맞출 때까지)  
	    playRecursive(game)
	    
		// 3. 재실행 여부 입력 및 재실행 or 종료 결정
        if (GameStatus.REPLAY == inputView.inputNextGameStatus()) {  
            run()
        }
    } 
```

굳이 한 줄을 줄인다면 이렇게 수정하면 되겠다.

이러고 넘어갈 수도 있겠지만, 한 가지 더 괜찮은 아이디어가 떠올라서 리팩토링에 반영해보겠다.

# 예외 처리

`GameStatus`는 반드시 `"1"`, `"2"` 중 하나의 값을 입력 받아야 하며,

그 외의 값이 입력된 경우는 예외를 발생시킨다.

```kotlin
// view/InputView.kt

	// ...
	
	companion object {  
	    fun of(input: String) =  
	            when (input) {  
	                "1" -> REPLAY  
	                "2" -> EXIT
		            // 💡잘못된 값 입력 시 예외 발생 
	                else -> throw IllegalArgumentException("GameStatus는 반드시 1, 2 중 하나를 입력해서 만들 수 있습니다.")  
	            }  
	}

```

그런데 예외라는 것 자체가 비용이 상당히 크다.

내부적으로 처리되는 스택을 되감는 등의 로직이 추가적으로 실행되기 때문이다.

그래서 속도가 중요시되는 C, C++ 진영 등에서는 예외를 거의 사용하지 않는다.(C는 예외라는 개념이 존재하지도 않는다.)

또한 Go와 Rust 같은 최신 언어도 예외가 아닌 다중 반환, 박싱 타입 등을 통해서 이를 처리하는 방식으로 예외 상황들을 다루는데, 아마 이런 성능 상의 문제도 한 몫 했을 것이다.

Java와 Kotlin은 예외를 가장 적극적으로 사용하는 언어인 것 같긴 하지만,

모든 것을 반드시 예외로 처리할 필요는 없을 것 같고, 가독성을 크게 희생하지 않는 선에서 성능 최적화가 가능하다면 시도해보는 것도 괜찮아 보인다.

아이디어는 간단하다.

```kotlin
// domain/GameStatus.kt
enum class GameStatus(private val input: String) {  
    REPLAY("1"),  
    EXIT("2"),  
  
    // 그 외 문자가 입력된 경우 UNKNOWN을 반환하여 알려줌  
    UNKNOWN("_");  
  
    companion object {  
        fun of(input: String) =  
                when (input) {  
                    "1" -> REPLAY  
                    "2" -> EXIT  
                    // "1", "2"가 아니면 UNKNOWN 반환  
                    else -> UNKNOWN  
                }  
    }  
}
```

`UNKNOWN`이라는 enum constant를 하나 더 만들어서 `"1"`, `"2"` 외의 값이 입력된 경우에 반환해주면 된다.

이제 `GameStatus`를 사용하는 쪽에서 이걸 적절히 처리해주면 될 것이다.

사용자에게 잘못된 입력임을 알려주고, 재입력을 하도록 유도하는 방법이 괜찮아보인다.

```kotlin
// view/InputView.kt

	// ...

	fun inputNextGameStatus(): GameStatus {  
	    println("\n3개의 숫자를 모두 맞히셨습니다! 게임 종료")  
	    println("게임을 새로 시작하려면 1, 종료하려면 2를 입력하세요.")  
	  
	    val status = GameStatus.of(readLine())  
	  
	    // 사용자가 잘못된 입력을 한 경우,  
	    // 다시 입력 받도록 함  
	    if (GameStatus.UNKNOWN == status) {  
	        // 경고 메세지를 출력하여 사용자의 올바른 입력을 유도  
	        println("잘못된 입력입니다. 반드시 1, 2 중 하나를 입력해야 합니다.")  
	        // 올바르지 않은 입력인 경우 올바른 입력을 할 때까지 재귀 호출하여 올바른 입력 반환  
	        return inputNextGameStatus()  
	    }  
	    // 올바른 입력인 경우 그대로 반환  
	    return status  
	}
}
```

`inputNextGameStatus`를 호출하는 `GameController` 측에서는 코드를 전혀 변경할 필요 없다.

올바른 값을 입력할 때까지 계속 재입력하게 로직을 변경했을 뿐, 리턴 타입이나 메소드 시그니처가 변경된 것은 아니기 때문이다.

여기서 추가적으로 해볼만한 것은 몇 회 이상 입력을 실패하면 프로그램을 강제 종료한다거나,

최적화를 위해 재귀 호출을 반복문으로 치환 한다던가 하고 싶다면 직접 적용해보면 좋을 듯 하다.

이제 `BallNumbers`를 리팩토링할 것인데, Primitive Type을 박싱해 볼 것이기 때문에

이에 대해 간단히 예시를 들어 설명해보겠다.

# Primitive Type Boxing

클린 코드에서는 Primitive 타입을 박싱하라는 조언이 존재한다.

`Person`으로 예시를 들어보겠다.

```kotlin
class Person(  
        val name: String,  
        val height: Int,  
) {  
  
    init {  
        // 검증 로직 호출  
        validateHeight(height)  
    }  
  
    // height를 검증하는 로직  
    private fun validateHeight(height: Int) {  
        if (height < 0) {  
            throw IllegalArgumentException("키는 음수가 될 수 없습니다.")  
        }  
    }  
}  
  
fun main() {  
    val person = Person("a", 1)  
}
```

키를 표현한 `height` 속성에 음수가 들어가는 것은 비상식적이다.

그래서 객체 생성 시 유효한 값을 넣기 위해 검증 단계가 필요하다.

그런데 사람 뿐만 아니라, 동물도 `height`를 가질 수 있을 것이다.

```kotlin
class Animal(val height: Int) {

	init {
		validateHeight(height)
	}

	// height를 검증하는 로직이 중복됨
	private fun validateHeight(height: Int) {
		if (height < 0) {
			throw IllegalArgumentException("키는 음수가 될 수 없습니다.");
		}
	}
}
```

`Animal`의 `height`에서 검증 로직 중복이 발생한다.

만약 검증해야 할 필드가 한 개가 아니라면 어떨까.

```kotlin
class Person {

	private fun validateHeight(height: Int) {
	}
	private fun validateWeight(weight: Int) {
	}
	private fun validateBirthDateTime(birth: LocalDateTime) {
	}
	private fun validateAge(age: Int) {
	}
	// 이하 수 십 개의 검증 로직이 더 필요하다고 가정
	
	// ...
}
```

`Person`이라는 클래스에서 너무 많은 책임을 담당하게 된다.

`height`, `weight`, `birthDateTime`, `age` 등의 검증까지 모두 처리해야 하니 말이다.

여러 곳에서 사용할수록 중복 횟수도 증가할 것이며, 변경 지점의 개수도 이와 비례할 것이다.

```kotlin
class Height(val height: Int) {

	init {
		validateHeight(height)
	}
	private fun validateHeight(height: Int) {
		if (height < 0) {
			throw IllegalArgumentException("키는 음수가 될 수 없습니다.")
		}
	}
}

class Weight(val weight: Int) {

	init {
		validateWeight(weight)
	}

	private fun validateWeight(weight: Int) {
		if (weight < 0) {
			throw IllegalArgumentException("무게는 음수가 될 수 없습니다.")
		}
	}
}
```

Primitive Type이라도 이렇게 클래스 내에 한 번 박싱하면 각 필드의 검증 책임을 각 클래스에 위임할 수 있다.

```kotlin
class Person(
	val height: Height;
	val weight: Weight;
	val birth: Birth;
	val age: Age;
) {

	// ...

	// 필드에 대한 검증 로직을 따로 만들 필요 없음
}

class Animal(
	val height: Height;
	val weight: Weight;
) {
	// ...
}
```

이제 각 필드는 어디서 사용되던 검증 로직을 중복 호출할 필요도, 변경 지점이 늘어날 일도 없어졌다.

단, 여기서 박싱에 대한 비용이 발생한다는 것을 인지는 해야 할 것이다.

`height`의 경우 4byte `Int` 하나를 클래스에 담기 위해 인스턴스를 만들게 되는데,

이는 필드의 값을 읽기 위해 한 번의 역참조가 더 필요하다는 뜻이고, 미세하지만 시간이 더 들어간다.

거기다 공간적 비용도 더 들어갈 것이다.

성능이 중요한 부분이거나 객체가 너무 많이 생성되야 하는 경우에는 대안이 필요할 것이라고 보여진다.

# BallNumber

이름이 비슷하므로 주의하자.(단수, 복수형 차이)

이전에 만들었던 `BallNumbers`는 `List<Integer> numbers`를 멤버로 가졌으며,

3자리 수인지, 각 숫자가 1~9 범위인지에 대한 검증 로직도 들고 있었다.

```kotlin
// domain/BallNumbers.kt
class BallNumbers(private val numbers: List<Integer>) {  

	// 생성 전에 검증 로직 수행
	init {
        validateNumbersLength(numbers)
        validateNumbers(numbers)
    }  

	// ...
}
```

`numbers`를 `BallNumber`라는 타입으로 박싱하여

`List<Integer>`가 아닌 `List<BallNumber>`로 만든다면 각 숫자에 대한 검증 책임은 `BallNumber`에 위임할 수 있다.

성능을 생각하면 꼭 박싱할 필요는 없다고 생각할 사람도 많을 것 같은데,

대규모 서비스, 여러 곳에서 범용적으로 사용되는 클래스라고 가정하고,

이제 막 접해본 클린 코드에 입문한다는 마음가짐으로 역할과 책임을 최대한 분리하기 위해 `BallNumber`를 박싱하여 `data class`로 만들어 보겠다.

```kotlin
// domain/BallNumber.kt
package baseball.domain  
  
data class BallNumber(val ballNumber: Int) {  
  
    companion object {  
        private const val MIN_BALL_NUMBER = 1  
        private const val MAX_BALL_NUMBER = 9  
    }  
  
    init {  
        validateBallNumber(ballNumber)  
    }  
  
    private fun validateBallNumber(ballNumber: Int) {  
        if (ballNumber < MIN_BALL_NUMBER || ballNumber > MAX_BALL_NUMBER) {  
            throw IllegalArgumentException("BallNumber는 1~9 사이의 숫자만 가능합니다.")  
        }  
    }  
}
```

코드는 매우 간단하며, 3자리 중 숫자 1자리를 담당하는 `BallNumber`는 스스로 자신의 범위를 검증한다.

`BallNumbers`에 있던 `MIN_BALL_NUMBER`와 `MAX_BALL_NUMBER`도 옮기는 것이 좋아 보인다.

해당 상수를 사용하던 검증 로직도 이쪽으로 옮겨왔으며, 하나의 `BallNumber`에 대한 정보를 이쪽으로 몰아 넣는 것이 응집도를 높이기 때문이다.

```kotlin
// domain/BallNumbers.kt
class BallNumbers(private val numbers: List<BallNumber>) {
    
    private final List<BallNumber> numbers;  

	// 생성 전에 검증 로직 수행
	// 이제 BallNumber의 검증은 해당 데이터를 들고 있는 각 BallNumber에게 위임하고
	// List의 길이만 검증하면 된다.
	init {  
	    validateNumbersLength(numbers)  
	}

	companion object {  
	  
	    private const val BALL_COUNT = 3  
	  
		// 인자의 타입을 List<Int> -> List<BallNumber>로 맞춰준다.
	    private fun validateNumbersLength(numbers: List<BallNumber>) {  
	        if (numbers.size != BALL_COUNT) {  
	            throw IllegalArgumentException("BallNumbers는 3자리로 이루어진 숫자여야 합니다.")  
	        }  
	    }  
}
}
```

간단하게 primitive type 데이터를 갖는 `BallNumber` 박싱을 완료했다.

```kotlin
// domain/AnswerCreator.kt    
class AnswerCreator {  
    fun create(): BallNumbers {  
        val set = mutableSetOf<BallNumber>()  
  
        while (set.size < BALL_COUNT) { 
	        // BallNumber로 박싱
            set.add(BallNumber(pickRandomNumber()))
        }  
  
        return BallNumbers(set.toList())  
    }  

}
```

`AnswerCreator`에서 `BallNumbers`를 생성할 때도

`Int`가 아닌 `BallNumber`를 넣어주도록 변경해야 한다.

```kotlin
// view/InputView.kt

	// ...
  
    fun inputBallNumbers(): List<BallNumber> {  
        if (executionCount++ == 0) {  
            println("숫자 야구 게임을 시작합니다.")  
        }  
        print("숫자를 입력해주세요 : ")  
  
        return readLine()  
            .map { it.toString().toInt() } 
            .map { BallNumber(it) }  
    }
```

`InputView`의 `inputBallNumbers`도 마찬가지다.

# BallNumbers 역할에 따라 분리

현재 `BallNumbers`는 3자리 숫자로 이루어진 정답을 담거나 사용자가 입력한 추측을 담는 범용 객체로 사용되고 있다.

그런데 만약, 코드가 복잡해지고 협업하는 개발자가 늘어나서

사용자에게 입력 받은 추측을 담고 있는 객체를 정답으로 사용하거나,

반대로 정답으로 사용하기 위한 객체를 추측으로 입력하는 등의 실수가 생길 수도 있을 것이다.

동일한 `BallNumbers` 객체라 타입만으로 구분할 수 없기 때문이다.

그래서 `BallNumbers`를 용도에 맞게 `Answer`, `Guess`라는 별개의 클래스로 분리하려고 한다.

정답이던 추측이던 3개의 숫자로 이루어졌기 때문에, 중복이 생길 것이다.

그래서 `BallNumbers`는 지우지 않고 추상 클래스로 만든 뒤,

`Answer`, `Guess`에서 상속 받도록 할 것이다.

```kotlin
// domain/BallNumbers.kt
// BallNumbers를 상속 받을 Guess, Answer에서 모두 필요  
// BallNumbers의 자식에서만 호출할 것이기 때문에 protected
abstract class BallNumbers(
	protected val numbers: List<BallNumber>,
) {  
  
    init {  
        validateNumbersLength(numbers)  
    }  
  
    companion object {  
  
        private const val BALL_COUNT = 3  
  
        private fun validateNumbersLength(numbers: List<BallNumber>) {  
            if (numbers.size != BALL_COUNT) {  
                throw IllegalArgumentException("BallNumbers는 3자리로 이루어진 숫자여야 합니다.")  
            }  
        }  
    }  
  
}
```

```kotlin
// domain/Guess.kt
class Guess(numbers: List<BallNumber>) : BallNumbers(numbers)
```

```kotlin
// domain/Answer.kt
class Answer(numbers: List<BallNumber>) : BallNumbers(numbers) {  

	// 정답 숫자인 경우에만 Guess를 인자로 받아서 strike인지 ball인지 확인 가능
    fun isStrikeAt(index: Int, other: Guess) =  
		    // Cannot access 'numbers': it is protected in 'Guess'
            numbers[index] == other.numbers[index]  
  
    fun isBallAt(index: Int, other: Guess) =  
            !isStrikeAt(index, other) &&  
		            // Cannot access 'numbers': it is protected in 'Guess'
                    numbers.any { it == other.numbers[index] }  
}
```

그런데 오류가 발생한다.

`numbers`는 `BallNumbers` 에 선언된 `protected` 멤버다.

`Answer`, `Guess`가 모두 `BallNumbers`를 상속 받고 있기 때문에 둘 다 자식 클래스다.

그래서 Java에서는 이런 경우, `Answer`에서 `Guess`의 `numbers`에 접근할 수 있는데,

Kotlin에서는 이것이 불가하다고 나온다.

Java 코드로 디컴파일해도 분명 `protected getNumbers`가 존재하는데, 정확한 문제를 모르겠다.

```kotlin
class Guess(numbers: List<BallNumber>) : BallNumbers(numbers) {  
  
    fun getAt(index: Int) =  
            numbers[index]  
}
```

`Guess`에 `getAt`을 만들면 간단히 해결되는 문제긴 하다.

`BallNumbers`를 사용하는 클래스인 `BaseballGame`도 변경하도록 하자.

```kotlin
// domain/BaseballGame.kt
class BaseballGame(private val answer: BallNumbers) {  
  
    fun play(guess: Guess) =  
            BallResult(
	            strikeCount(guess),
	            ballCount(guess),
	        )  
  
    private fun strikeCount(guess: Guess): Int {  
        var count = 0  
  
        for (i in 0..<BallNumbers.BALL_COUNT) {  
            val isStrike = answer.isStrikeAt(i, guess)  
            if (isStrike) {  
                count++  
            }  
        }  
  
        return count  
    }  
  
    private fun ballCount(guess: Guess): Int {  
        var count = 0  
  
        for (i in 0..<BallNumbers.BALL_COUNT) {  
            val isBall = answer.isBallAt(i, guess)  
            if (isBall) {  
                count++  
            }  
        }  
  
        return count  
    }  
}
```

그런데 다시 생각해보면 `BaseballGame`이 정말 필요한가 하는 의문이 든다.

결국 `BaseballGame`이 하는 일은 `Answer`가 가지고 있는 데이터와 `Guess`가 가지고 있는 데이터를 비교하여 `BallResult`를 도출하는 것이 전부인데,

이 일은 `Answer`가 가진 데이터만으로도 가능하다.

즉, `Answer` 내부에서 독자적으로 처리할 수 있는 작업이며,

`Answer` 내부에서 처리할 경우, `isBallAt`, `isStrikeAt` 등의 메소드를 외부에 노출할 필요조차 없어진다.

게다가 자기 자신이 소유한 데이터를 스스로 처리하니 응집도도 올라간다고 생각된다.

그대로 `BaseballGame`을 제거하고 정답과 추측을 비교하여 결과를 도출하는 기능을 `Answer`로 옮겨보자.

# Answer로 BaseballGame 기능 이전

```kotlin
// domain/Answer.kt
class Answer(numbers: List<BallNumber>) : BallNumbers(numbers) {  
  
    // BaseballGame으로부터 가져온 play 메소드. 이름만 compareTo로 변경  
    fun compareTo(guess: Guess) =  
        BallResult(  
            strikeCount(guess),  
            ballCount(guess),  
        )
  
    // BaseballGame으로부터 가져온 메소드  
    private fun strikeCount(guess: Guess): Int {  
        var strikeCount = 0
        // BallNumbers를 상속 받기 때문에 BallNumbers.BALL_COUNT가 아닌
        // BALL_COUNT로 가져올 수 있음
        for (i in 0..<BALL_COUNT) {  
            if (isStrikeAt(i, guess)) {  
                strikeCount++  
            }  
        }  
        return strikeCount  
    }  
  
    // BaseballGame으로부터 가져온 메소드  
    private fun ballCount(guess: Guess): Int {  
        var ballCount = 0  
        for (i in 0..<BALL_COUNT) {  
            if (isBallAt(i, guess)) {  
                ballCount++  
            }  
        }  
        return ballCount  
    }  
  
    // 내부에서 스스로 데이터를 처리하므로 private로 변경  
    private fun isStrikeAt(index: Int, other: Guess) =  
        numbers[index] == other.getAt(index)  
  
    // 내부에서 스스로 데이터를 처리하므로 private로 변경  
    private fun isBallAt(index: Int, other: Guess) =  
        !isStrikeAt(index, other) &&  
                numbers.any { it == other.getAt(index) }  
  
}
```

`BaseballGame`의 로직을 그대로 `Answer`로 옮겼다.

이제 `Answer` 내의 `isBallAt`, `isStrikeAt` 메소드를 외부로 노출시킬 필요가 없어졌으므로

`private`으로 변경한다.

`GameController`도 변경 사항에 맞춰서 수정해야 할 것이다.

```java
// controller/GameController.java
class GameController(  
    private val inputView: InputView,  
    private val outputView: OutputView,  
    private val answerCreator: AnswerCreator,  
) {  
    fun run() {  
        // 1. 게임 생성  
        val answer = answerCreator.create()  
  
        // 2. 게임 실행(정답 맞출 때까지)  
        guessRecursive(answer)  
  
        // 3.재실행 여부 입력 및 재실행 or 종료 결정  
        if (GameStatus.REPLAY == inputView.inputNextGameStatus()) {  
            run()  
        }  
    }  

	// playRecursive -> guessRecursive로 이름 변경
	// 인자 타입도 BallNumbers -> Answer로 변경
    private fun guessRecursive(answer: Answer) {  
        // 사용자가 추측한 숫자 입력  
        val ballNumbers = Guess(inputView.inputBallNumbers())  
        // 정답과 비교 후 결과 반환  
        val result = answer.compareTo(ballNumbers)  
  
        // 결과 출력  
        outputView.printResult(result)  
  
        // 3스트라이크가 아닌 경우 재귀 호출  
        if (result.strikeCount != BallNumbers.BALL_COUNT) {  
            guessRecursive(answer)  
        }  
    }  
  
}
```

맥락에 맞게 메소드 명을 변경하고, 인자도 맞춰주었다.

마지막으로 입력을 받을 때, 반드시 숫자로만 이루어진 문자열이 들어오는지 검증해주는 로직을 추가했다.

```kotlin
// view/InputValidator.kt
package baseball.view  
  
class InputValidator {  
    fun validateNumbersInput(input: String?) {  
        if (input == null || input.matches(Regex("^\\d+$"))) {  
            throw IllegalArgumentException("입력은 반드시 숫자로만 이루어져야 합니다.")  
        }  
    }  
}

// view/InputView.kt
fun inputBallNumbers(): List<BallNumber> {  
    if (executionCount++ == 0) {  
        println("숫자 야구 게임을 시작합니다.")  
    }  
    print("숫자를 입력해주세요 : ")  
  
    val input = readLine()  
    // 입력 받은 값이 숫자로 이루어진 문자열인지 검증  
    inputValidator.validateNumbersInput(input)  
  
    return input  
        .map { it.toString().toInt() }
        .map { BallNumber(it) }  
}

// Application.kt
fun main() {  
    val inputValidator = InputValidator()  
    val inputView = InputView(inputValidator)  
    val outputView = OutputView()  
    val answerCreator = AnswerCreator()  
    val gameController = GameController(inputView, outputView, answerCreator)  
  
    gameController.run()  
}
```

여기까지 모든 기능 구현 및 리팩토링이 완료되었다.

기능이 잘 동작하며, 모든 테스트를 통과하는 것을 확인했으나,

워낙 수정을 많이 했기 때문에 글에서 누락되었거나 잘못된 부분이 있을 수 있다.

그래도 지금까지 코드를 잘 따라왔다면 스스로 고칠 수 있을 것이다.

# 테스트

코드는 한번 작성하고 끝나는 것이 아니라, 계속해서 변화한다.

아무리 코드를 잘 짜더라도 기능을 추가하거나 변경하는 순간 이전에 작동하던 코드들이 돌아가지 않을 수 있다.

이전에 제대로 작동했던 기능이 변경에 의해 문제가 발생하는 경우를 '회귀 버그'라고 한다.

예를 들어 개발자가 A를 변경했는데, B에 문제가 생길 수 있다.

그럼 B에 문제가 생겼다는 사실을 즉각 인지해야 하는데, 

개발자가 직접 바꾼 부분은 A이기 때문에 B는 전혀 신경 쓰지 않아서 문제가 생긴 것을 모르고 넘어간다면 잘 돌아가던 프로그램이 제대로 동작하지 않게 될 것이다.

이 때, A, B에 대한 테스트 코드를 작성해두고 A를 변경한 뒤 테스트를 실행 해보면 문제가 있다는 것을 즉시 파악하고 고칠 수 있다.

테스트를 통해 코드를 수정할 때마다 회귀 버그 발생 여부를 코드 한 줄 보지 않고 확인할 수 있는 것이다.

테스트를 돌리면 자동으로 확인해주기 때문이다.

테스트를 통해 버그를 사전에 예방할 수 있고, 테스트를 안전 장치라고 생각한다면 자신감 있는 리팩토링이 가능해져서 유지 보수에 큰 도움이 되고,
테스트 코드 자체가 '문서'로서 기능하기도 하는데, 이를테면 어플리케이션의 각 기능들이 어떤 입력을 넣으면 어떤 결과를 도출하는지를 테스트 코드 자체만 보고도 쉽게 파악할 수 있다.

이제 테스트의 장점을 대략적으로 파악했으니, 테스트 코드를 작성해보자.

# BallNumberTest

매우 간단한 테스트 코드부터 작성해보겠다.

인텔리제이를 사용중이라면, `BallNumber` 클래스에 가서 `Cmd + Shift + T` 단축키를 눌러서 `BallNumber`의 테스트 파일을 만들 수 있다.

파일명은 자동으로 `BallNumberTest`가 되며, 파일 경로는 `src/main`이 아닌 `src/test` 하위에 위치하게 된다.
```kotlin
// BallNumberTest.kt
class BallNumberTest {  
  
    @Test  
    fun createBallNumberTest() {  
        // given: 어떤 값이 주어지고(1이 주어지고)
        val input = 1
  
        // when: 어떤 행위를 한다면(1로 BallNumber를 생성한다면)
        val ballNumber = BallNumber(1)  
  
        // then: 이런 결과가 도출된다(생성된 BallNumber의 ballNumber 속성에 1이 담겨있다.)
        assertThat(ballNumber.ballNumber).isEqualTo(1)  
    }  
}
```

BDD (Behavior-Driven Development),

행위 주도 개발의 약자로,

`given`, `when`, `then`이라는 일관된 시나리오대로 테스트 코드를 작성하여

가독성도 향상되고 이해하기 용이하게 만드는 테스트 방법을 뜻한다.

given:  ~라는 값이 주어지고

when: 그 값으로 ~기능을 실행했을 때,

then: ~라는 결과가 도출된다.

위와 같은 양식으로 테스트를 작성하면 된다.

`BallNumber`는 1~9 범위의 숫자만 들어갈 수 있으며,

그 외의 숫자, 0이나 10 등이 들어가면 예외가 발생한다.

방금 작성한 건 성공 케이스에 대한 테스트였기 때문에 무사히 통과할 것이다.

여기까진 좋다.

지금까지 우리가 작성한 테스트는 고작 1개이기 때문에 코드를 읽어보면 금방 어떤 테스트인지 파악할 수 있다.

그런데 만약 하나의 파일에 테스트가 수 십개 존재한다면, 바로 파악하는 것은 불가능 할 것이다.

테스트 코드는 '문서' 역할도 한다고 했었는데,

문서 역할에 충실하게 만들기 위해서 `@DisplayName`이라는 어노테이션을 사용하여

해당 테스트에 대한 설명을 좀 더 자세하게 묘사하여 한 눈에 어떤 테스트인지 인지할 수 있게 만드는 것이 좋다.

```kotlin
// BallNumberTest.kt
class BallNumberTest {  

	// DisplayName 어노테이션을 통해 테스트에 대한 자세한 설명 가능
    @DisplayName("1~9 범위의 수로 BallNumber를 생성할 수 있다")  
    @Test  
    fun createBallNumberTest() {  
        // given  
        val input = 1  
  
        // when  
        val ballNumber = BallNumber(1)  
  
        // then  
        assertThat(ballNumber.ballNumber).isEqualTo(1)  
    }  

	// 30개의 테스트 함수가 더 있다고 생각하자.
}
```

간단한 어노테이션 추가로 테스트 코드의 가독성을 올려서 문서로서의 기능을 강화했다.

이제 실패 케이스를 만들어보자.

```kotlin
// BallNumberTest.kt

// ...

@DisplayName("1~9 범위의 수가 아니라면 BallNumber를 생성 시 예외가 발생한다")  
@Test  
fun createBallNumberFailureTest() {  
    // given  
    val input1 = 0  
    val input2 = 10  
  
    // when  
    // then    
	// BallNumber를 만드는 순간 예외가 발생하기 때문에 when과 then이 동시에 발생한다.
    assertThatThrownBy { BallNumber(input1) }  
		// 발생한 예외 클래스
        .isInstanceOf(IllegalArgumentException::class.java)  
		// 발생한 예외 객체가 가진 메세지
        .hasMessage("BallNumber는 1~9 사이의 숫자만 가능합니다.")  
  
    assertThatThrownBy { BallNumber(input2) }  
        .isInstanceOf(IllegalArgumentException::class.java)  
        .hasMessage("BallNumber는 1~9 사이의 숫자만 가능합니다.")  
}
```

`BallNumber`가 가질 수 있는 숫자의 범위는 1~9다.

범위를 벗어나는 경계에 위치한 값은 0과 10인데,

범위 테스트를 할 때는 이런 경계값을 잘 확인해줘야 한다.

0과 10을 입력해서 `BallNumber`를 생성하면 예외가 발생하는데,

성공 케이스에서는 우선 `when`에서 `BallNumber`가 생성되고 난 뒤에

그 안에 들어있는 값을 `then`에서 확인하는 식으로 작성했지만,

실패 케이스에서 예외가 발생했기 때문에 `when`과 `then`을 분리할 수 없다.

애초에 객체 생성이 실패했기 때문에 객체가 만들어 질 수가 없기 때문이다.

그래서 `when`과 `then`을 묶어서 예외가 잘 발생했는지,

던져진 예외 타입과 메세지가 구현과 일치하는지 한 번에 확인하면 된다.

# GameStatusTest

`GameStatus`는 3가지 상태를 가진다.

`"1"`을 입력하면 `REPLAY`,

`"2"`을 입력하면 `EXIT`,

그 외 값이 입력되면 유효하지 않은 상태임을 나타내는 `UNKNOWN`이 반환된다.

원래 유효하지 않은 값이 입력되면 예외를 던졌었는데,

`UNKNOWN`으로 이를 대체했던 것을 기억할 것이다.

그래서 유효한 상태를 만드는 테스트,

유효하지 않은 상태가 만드는 테스트

각 2개를 작성했는데, 내용이 매우 간단하므로 설명을 코드로 대신한다.

```kotlin
// GameStatusTest.kt
  
class GameStatusTest {  
  
    @DisplayName("문자열 1, 2로 GameStatus를 생성할 수 있다")  
    @Test  
    fun createGameStatusTest() {  
        // given  
        val input1 = "1"  
        val input2 = "2"  
  
        // when  
        val status1 = GameStatus.of(input1)  
        val status2 = GameStatus.of(input2)  
  
        // then  
        assertThat(status1).isEqualTo(GameStatus.REPLAY)  
        assertThat(status2).isEqualTo(GameStatus.EXIT)  
    }  
  
    @DisplayName("문자열 1, 2외의 값으로 생성하면 UNKNOWN 상태가 된다")  
    @Test  
    fun createUnknownGameStatusTest() {  
        // given  
        val input1 = "X"  
        val input2 = "O"  
  
        // when  
        val status1 = GameStatus.of(input1)  
        val status2 = GameStatus.of(input2)  
  
        // then  
        assertThat(status1).isEqualTo(GameStatus.UNKNOWN)  
        assertThat(status2).isEqualTo(GameStatus.UNKNOWN)  
    }  
}
```

# BallNumbersTest

처음엔 사용자가 입력한 3자리 숫자와 3자리 정답 숫자를 `BallNumbers`라는 하나의 클래스로 퉁 쳤었는데,
이후 `BallNumbers`를 추상 클래스로 변환한 뒤, `Answer`와 `Guess`로 분리했었다.

두 클래스 모두 `BallNumbers`를 상속 받고 `BallNumbers`에 위치한 `protected` 멤버 변수를 공유하기 때문에 `Guess`, `Answer`를 생성할 때는 결국 `BallNumbers` 내에서 검증이 일어난다.

고로 `BallNumbers`에서 `Guess`와 `Answer` 생성 테스트를 모두 진행했다.

```kotlin
// BallNumbers.kt

class BallNumbersTest {  
    @DisplayName("서로 다른 3자리 BallNumber로 Guess, Answer를 만들 수 있다")  
    @Test  
    fun createGuessWithThreeBallNumbers() {  
        
        // given  
        val ballNumbers = listOf(  
            BallNumber(1),  
            BallNumber(2),  
            BallNumber(3),  
        )  
  
        // when  
        val guess = Guess(ballNumbers)  
        val answer = Answer(ballNumbers)  
  
        assertThat(ballNumbers).isNotSameAs(ballNumbers.toList())  
  
        // then  
        assertThat(guess)
		    // Guess의 프로퍼티인 numbers를 List로 추출
            .extracting("numbers")  
            .asList()             
            // numbers는 List<BallNumbers>이며,
            // 아래 요소를 모두 정확한 순서대로 포함해야 함
		    .containsExactly(  
		        BallNumber(1),  
		        BallNumber(2),  
		        BallNumber(3),  
		    )
  
        assertThat(answer)  
            .extracting("numbers")  
            .asList()  
            // BallNumber가 data class이기 때문에
            // ballNumbers.toList()는 deep copy 된 List
            // containsExactlyElementsOf는 현재 List와 인자로 주어진 List의
            // 각 요소의 동등성을 비교
            .containsExactlyElementsOf(ballNumbers.toList())  
    }

	@DisplayName("3자리가 아닌 BallNumber로 Guess, Answer를 생성하면 예외 발생한다")  
	@Test  
	fun throwsIfCreateGuessWithTwoBallNumbers() {  
	  
	    // given  
	    val ballNumbers = listOf(  
	        BallNumber(1),  
	        BallNumber(2),  
	    )  
	  
	    // when  
	    // then    assertThatThrownBy { Guess(ballNumbers) }  
	        .isInstanceOf(IllegalArgumentException::class.java)  
	        .hasMessage("BallNumbers는 3자리로 이루어진 숫자여야 합니다.")  
	  
	    assertThatThrownBy { Answer(ballNumbers) }  
	        .isInstanceOf(IllegalArgumentException::class.java)  
	        .hasMessage("BallNumbers는 3자리로 이루어진 숫자여야 합니다.")  
	  
	}

}
```

테스트를 하다 보니 검증하는 것을 빠뜨린 부분이 있었는데,

숫자 야구는 1, 1, 2와 같이 중복 숫자가 있으면 안 된다.

그런데 `Answer`나 `Guess`를 만들 때, 중복된 숫자가 들어가도 예외가 던져지지 않는다.

```kotlin
// BallNumbers.kt

abstract class BallNumbers(protected val numbers: List<BallNumber>) {  
  
    init {  
        validateNumbersLength(numbers)  
        validateDuplicateNumbers(numbers)  
    }
    
	private fun validateDuplicateNumbers(numbers: List<BallNumber>) {  
	    if (HashSet(numbers).size != BALL_COUNT) {  
	        throw IllegalArgumentException("BallNumbers에 중복 숫자가 존재하면 안 됩니다.")  
	    }  
	}
}
```

`BallNumbers`에 중복 숫자 검증 로직을 추가했다.

```kotlin
// BallNumbersTest.kt

// ...

@DisplayName("중복 숫자가 존재하는 3자리 BallNumber로 Guess, Answer를 생성하면 예외 발생한다")  
@Test  
fun throwsIfCreateGuessWithDuplicateBallNumbers() {  
  
    // given  
    val ballNumbers = listOf(  
        BallNumber(1),  
        BallNumber(1),  
        BallNumber(2),  
    )  
  
    // when  
    // then    
    assertThatThrownBy { Guess(ballNumbers) }  
        .isInstanceOf(IllegalArgumentException::class.java)  
        .hasMessage("BallNumbers에 중복 숫자가 존재하면 안 됩니다.")  
  
    assertThatThrownBy { Answer(ballNumbers) }  
        .isInstanceOf(IllegalArgumentException::class.java)  
        .hasMessage("BallNumbers에 중복 숫자가 존재하면 안 됩니다.")  
  
}
```

이제 중복 숫자가 있는 경우도 예외를 제대로 던지는 것을 테스트했다.

# Answer

`BallNumbers`에서 생성 로직은 성공, 실패 케이스를 모두 테스트 했기 때문에,

`Answer`에서는 나머지 부분만 테스트 하면 될 듯 싶다.

로직은 매우 간단하지만..

```kotlin
// AnswerTest

class AnswerTest {  
  
    @DisplayName("Answer와 일치하는 Guess를 비교하면 3스트라이크를 결과로 얻는다")  
    @Test  
    fun compareToGuessWithEqualNumbersTest() {  
        // given  
        val answerCreator = AnswerCreator()  
        val answer = answerCreator.create()  
        // ❌ 랜덤으로 생성된 answer에 무슨 숫자가 들어있는지 어떻게 알아낼 수 있을까
        val guess = ?
  
        // when  
        val (strikeCount, ballCount) = answer.compareTo(guess)  
  
        // then  
        assertThat(strikeCount).isEqualTo(3)  
        assertThat(ballCount).isEqualTo(0)  
    }  
}
```

`answerCreator.create()`로 생성된 `Answer`는 임의의 3자리 숫자를 갖는다.

매 번 생성할 때마다 완전히 달라지기 때문에 예측할 수 없는 존재다.

```kotlin
val ballNumbers = listOf(  
    BallNumber(1),  
    BallNumber(2),  
    BallNumber(3),  
)
val answer = Answer(ballNumbers)  
val guess = Guess(ballNumbers)
```

물론 위와 같이 `AnswerCreator`를 사용하지 않고, 고정된 `ballNumbers`를 사용하는 것도 방법이라면 방법이다.

그래도 학습을 위해, 매우 반복적으로 `AnswerCreator`를 사용해야 하는 케이스라고 가정하고

좀 더 나은 방법을 알아보겠다.

```kotlin
// NumberPicker.kt
interface NumberPicker {  
    fun pick(): Int
}

// RandomNumberPicker.kt
const val DEFAULT_START_INCLUSIVE = 1  
const val DEFAULT_END_INCLUSIVE = 9  
  
class RandomNumberPicker(  
    private val startInclusive: Int = DEFAULT_START_INCLUSIVE,  
    private val endInclusive: Int = DEFAULT_END_INCLUSIVE,  
) : NumberPicker {  
    override fun pick() = Randoms.pickNumberInRange(startInclusive, endInclusive)  
}
```

기본적으로 1~9 범위 내에서 랜덤 숫자를 뽑는 `pick`메소드를 가진 `RandomNumberPicker`을 만들었다.

원한다면 범위를 변경할 수도 있지만, 그대로 사용하겠다.

`NumberPicker` 인터페이스를 상속하기 때문에 `RandomNumberPicker`은 `NumberPicker`로 타입 변환 가능하다.

```kotlin
// AnswerCreator.kt
class AnswerCreator(private val numberPicker: NumberPicker) {  
    fun create(): Answer {  
        val set = mutableSetOf<BallNumber>() as LinkedHashSet  
  
        while (set.size < BallNumbers.BALL_COUNT) {  
	        // 이제 numberPicker로부터 랜덤 숫자를 뽑아준다
            set.add(BallNumber(numberPicker.pick()))  
        }  
        return Answer(set.toList())  
    }  
}
```

`NumberPicker`를 생성자로 주입 받아서 랜덤 숫자 뽑는 역할을 위임하였다.

```kotlin
// Application.kt
fun main() {  
    val inputValidator = InputValidator()  
    val inputView = InputView(inputValidator)  
    val outputView = OutputView()  
    // RandomNumberPicker 생성
    val randomNumberPicker = RandomNumberPicker()  
    // 이제 AnswerCreator 생성 시, NumberPicker 주입해야 함
    val answerCreator = AnswerCreator(randomNumberPicker)  
    val gameController = GameController(inputView, outputView, answerCreator)  
  
    gameController.run()  
}
```

`Application`에도 변경 사항을 반영하였다.

여기서부터가 중요한데,

테스트를 위한 `FixedNumberPicker`를 생성할 것이다.

이 클래스 역시 `NumberPicker`를 상속받기 때문에 `AnswerCreator`의 속성으로 `RandomNumberPicker`를 대체하여 주입 가능하다.

```kotlin
// FixedNumberPicker.kt
class FixedNumberPicker(vararg numbers: Int) : NumberPicker {  
    private val numbers: Queue<Int> = LinkedList(numbers.toList())  
  
    override fun pick(): Int {  
        return numbers.poll() ?: throw NoSuchElementException("더 이상 뽑을 숫자가 없습니다.")  
    }  
}
```

인스턴스 생성 시, `vararg`로 `Int` 여러 개를 생성자의 인자로 받을 수 있으며,

인자는 `Queue`에 저장된다.

그리고 `pick`을 호출할 때마다 하나씩 빼서 반환한다.

즉, 다음과 같이 사용할 수 있다.

```kotlin
val numberPicker = FixedNumberPicker(1, 2, 3)
numberPicker.pick() // 1
numberPicker.pick() // 2
numberPicker.pick() // 3
```

이렇게 `NumberPicker`라는 추상 타입(인터페이스)에 의존하게 되면,

실 사용시에는 랜덤 숫자를 뽑는 `RandomNumberPicker`를 사용하고,

테스트 시에는 우리가 입력한, 고정된 값을 하나씩 뽑는 `FixedNumberPicker`로 교체할 수 있기 때문에 아주 편하게  테스트를 수행할 수 있다.

```kotlin
// AnswerTest.kt
class AnswerTest {  
  
    @DisplayName("Answer와 일치하는 Guess를 비교하면 3스트라이크를 결과로 얻는다")  
    @Test  
    fun compareToGuessWithEqualNumbersTest() {  
        // given  
		// 테스트 시에는 FixedNumberPicker에 항상 고정된 값을 뽑도록 설정할 수 있다.
        val fixedNumberPicker = FixedNumberPicker(1, 2, 3)  
        val answerCreator = AnswerCreator(fixedNumberPicker)
        // 이제 answer는 내부적으로 FixedNumberPicker에 의해 값을 뽑는다.
        // 값을 고정했기 때문에 항상 1, 2, 3이 뽑힐 것이다.  
        val answer = answerCreator.create()  
  
        val guess = Guess(  
            listOf(  
                BallNumber(1),  
                BallNumber(2),  
                BallNumber(3),  
            ),
        )  
  
        // when  
        val (strikeCount, ballCount) = answer.compareTo(guess)  
  
        // then  
        assertThat(strikeCount).isEqualTo(3)  
        assertThat(ballCount).isEqualTo(0)  
    }  
}
```

모든 `index`와 값이 일치하므로 테스트 결과 3스트라이크가 나온다.

```kotlin
// AnswerTest.kt

// ...

@DisplayName("Answer에 존재하지만 index는 다른 3개의 BallNumber를 가진 Guess를 비교하면 3볼을 결과로 얻는다")  
@Test  
fun compareToGuessContainedAtDifferentIndicesNumbersTest() {  
    // given  
    val fixedNumberPicker = FixedNumberPicker(1, 2, 3)  
    val answerCreator = AnswerCreator(fixedNumberPicker)  
    val answer = answerCreator.create()  
  
    val guess = Guess(  
        listOf(  
            BallNumber(3),  
            BallNumber(1),  
            BallNumber(2),  
        ),
    )  
  
    // when  
    val (strikeCount, ballCount) = answer.compareTo(guess)  
  
    // then  
    assertThat(strikeCount).isEqualTo(0)  
    assertThat(ballCount).isEqualTo(3)  
}
```

이번에는 3볼을 반환하는 케이스에 대해 테스트를 만들었다.

```kotlin
// AnswerTest.kt

// ...

@DisplayName("Answer와 Guess 간에 일치하는 BallNumber가 없는 경우 ball, strike는 모두 0이 된다.")  
@Test  
fun nothingTest() {  
    // given  
    val fixedNumberPicker = FixedNumberPicker(1, 2, 3)  
    val answerCreator = AnswerCreator(fixedNumberPicker)  
    val answer = answerCreator.create()  
  
    val guess = Guess(  
        listOf(  
            BallNumber(4),  
            BallNumber(5),  
            BallNumber(6),  
        ),
    )  
  
    // when  
    val (strikeCount, ballCount) = answer.compareTo(guess)  
  
    // then  
    assertThat(strikeCount).isEqualTo(0)  
    assertThat(ballCount).isEqualTo(0)  
}
```

마지막으로 일치하는 `BallNumber`가 없는 경우를 테스트했다.
