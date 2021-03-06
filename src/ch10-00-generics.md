# 제네릭 타입, 트레잇, 라이프타임

모든 프로그래밍 언어는 중복되는 개념을 효율적으로 처리하기 위한 도구를 가지고 있습니다.
러스트에선 *제네릭(generic)* 이 그 역할을 맡습니다.
제네릭은 구체적(concrete)인 타입이나 다른 속성들을 추상화하고, 그에 대한 대역을 맡습니다.
우린 코드를 작성하고 컴파일할 때 제네릭들이 실제로 어떻게 완성되는지
알 필요 없이, 제네릭의 동작이나 다른 제네릭과의 관계를 표현할 수 있습니다.

함수가 어떤 값을 담을지 알 수 없는 매개변수를 전달받아서 동일한 코드를
다양한 구체적 값으로 실행하는 것처럼, 함수는 `i32`, `String` 같은
구체 타입 대신 제네릭 타입의 파라미터를 전달받을 수 있습니다.
사실 우린 이미 여러 제네릭을 사용해봤었습니다.
6장에서는 `Option<T>`, 8장에서는 `Vec<T>`와 `HashMap<K, V>`, 9장에서는 `Result<T, E>` 제네릭을
사용했죠. 이번 장에서는 제네릭을 사용해 우리만의 타입, 함수, 메소드를 정의하는 방법을 살펴보겠습니다.

우선, 중복되는 코드를 함수로 분리하는 방법을 살펴볼 겁니다.
그러고 나서 매개변수의 타입만 다른 두 함수가 생길 경우, 제네릭 함수를 사용해
코드 중복을 한 번 더 줄여보겠습니다. 또한, 제네릭 타입을 구조체 및 열거형 정의에
사용하는 방법도 살펴보겠습니다.

그리고, *트레잇(trait)* 을 이용해 동작을 제네릭한 방식으로 정의하는 법을 배워보겠습니다.
트레잇을 제네릭 타입과 함께 사용하면, 온갖 타입을 전부 허용하지 않고 특정 동작을 하는 타입만
허용할 수 있습니다.

마지막으로는 *라이프타임(lifetime)* 을 살펴보겠습니다. 라이프타임은 제네릭의 일종이며,
우리가 컴파일러에게 참조자들이 서로 어떤 관계에 있는지를 알려주는 데에 사용합니다.
우리가 수많은 상황에서 값을 borrow 하면서도 컴파일러가 참조자의 유효성을 검증할 수 있는 이유도
라이프타임이 있기 때문입니다.

## 함수로 분리하여 중복 없애기

제네릭 문법을 배우기 전에,
먼저 제네릭 타입을 이용하지 않고서 중복된 코드를 없애는 요령을 알아보겠습니다.
여러분이 함수로 분리할 중복 코드를 찾아내는 방법은,
제네릭을 이용할 수 있는 중복 코드를 찾아낼 때도 그대로 적용할 수 있습니다.
요령을 배우고 나면 제네릭 함수를 작성해보도록 하겠습니다!

Listing 10-1과 같이 리스트에서 가장 큰 숫자를 찾아내는
간단한 프로그램을 생각해보죠.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-01/src/main.rs:here}}
```

<span class="caption">Listing 10-1: 숫자 리스트에서
가장 큰 수를 찾는 코드</span>

이 코드는 `number_list` 변수에 정수 리스트를 저장하고,
`largest` 변수에 리스트의 첫 번째 아이템을 집어넣습니다.
그리고 리스트 내 모든 숫자들을 순회하는데,
만약 현재 값이 `largest` 에 저장된 값보다 크다면
`largest` 의 값을 현재 값으로 변경합니다.
현재 값이 여태까지 본 가장 큰 값보다 작다면 `largest`의 값은 바뀌지 않습니다.
리스트 내 모든 숫자를 돌아보고 나면 `largest` 는 가장 큰 값을 갖습니다
(이 경우에는 100이 됩니다).

만일 두 개의 숫자 리스트에서 각각 가장 큰 숫자를 찾고 싶다면,
Listing 10-2처럼 Listing 10-1의 코드를 복사해,
프로그램에 동일한 로직을 두 번 작성할 수도 있습니다.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-02/src/main.rs}}
```

<span class="caption">Listing 10-2: *두* 개의 숫자 리스트에서 가장 큰 숫자를
찾는 코드</span>

이 코드는 잘 동작하지만, 중복된 코드를 생성하는 일은 지루하고 오류가 발생할 가능성도 커집니다.
또한, 로직을 바꾸고 싶을 때 수정해야 할 부분이 여러 곳으로 늘어난다는 의미이기도 합니다.

이러한 중복을 제거하기 위해서, 정수 리스트를 매개변수로 전달받아
동작하는 함수를 정의하여 추상화해볼 수 있습니다.
이렇게 하면 코드가 더 명확해지고 목록에서 가장 큰 숫자를 찾는다는
개념을 추상적으로 표현할 수 있습니다.

Listing 10-3에서는 가장 큰 수를 찾는 코드를
`largest`라는 이름의 함수로 분리했습니다.
Listing 10-1 코드는 하나의 특정한 리스트에서만 가장 큰 숫자를 찾을 수 있는데 반해,
이 프로그램은 각각의 두 리스트에서 가장 큰 숫자를 찾을 수 있습니다.

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-03/src/main.rs:here}}
```

<span class="caption">Listing 10-3: 두 리스트에서 가장 큰 수를 찾는
추상화된 코드</span>

`largest` 함수는 `list` 매개변수를 갖는데,
이는 함수로 전달될 임의의 `i32` 값 슬라이스를 나타냅니다.
실제로 `largest` 함수가 호출될 때는 우리가 넘겨준 구체적인 값으로
실행됩니다.

Listing 10-2에서부터 Listing 10-3까지 우리가 거친 과정을 요약하면
다음과 같습니다:

1. 중복된 코드를 식별합니다.
2. 중복된 코드를 함수의 본문으로 분리하고, 함수의 시그니처 내에
   해당 코드의 입력값 및 반환 값을 명시합니다.
3. 중복됐었던 두 지점의 코드를 함수 호출로 변경합니다.

다음에는 제네릭으로 이 과정을 그대로 진행하여 새로운 방법으로
중복된 코드를 제거해보겠습니다.
함수 본문이 특정한 값 대신 추상화된 `list` 로 동작하는 것과 같이,
제네릭을 이용한 코드는 추상화된 타입으로 동작합니다.

만약 우리가 `i32` 슬라이스에서 최댓값을 찾는 함수와,
`char` 슬라이스에서 최댓값을 찾는 함수를 따로 가지고 있다면 어떨까요?
이런 중복은 어떻게 제거해야 할지 한번 알아봅시다!
