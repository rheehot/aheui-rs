아희 러스트 매크로
===========

아희 러스트 매크로는 프로페셔널한 Rust 코드베이스에 과학적이고 친숙한 한글 기반의 프로그래밍 언어인 아희로 작성된 코드를 손쉽게 삽입하고 연동할 수 있도록 도와드립니다.

## 사용방법

`Cargo.toml`에 `aheui-macro` 패키지를 의존성으로 등록합니다.

```toml
[dependencies]
aheui-macro={ git = "https://github.com/foriequal0/aheui-rs" }
```

이제 아희로 작성된 함수 상단에 `#[aheui]` 어트리뷰트를 붙이기만 하면 됩니다.

```rust
use aheui_macro::aheui;

#[aheui]
fn answer() -> i32 {
    밦밠따희
}

fn main() {
    println!("삶과 우주, 그리고 모든 것에 대한 궁극적인 질문의 답: {}", answer())
}
```

## 들여쓰기

아희는 들여쓰기에 민감한 언어이나, Rust는 그렇지 않습니다. IDE로 코드를 작성하거나,
코드 포매터를 사용하는 경우 이들 도구가 아희 코드의 들여쓰기를 변경해 코드를 망가뜨릴 수 있습니다.
아희 러스트 매크로는 이들 도구에 의해 쉽게 코드가 망가지지 않도록 들여쓰기를 인식합니다. 탭(`\t`)과 띄어쓰기(` `)에 무관하며
여러 줄에 공통으로 적용된 들여쓰기를 무시하고 코드를 인식합니다.

다음 두 코드는 동일하게 인식됩니다.

```rust
#[aheui]
fn answer() -> i32 {
  밦붌
  희떠
}
```

```rust
#[aheui]
fn answer() -> i32 {
밦붌
희떠
}
```

하지만 다음 코드는 위와는 다르게 인식됩니다.

```rust
#[aheui]
fn answer() -> i32 {
  밦붌
   희떠
}
```

이 코드는 다음과 같이 인식됩니다.

```aheui
밦붌
 희떠
```



## 다양한 옵션

### 입력

아희의 명세에서 `ㅇ`과 `ㅎ`을 받침으로 하는 `ㅂ`닿소리 명령은 표준입력으로부터 값을 받아 저장공간에 집어넣도록 되어있습니다.
아희 러스트 매크로에서는 Rust와의 유연한 상호연동을 위해 함수 인자로 넘겨주는 문자열로 표준입력을 대체할 수 있도록 지원합니다.

`input`이라는 이름의 매개변수가 있는 경우 `ㅂ`닿소리 명령은 표준입력 대신 해당 매개변수에서 값을 받습니다.
```rust
#[aheui]
fn codepoint(input: &str) -> i32 {
    밯희
}

assert_eq!(codepoint("뉮"), 45678);
```

`input=args(<매개변수 이름>)` 옵션을 `#[aheui]` 어트리뷰트에 명시하면 `ㅂ`닿소리 명령은 표준입력 대신 해당 매개변수에서 값을 받습니다.
```rust
#[aheui(input=args(x))]
fn codepoint_x(x: &str) -> i32 {
    밯희
}

// codepoint() 함수와 동일합니다.
assert_eq!(codepoint("뉮"), 45678);
```

`input=stdin` 옵션을 `#[aheui]` 어트리뷰트에 명시해 `ㅂ`닿소리 명령이 표준입력에서 값을 받게끔 강제합니다.
```rust
#[aheui(input=stdin)]
fn codepoint_stdin(input: &str) -> i32 {
    밯희
}

// 인자 "뉮"은 무시되고, codepoint 함수는 표준입력으로부터 값을 받아옵니다.
print!(codepoint("뉮"));
```

`input=stdin` 옵션은 `#[aheui]` 어트리뷰트를 `main` 함수 상단에 붙였을 경우에도 동일하게 적용됩니다.
```rust
#[aheui(input=stdin)]
fn main() {
    밯희
}
```
```bash
> echo "뉮" | cargo run --quiet
> echo $?
45678
```

`input=cli` 옵션을 `main` 함수 상단에 붙은 `#[aheui]` 어트리뷰트에 명시하면 `ㅂ`닿소리 명령은 표준입력 대신 명령행 실행 인자 중 첫번째 인자로부터 값을 받습니다.

 ```rust
#[aheui(input=cli)]
fn main() {
    밯희
}
 ```
```bash
> cargo run --quiet -- "뉮"
> echo $?
45678
```

`input=auto` 옵션을 `main` 함수 상단에 붙은 `#[aheui]` 어트리뷰트에 명시하면 실행 인자가 주어진 경우 `input=cli`처럼,
실행 인자가 주어지지 않은 경우 `input=stdin`처럼 동작합니다.
`main` 함수 상단에 붙은 `#[aheui]` 어트리뷰트의 경우, `input` 옵션의 기본값은 `auto` 이므로 `input=auto`는 생략할 수 있습니다.
```rust
#[aheui]
fn main() {
    밯희
}
```
```bash
> cargo run --quiet -- "뉮"
> echo $?
45678
> echo "뉮" | cargo run --quiet
> echo $?
45678
```

