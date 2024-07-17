# GBNF 가이드

GBNF (GGML BNF)는 `llama.cpp`에서 모델 출력을 제한하는 데 사용되는 [형식 문법](https://ko.wikipedia.org/wiki/%EC%88%98%EC%82%AC_%EB%B0%A9%EB%B2%95)을 정의하는 형식입니다. 예를 들어, 이를 사용하여 모델이 유효한 JSON을 생성하거나, 오직 이모티콘으로만 말하도록 강제할 수 있습니다. GBNF 문법은 `examples/main` 및 `examples/server`에서 다양한 방식으로 지원됩니다.

## 배경

[백스-나우어 형식 (BNF)](https://ko.wikipedia.org/wiki/백스%E2%80%93나우어_형식)은 프로그래밍 언어, 파일 형식 및 프로토콜과 같은 형식 언어의 문법을 설명하는 표기법입니다. GBNF는 BNF의 확장으로 주로 몇 가지 현대적인 정규식과 유사한 기능을 추가합니다.

## 기본 개념

GBNF에서 우리는 *생산 규칙*을 정의합니다. 이 규칙은 *비종료 기호* (규칙 이름)를 *종료 기호* (문자, 특히 Unicode [코드 포인트](https://ko.wikipedia.org/wiki/코드_포인트))와 다른 비종료 기호의 순서로 대체하는 방법을 지정합니다. 생산 규칙의 기본 형식은 `비종료 기호 ::= 순서...` 입니다.

## 예시

더 깊이 있게 들어가기 전에, `grammars/chess.gbnf`와 같은 체스 표기법 문법에서 보여주는 몇 가지 기능을 살펴보겠습니다.
```
# `root` specifies the pattern for the overall output
root ::= (
    # it must start with the characters "1. " followed by a sequence
    # of characters that match the `move` rule, followed by a space, followed
    # by another move, and then a newline
    "1. " move " " move "\n"

    # it's followed by one or more subsequent moves, numbered with one or two digits
    ([1-9] [0-9]? ". " move " " move "\n")+
)

# `move` is an abstract representation, which can be a pawn, nonpawn, or castle.
# The `[+#]?` denotes the possibility of checking or mate signs after moves
move ::= (pawn | nonpawn | castle) [+#]?

pawn ::= ...
nonpawn ::= ...
castle ::= ...
```

## 비종합 기호와 종합 기호

비종합 기호(규칙 이름)는 종합 기호와 다른 비종합 기호의 패턴을 나타냅니다.  `move`, `castle`, 또는 `check-mate`와 같이 연장선 아래 소문자 단어여야 합니다.

종합 기호는 실제 문자입니다 ([코드 포인트](https://ko.wikipedia.org/wiki/코드_포인트)). `"1"` 또는 `"O-O"`와 같이 순서로 지정하거나 `[1-9]` 또는 `[NBKQR]`와 같이 범위로 지정할 수 있습니다.

## 문자 및 문자 범위

단어들은 유니코드의 전체 범위를 지원합니다. 유니코드 문자는 문법에 직접 지정할 수 있습니다. 예를 들어 `hiragana ::= [ぁ-ゟ]` 또는 이스케이프 문자를 사용하여 지정할 수 있습니다: 8비트 (`\xXX`), 16비트 (`\uXXXX`) 또는 32비트 (`\UXXXXXXXX`).

문자 범위는 `^`를 사용하여 부정할 수 있습니다:
```
single-line ::= [^\n]+ "\n"`
```

## 순서와 선택


순서가 중요합니다. 예를 들어, `"1. " move " " move "\n"` 에서는 `"1. "` 가 첫 번째 `move` 전에 와야 하며, 이와 같이 모든 순서가 중요합니다.

Alternatives, denoted by `|`, give different sequences that are acceptable. For example, in `move ::= pawn | nonpawn | castle`, `move` can be a `pawn` move, a `nonpawn` move, or a `castle`.

괄호 `()`를 사용하여 순서를 그룹화할 수 있습니다. 이를 통해 더 큰 규칙에 대안을 포함하거나 반복 및 선택적 기호(아래)를 순서에 적용할 수 있습니다.

## 반복 및 선택적 기호

- 기호 또는 문자열 뒤에 `*`를 붙이면 0회 이상 반복될 수 있습니다 ( `{0,}` 와 동일합니다).
- `+`는 기호 또는 문자열이 한 번 이상 나타나야 함을 나타냅니다 ( `{1,}` 와 동일합니다).
- `?`는 preceding 기호 또는 문자열을 선택적으로 만듭니다 ( `{0,1}` 와 동일합니다).
- `{m}`는 preceding 기호 또는 문자열을 정확히 `m` 번 반복합니다
- `{m,}`는 preceding 기호 또는 문자열을 최소 `m` 번 반복합니다
- `{m,n}`는 preceding 기호 또는 문자열을 `m` 에서 `n` 까지 (포함) 반복합니다
- `{0,n}`는 preceding 기호 또는 문자열을 최대 `n` 번 (포함) 반복합니다

## 주석 및 새 줄

`#`를 사용하여 주석을 지정할 수 있습니다.:
```
# defines optional whitespace
ws ::= [ \t\n]+
```

Newlines are allowed between rules and between symbols or sequences nested inside parentheses. Additionally, a newline after an alternate marker `|` will continue the current rule, even outside of parentheses.

## 루트 규칙

완전한 문법에서 `root` 규칙은 항상 문법의 시작점을 정의합니다. 즉, 전체 출력이 일치해야 하는 것을 지정합니다.

```
# a grammar for lists
root ::= ("- " item)+
item ::= [^\n]+ "\n"
```

## 다음 단계

이 가이드는 간략한 개요를 제공합니다. 이 디렉토리 (`grammars/`)의 GBNF 파일을 확인하여 전체 문법의 예를 살펴보세요. 다음과 같은 방법으로 시도해 볼 수 있습니다:
```
./llama-cli -m <model> --grammar-file grammars/some-grammar.gbnf -p 'Some prompt'
```

`llama.cpp`는 JSON 스키마를 미리 또는 요청마다 문법으로 변환할 수 있습니다. 아래를 참조하세요.

## 문제 해결

현재 문법은 성능 문제가 있습니다 (https://github.com/ggerganov/llama.cpp/issues/4218 참조).

### 효율적인 선택적 반복

흔한 패턴은 패턴 `x`를 최대 N번 반복할 수 있도록 허용하는 것입니다.

문맥적으로는 올바르지만, 문법 `x? x? x?.... x?` (N번 반복)는 샘플링 속도가 매우 느려질 수 있습니다. 대신 `x{0,N}` (또는 이전 llama.cpp 버전의 N-깊이  pertanian `(x (x (x ... (x)?...)?)?)?`)를 사용할 수 있습니다.

## GBNF 문법 사용하기

GBNF 문법을 사용할 수 있습니다.

- [llama-server](../examples/server)의 완성 앤드포인트에서 `grammar` 바디 필드로 전달합니다.
- [llama-cli](../examples/main)에서 `--grammar` 및 `--grammar-file` 플래그로 전달합니다.
- [llama-gbnf-validator](../examples/gbnf-validator) 도구와 함께 문자열에 대해 테스트합니다.

## JSON 스키마 → GBNF

`llama.cpp`는 https://json-schema.org/의 일부를 GBNF 문법으로 변환할 수 있습니다.

- [llama-server](../examples/server) 에서:
    - `json_schema` 바디 필드로 전달되는 모든 완성 앤드포인트
    - `/chat/completions` 엔드포인트에서 `result_format` 바디 필드 내부에 전달됩니다 (예: `{"type", "json_object", "schema": {"items": {}}}`)
- [llama-cli](../examples/main) 에서 `--json` / `-j` 플래그로 전달됩니다.
- 미리 문법으로 변환하려면:
    - CLI에서 [examples/json_schema_to_grammar.py](../examples/json_schema_to_grammar.py)를 사용합니다.
    - JavaScript에서 [json-schema-to-grammar.mjs](../examples/server/public/json-schema-to-grammar.mjs)를 사용합니다 (이것은 [서버](../examples/server)의 웹 UI에서 사용됩니다).

[tests](../tests/test-json-schema-to-grammar.cpp)를 참조하여 지원되는 기능을 확인할 수 있습니다 (https://github.com/ggerganov/llama.cpp/pull/5978, https://github.com/ggerganov/llama.cpp/pull/6659 & https://github.com/ggerganov/llama.cpp/pull/6555 에서도 사용 예제를 찾을 수 있습니다).

```bash
llama-cli \
  -hfr bartowski/Phi-3-medium-128k-instruct-GGUF \
  -hff Phi-3-medium-128k-instruct-Q8_0.gguf \
  -j '{
    "type": "array",
    "items": {
        "type": "object",
        "properties": {
            "name": {
                "type": "string",
                "minLength": 1,
                "maxLength": 100
            },
            "age": {
                "type": "integer",
                "minimum": 0,
                "maximum": 150
            }
        },
        "required": ["name", "age"],
        "additionalProperties": false
    },
    "minItems": 10,
    "maxItems": 100
  }' \
  -p 'Generate a {name, age}[] JSON array with famous actors of all ages.'
```

<details>

<summary>문법 보기</summary>

명령줄에서 스키마를 어떤 방식으로 변환할 수 있는지 확인하세요.

```bash
examples/json_schema_to_grammar.py name-age-schema.json
```

```
char ::= [^"\\\x7F\x00-\x1F] | [\\] (["\\bfnrt] | "u" [0-9a-fA-F]{4})
item ::= "{" space item-name-kv "," space item-age-kv "}" space
item-age ::= ([0-9] | ([1-8] [0-9] | [9] [0-9]) | "1" ([0-4] [0-9] | [5] "0")) space
item-age-kv ::= "\"age\"" space ":" space item-age
item-name ::= "\"" char{1,100} "\"" space
item-name-kv ::= "\"name\"" space ":" space item-name
root ::= "[" space item ("," space item){9,99} "]" space
space ::= | " " | "\n" [ \t]{0,20}
```

</details>

다음은 알려진 제한 사항 목록입니다 (기여 환영):

- `additionalProperties`는 기본적으로 `false`로 설정되어 있습니다 (빠른 문법 생성 및 홀로우 이션 감소).
- `"additionalProperties": true`는 탈출되지 않은 새 줄이 포함된 키를 생성할 수 있습니다.
- 지원되지 않는 기능은 무음으로 건너뜁니다. 현재는 위에서 언급한 명령줄 Python 변환기를 사용하여 경고를 확인하고, 결과 문법을 검사하거나 [llama-gbnf-validator](../examples/gbnf-validator/gbnf-validator.cpp)와 함께 테스트하는 것이 좋습니다.
- `properties`를 `anyOf` / `oneOf`과 동시에 같은 유형에서 사용할 수 없습니다 (https://github.com/ggerganov/llama.cpp/issues/7703)
- [prefixItems](https://json-schema.org/draft/2020-12/json-schema-core#name-prefixitems)는 작동하지 않습니다 (그러나 [items](https://json-schema.org/draft/2020-12/json-schema-core#name-items)는 작동합니다)
- `minimum`, `exclusiveMinimum`, `maximum`, `exclusiveMaximum`: 현재는 `"type": "integer"`에만 지원되며 `number`에는 지원되지 않습니다.
- 중첩된 `$ref`는 작동하지 않습니다 (https://github.com/ggerganov/llama.cpp/issues/8073)
- [pattern](https://json-schema.org/draft/2020-12/json-schema-validation#name-pattern)은 `^`로 시작하고 `$`으로 끝나야 합니다.
- C++ 버전에서는 원격 `$ref`가 지원되지 않습니다 (Python 및 JavaScript 버전은 https 참조를 가져옵니다)
- `string` [formats](https://json-schema.org/draft/2020-12/json-schema-validation#name-defined-formats)는 `uri`, `email`이 부족합니다.
- [`patternProperties`](https://json-schema.org/draft/2020-12/json-schema-core#name-patternproperties)가 없습니다.

그리고 상태 없는 문법으로 지원하기 어렵거나 너무 느리기 때문에 구현될 가능성이 낮은 다른 지원되지 않는 기능 목록입니다.

- [`uniqueItems`](https://json-schema.org/draft/2020-12/json-schema-validation#name-uniqueitems)
- [`contains`](https://json-schema.org/draft/2020-12/json-schema-core#name-contains) / `minContains`
- `$anchor` (cf. [dereferencing](https://json-schema.org/draft/2020-12/json-schema-core#name-dereferencing))
- [`not`](https://json-schema.org/draft/2020-12/json-schema-core#name-not)
- [조건문](https://json-schema.org/draft/2020-12/json-schema-core#name-keywords-for-applying-subsche) `if` / `then` / `else` / `dependentSchemas`

### additionalProperties에 대한 추가 정보

> [!WARNING]
> JSON 스키마 사양은 `object`가 기본적으로 [추가 속성](https://json-schema.org/understanding-json-schema/reference/object#additionalproperties)을 허용한다고 명시합니다.
> 이는 느리고 홀루시네이션에 취약해 보이므로, 기본적으로 추가 속성을 허용하지 않습니다.
> 어떤 객체의 스키마에서 `"additionalProperties": true`를 설정하면 명시적으로 추가 속성을 허용할 수 있습니다.

[Pydantic](https://pydantic.dev/)를 사용하여 스키마를 생성하는 경우, 각 모델 클래스의 `extra` 구성을 사용하여 추가 속성을 활성화할 수 있습니다:

```python
# pip install pydantic
import json
from typing import Annotated, List
from pydantic import BaseModel, Extra, Field
class QAPair(BaseModel):
    class Config:
        extra = 'allow'  # triggers additionalProperties: true in the JSON schema
    question: str
    concise_answer: str
    justification: str

class Summary(BaseModel):
    class Config:
        extra = 'allow'
    key_facts: List[Annotated[str, Field(pattern='- .{5,}')]]
    question_answers: List[Annotated[List[QAPair], Field(min_items=5)]]

print(json.dumps(Summary.model_json_schema(), indent=2))
```

<details>
<summary>JSON 스키마 및 문법 보기</summary>

```json
{
  "$defs": {
    "QAPair": {
      "additionalProperties": true,
      "properties": {
        "question": {
          "title": "Question",
          "type": "string"
        },
        "concise_answer": {
          "title": "Concise Answer",
          "type": "string"
        },
        "justification": {
          "title": "Justification",
          "type": "string"
        }
      },
      "required": [
        "question",
        "concise_answer",
        "justification"
      ],
      "title": "QAPair",
      "type": "object"
    }
  },
  "additionalProperties": true,
  "properties": {
    "key_facts": {
      "items": {
        "pattern": "^- .{5,}$",
        "type": "string"
      },
      "title": "Key Facts",
      "type": "array"
    },
    "question_answers": {
      "items": {
        "items": {
          "$ref": "#/$defs/QAPair"
        },
        "minItems": 5,
        "type": "array"
      },
      "title": "Question Answers",
      "type": "array"
    }
  },
  "required": [
    "key_facts",
    "question_answers"
  ],
  "title": "Summary",
  "type": "object"
}
```

```
QAPair ::= "{" space QAPair-question-kv "," space QAPair-concise-answer-kv "," space QAPair-justification-kv ( "," space ( QAPair-additional-kv ( "," space QAPair-additional-kv )* ) )? "}" space
QAPair-additional-k ::= ["] ( [c] ([o] ([n] ([c] ([i] ([s] ([e] ([_] ([a] ([n] ([s] ([w] ([e] ([r] char+ | [^"r] char*) | [^"e] char*) | [^"w] char*) | [^"s] char*) | [^"n] char*) | [^"a] char*) | [^"_] char*) | [^"e] char*) | [^"s] char*) | [^"i] char*) | [^"c] char*) | [^"n] char*) | [^"o] char*) | [j] ([u] ([s] ([t] ([i] ([f] ([i] ([c] ([a] ([t] ([i] ([o] ([n] char+ | [^"n] char*) | [^"o] char*) | [^"i] char*) | [^"t] char*) | [^"a] char*) | [^"c] char*) | [^"i] char*) | [^"f] char*) | [^"i] char*) | [^"t] char*) | [^"s] char*) | [^"u] char*) | [q] ([u] ([e] ([s] ([t] ([i] ([o] ([n] char+ | [^"n] char*) | [^"o] char*) | [^"i] char*) | [^"t] char*) | [^"s] char*) | [^"e] char*) | [^"u] char*) | [^"cjq] char* )? ["] space
QAPair-additional-kv ::= QAPair-additional-k ":" space value
QAPair-concise-answer-kv ::= "\"concise_answer\"" space ":" space string
QAPair-justification-kv ::= "\"justification\"" space ":" space string
QAPair-question-kv ::= "\"question\"" space ":" space string
additional-k ::= ["] ( [k] ([e] ([y] ([_] ([f] ([a] ([c] ([t] ([s] char+ | [^"s] char*) | [^"t] char*) | [^"c] char*) | [^"a] char*) | [^"f] char*) | [^"_] char*) | [^"y] char*) | [^"e] char*) | [q] ([u] ([e] ([s] ([t] ([i] ([o] ([n] ([_] ([a] ([n] ([s] ([w] ([e] ([r] ([s] char+ | [^"s] char*) | [^"r] char*) | [^"e] char*) | [^"w] char*) | [^"s] char*) | [^"n] char*) | [^"a] char*) | [^"_] char*) | [^"n] char*) | [^"o] char*) | [^"i] char*) | [^"t] char*) | [^"s] char*) | [^"e] char*) | [^"u] char*) | [^"kq] char* )? ["] space
additional-kv ::= additional-k ":" space value
array ::= "[" space ( value ("," space value)* )? "]" space
boolean ::= ("true" | "false") space
char ::= [^"\\\x7F\x00-\x1F] | [\\] (["\\bfnrt] | "u" [0-9a-fA-F]{4})
decimal-part ::= [0-9]{1,16}
dot ::= [^\x0A\x0D]
integral-part ::= [0] | [1-9] [0-9]{0,15}
key-facts ::= "[" space (key-facts-item ("," space key-facts-item)*)? "]" space
key-facts-item ::= "\"" "- " key-facts-item-1{5,} "\"" space
key-facts-item-1 ::= dot
key-facts-kv ::= "\"key_facts\"" space ":" space key-facts
null ::= "null" space
number ::= ("-"? integral-part) ("." decimal-part)? ([eE] [-+]? integral-part)? space
object ::= "{" space ( string ":" space value ("," space string ":" space value)* )? "}" space
question-answers ::= "[" space (question-answers-item ("," space question-answers-item)*)? "]" space
question-answers-item ::= "[" space question-answers-item-item ("," space question-answers-item-item){4,} "]" space
question-answers-item-item ::= QAPair
question-answers-kv ::= "\"question_answers\"" space ":" space question-answers
root ::= "{" space key-facts-kv "," space question-answers-kv ( "," space ( additional-kv ( "," space additional-kv )* ) )? "}" space
space ::= | " " | "\n" [ \t]{0,20}
string ::= "\"" char* "\"" space
value ::= object | array | string | number | boolean | null
```

</details>

[Zod](https://zod.dev/)를 사용하는 경우, `nonstrict()` / `passthrough()`으로 객체에 추가 속성을 명시적으로 허용하거나 (또는 `z.object(...).strict()` 또는 `z.strictObject(...)`으로 명시적으로 추가 속성을 허용하지 않음) 하지만 [zod-to-json-schema](https://github.com/StefanTerdell/zod-to-json-schema)는 현재 항상 `"additionalProperties": false`로 설정됩니다.

```js
import { z } from 'zod';
import { zodToJsonSchema } from 'zod-to-json-schema';

const Foo = z.object({
  age: z.number().positive(),
  email: z.string().email(),
}).strict();

console.log(zodToJsonSchema(Foo));
```

<details>
<summary>JSON 스키마 및 문법 보기</summary>

```json
{
  "type": "object",
  "properties": {
    "age": {
      "type": "number",
      "exclusiveMinimum": 0
    },
    "email": {
      "type": "string",
      "format": "email"
    }
  },
  "required": [
    "age",
    "email"
  ],
  "additionalProperties": false,
  "$schema": "http://json-schema.org/draft-07/schema#"
}
```

```
age-kv ::= "\"age\"" space ":" space number
char ::= [^"\\\x7F\x00-\x1F] | [\\] (["\\bfnrt] | "u" [0-9a-fA-F]{4})
decimal-part ::= [0-9]{1,16}
email-kv ::= "\"email\"" space ":" space string
integral-part ::= [0] | [1-9] [0-9]{0,15}
number ::= ("-"? integral-part) ("." decimal-part)? ([eE] [-+]? integral-part)? space
root ::= "{" space age-kv "," space email-kv "}" space
space ::= | " " | "\n" [ \t]{0,20}
string ::= "\"" char* "\"" space
```

</details>
