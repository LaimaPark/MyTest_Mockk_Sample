# Mockk 공부해보기

## 알아야 할 개념들

참고자료
- https://blog.kotlin-academy.com/mocking-is-not-rocket-science-basics-ae55d0aadf2b

### System Under Test (SUT)
- 테스트 중인 시스템 그 자체를 의미한다.
- It refers to a **system that is being tested** for correct operation.

### Depended On Component (DOC)
- SUT 를 테스트 하기 위한 구성요소이다. (내부 함수 같은 것들)
- 테스트할 때, DOC들을  mock 으로 대체해서 SUT에 연결한다.


### Example

- 테스트할 대상

```kotlin
class Dependency1(val value1: Int)
class Dependency2(val value2: String)

class SystemUnderTest(
    val dependency1: Dependency1,
    val dependency2: Dependency2
) {
 
	fun calculate() = 
	    dependency1.value1 + dependency2.value2.toInt()

}

```

- 테스트 코드

```kotlin
@Test
fun calculateAddsValues() {
    val doc1 = mockk<Dependency1>()    // 첫번째 DOC 를 mock 으로 대체
    val doc2 = mockk<Dependency2>()    // 두번째 DOC 를 mock 으로 대체

    every { doc1.value1 } returns 5    // Expected behavior of value1
    every { doc2.value2 } returns "6"  // Expected behavior of value2

    val sut = SystemUnderTest(doc1, doc2) 

    assertEquals(11, sut.calculate())
}
```

- While executing `calculate` function, mocked components are called and expected behaviors are executed.
- After that, we verify that result of executing a function with dependencies is correct.

### **Argument Matching**

```kotlin
every { mock.call(more(5)) } returns 1
every { mock.call(or(less(5), eq(5))) } returns -1
```

- more, less, eq 같은 명령어들을 Argument matchers 이라고 한다.
- Argument matchers 는 every 나 verify 에서 사용이 허용되는 argument 들을 지정하는 것이다.

### Expected Answer

- returns 등을 통해 DOC 의 반환값을 정의한다.

```kotlin
// mock1.call 할때마다 1, 2, 3 순으로 return
every { mock1.call(5) } returnsMany listOf(1, 2, 3)
every { mock1.call(5) } returns 1 andThen 2 andThen 3

// Exception 
every { mock1.call(5) } throws RuntimeException("error happened")

// return UNIT
every { mock1.callReturningUnit(5) } just Runs

// mock1.call(5) == arg<Int>(0)
every { mock1.call(5) } answers { arg<Int>(0) + 5 }
```

### Behavior verifiaction

- Mock 이 다른 double 들과 다른 것은, State 가 아닌 Behavior verification 이다.
- Behavior verification 은 DOC 가 호출이 되었는가를 검증 하는것.
- 아래처럼 verify 로 Behavior verification 를 하면 그때서야 다른데서 쓰이는 mock 과 같아진다.

```kotlin
@Test
fun calculateAddsValues() {
  val doc1 = mockk<Dependency1>()
  val doc2 = mockk<Dependency2>()

  every { doc1.value1 } returns 5
  every { doc2.value2 } returns "6"

  val sut = SystemUnderTest(doc1, doc2)

  assertEquals(11, sut.calculate())  // SUT

	// Behavior verifiaction 는 SUT 밑에서 이뤄어져야 한다. 
  verify { 
    doc1.value1    // 첫번째 DOC(doc1.value1) 가 호출되었나 체크
    doc2.value2    // 첫번째 DOC(doc2.value2) 가 호출되었나 체크
  }
}
```

### Unordered verification

```kotlin
 // verify 블록 안에 있는 DOC 들이 적어도 한번씩은 호출 됐다.
verify { 
  mock1.call(5)
  mock1.call(6)
}

 // 기본값 : atLeast = 1, atMost = Int.MAX_VALUE
verify(atLeast = 5, atMost = 7) { 
  mock1.call(5)
}

verify(exactly = 7) { 
  mock1.call(5)
}

// 호출이 안일어났는지 체크하는 방법
verify(exactly = 0) { 
  mock1.call(5)
}

// 이렇게도 할 수 있음 (Mockito 에서의 verifyZeroInteractions)
verify { 
  mock1 wasNot Called 
}

// 딱 이 안에 있는 것들만 불렸는지 체크
verifyAll { 
  mock1.call(5)
}
```

### VerifyOrder & VerifySequence

```kotlin
// DOC 들이 정확히 순차적으로 수행됐는데 체크
verifySequence {
  mock1.call(1)
  mock1.call(2)
  mock1.call(3)
}

// 순서만 체크할 때는 ( -> allowing gaps in the sequence of calls)
verifyOrder {
  mock1.call(1)
  mock1.call(3)
}
```