### 출력

아희의 명세에서 `ㅇ`과 `ㅎ`을 받침으로 하는 ㅁ닿소리 명령은 저장공간에서 값을 뽑아 표준출력에 값을 출력하게 되어있습니다.
또한 아희 코드가 `ㅎ`닿소리 명령을 만났을 때 스택의 값을 뽑아 운영체제에 종료코드로 돌려주고, 뽑을 값이 없는 경우 0을 돌려주게 되어있습니다.
아희 러스트 매크로에서는 Rust와의 유연한 상호연동을 위해 종료코드와 표준출력으로 값을 출력하는 대신 반환값으로 돌려줄 수 있도록 지원합니다.

앞서 본 예시와 같이, 반환 타입을 `(u|i)(size|8|16|32|64|128)` 정수타입으로 지정하면, 종료코드를 반환값으로 돌려줍니다.
이 때 `ㅁ`닿소리 명령에 의한 출력은 표준출력으로 출력됩니다.
```rust
#[aheui]
fn codepoint(input: &str) -> i32 {
    밯희
}

assert_eq!(codepoint("뉮"), 45678);
```

`ㅁ`닿소리 명령이 내보낸 값을 표준출력으로 출력하지 않고 함수의 호출자에게 돌려주고싶다면 반환 타입을 `String` 타입으로 지정합니다.
```rust
#[aheui]
fn character(input: &str) -> String {
    방맣희
}

assert_eq!(character("45678"), "뉮".to_string());
```

정수타입과, `String` 타입을 튜플로 묶어 반환 타입으로 지정하면 두가지 출력을 모두 얻을 수 있습니다. 순서는 무관합니다.
```rust
#[aheui]
fn answer_character(input: &str) -> (i32, String) {
    밦밠따방맣희
}

assert_eq!(answer_character("45678"), (42, "뉮".to_string()));
```

아무 반환타입이 없으면 `ㅁ`닿소리 명령이 내보낸 값은 표준출력으로 출력되고, `ㅎ`닿소리 명령에 의한 값은 버려집니다.
```rust
#[aheui]
fn character(input: &str) {
    방맣희
}

fn main() {
    character("45678")
}
```

```bash
> cargo run --quiet
뉮
```

`main` 함수는 현재 아무 반환타입도 지정할 수 없습니다. Rust 에서 `main` 함수는 반환 타입으로 `i32`나 `String`혹은 튜플을 지정할 수 없고,
아희 러스트 매크로는 위에 언급한 반환타입 이외에 타입을 지원하지 않기 때문입니다.
그러나 이 경우 `ㅎ`닿소리 명령에 의한 값은 버려지지 않고 OS에 종료코드로 제공되고, `ㅁ`닿소리에 의한 값 역시 표준출력으로 출력됩니다.

```rust
#[aheui]
fn main() {
    밦밠따방맣희
}
```

```bash
> cargo run --quiet -- 45678
뉮
> echo $?
42
```

### 코드 인용

임의의 유니코드 문자열은 적법한 아희 코드가 될 수 있지만, Rust 는 코드를 파싱 한 이후에 매크로를 평가하기 때문에
아희 러스트 매크로 안에 작성된 코드는 적법한 아희 코드이전에 적법한 Rust 코드여야 합니다.

다음은 올바른 아희 코드이지만 컴파일되지 않는 아희 러스트 매크로의 예시입니다.

```rust
#[aheui]
fn main() {
  붛
  뭏
  희
}
```

```
error: expected one of `!`, `.`, `::`, `;`, `?`, `{`, `}`, or an operator, found `망`
 --> src/main.rs:6:5
  |
5 |     밯
  |       - expected one of 8 possible tokens here
6 |     망
  |     ^^ unexpected token
```

이 경우 다음의 방법을 시도할 수 있습니다.

1. 적법한 Rust 코드가 되게 만듭니다.

   1. 적당한 키워드나 특수기호들을 삽입해 Rust 코드처럼 보이게 만듭니다.

      ```rust
      #[aheui]
      fn main() {
        붛;
        뭏;
        희;
      }
      ```

   2. 문자열 역시 적법한 Rust expression 이므로 아희 코드를 문자열 따옴표 안에 작성합니다.
      `""` 따옴표 뿐 아니라 `r""`, `r#""#` 등 지원합니다. 아희 러스트 매크로는 따옴표를 제외한 부분을 아희 코드로 인식합니다.

      ```rust
      #[aheui]
      fn main() {
        "
        붛
        뭏
        희
        "
      }
      ```

2. 블록 주석이나, 독스트링 주석으로 코드를 작성합니다. 아희 러스트 매크로는 `/*!`와 `*/`, `//!`를 제외한 부분을 아희 코드로 인식합니다.

   ```rust
   #[aheui]
   fn main() {
     /*!
     붛
     뭏
     희
     */
   }
   ```

   ```rust
   #[aheui]
   fn main() {
     //! 붛
     //! 뭏
     //! 희
   }
   ```
