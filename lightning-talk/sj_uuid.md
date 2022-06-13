# 목차

1. 정의 및 탄생 배경
2. 스펙
3. 사용처
4. Use case, Retry key of LINE Bot messages
5. v5 UUID with open source contribution
6. 3줄 요약
7. References

# 정의 및 탄생 배경

## UUID?

```
범용 고유 식별자(汎用固有識別子, 영어: universally unique identifier, UUID)는 소프트웨어 구축에 쓰이는 식별자 표준으로, 개방 소프트웨어 재단(OSF)이 분산 컴퓨팅 환경(DCE)의 일부로 표준화하였다. - 위키피디아
```

## 탄생 배경

```
네트워크 상에서 서로 모르는 개체들을 식별하고 구별하기 위해서는 각각의 고유한 이름이 필요하다. 이 이름은 고유성(유일성)이 매우 중요하다. 같은 이름을 갖는 개체가 존재한다면 구별이 불가능해 지기 때문이다. 고유성을 완벽하게 보장하려면 중앙관리시스템이 있어서 일련번호를 부여해 주면 간단하지만 동시다발적이고 독립적으로 개발되고 있는 시스템들의 경우 중앙관리시스템은 불가능하다. 개발주체가 스스로 이름을 짓도록 하되 고유성을 충족할 수 있는 방법이 필요하다. 이를 위하여 탄생한 것이 범용고유식별자(UUID)이며 국제기구에서 표준으로 정하고 있다.
```

# 스펙

### RFC4122 문서

> https://datatracker.ietf.org/doc/html/rfc4122

### 샘플

```
// sample
550e8400-e29b-41d4-a716-446655440000
xxxxxxxx-xxxx-Mxxx-Nxxx-xxxxxxxxxxxx -> M = version, N = variant
8자리 - 4자리 - 4자리 - 4자리 - 12자리
A - B - C - D - E
```

### 최초 정의(V1, V2)

| 라벨 | 이름                               | 길이(바이트 / 비트) | 내용                                                       |
| ---- | ---------------------------------- | ------------------- | ---------------------------------------------------------- |
| A    | time_low                           | 4 / 8               | 시간의 low 32비트를 부여하는 정수                          |
| B    | time_mid                           | 2 / 4               | 시간의 middle 16비트를 부여하는 정수                       |
| C    | time_hi_and_version                | 2 / 4               | 최상위 비트에서 4비트 "version", 그리고 시간의 high 12비트 |
| D    | clock_seq_hi_and_res clock_seq_low | 2 / 4               | 최상위 비트에서 1-3비트, 그리고 13-15비트 클럭 시퀀스      |
| E    | node                               | 6 / 12              | 48비트 노드 id                                             |

- 16 옥텟 (128비트)의 수
- 32개의 십육진수로 표현되며 총 36개 문자(32개 문자와 4개의 하이픈)로 된 8-4-4-4-12라는 5개의 그룹을 하이픈으로 구분한다.
  - 0-9, a-f의 16개의 문자와 하이픈('-')으로 구성
- 생성할 때는 알파벳 문자의 경우 소문자로 생성되나 실제 사용시에는 대-소문자를 구분하지 않는다.
  - AAAAAAAA-AAAA-AAAA-AAAA-AAAAAAAAAAAA 와 aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa 는 같은 UUID로 취급한다.

#### 이 스펙은 v1, v2 UUID에서만 해당된다.

### Variant

```
xxxxxxxx-xxxx-xxxx-Nxxx-xxxxxxxxxxxx
```

- UUID의 layout을 결정한다.
- N에 해당하는 값의 3개의 MSB(Most Significant Bit)으로 결정한다

- Variant 0 (0xxx(2), N = 0..7, 1 bit)
  - 사용하지 않음
- Variant 1 (10xx(2), N = 8..b, 2 bits)
  - RFC 문서에 명시된 표준
- Variant 2 (110x(2), N = c..d, 3 bits)
  - Windows System의 GUID에서 사용
- Reserved (111x(2), N = e..f)

  - Microsoft Windows 플랫폼에서 GUID 라고 부르는 값에 사용하던 값

- Variant 1, 2가 현재 사용하고 있는 UUID spec이다.
- 다른점? 인코딩 방식이 다르다.
  - Variant 1 -> Big endian
  - Variant 2 -> Mixed endian(Big + Little)

```
00112233-4455-6677-8899-aabbccddeeff 를 Big endian 방식으로 인코딩하면
result: 00 11 22 33 44 55 66 77 88 99 aa bb cc dd ee ff

00112233-4455-6677-8899-aabbccddeeff 를 Mixed endian 방식으로 인코딩하면
A - B - C - D - E 중 A, B, C는 Little endian, D, E는 Big endian으로 인코딩
result: 33 22 11 00 55 44 77 66 c8 99 aa bb cc dd ee ff
```

### Version

- Version 1
  - 호스트 ID, 타임스탬프 기반 UUID
  - 첫 8자리는 타임스탬프, 나머지 24자리는 호스트 ID 기반으로 생성된다.
- Version 2
  - MAC 주소, 타임스탬프 기반 UUID
  - 스펙이 정의되지 않아 구현할 수 없음
- Version 3, 5
  - NAMESPACE, NAME 기반 UUID
  - 특정한 문자열 값을 기반으로 생성되는 UUID
  - 입력한 문자열에 대해 고유한 UUID가 생성된다.
  - 3은 문자열 해싱 알고리즘으로 MD5, 5는 SHA-1을 사용한다.
- Version 4
  - 특정 값을 기반으로 하지 않는 랜덤한 UUID

### Sample

```python
# uuid_test.py
# https://docs.python.org/ko/3/library/uuid.html
import uuid

name = "morning_reading_study"
print("v1 UUID")
print(uuid.uuid1())
print(uuid.uuid1())
print(uuid.uuid1())
print()
print("v3 UUID")
print(uuid.uuid3(uuid.NAMESPACE_URL, name))
print(uuid.uuid3(uuid.NAMESPACE_URL, name))
print(uuid.uuid3(uuid.NAMESPACE_X500, name))
print(uuid.uuid3(uuid.NAMESPACE_X500, name))
print()
print("v4 UUID")
print(uuid.uuid4())
print(uuid.uuid4())
print(uuid.uuid4())
print()
print("v5 UUID")
print(uuid.uuid5(uuid.NAMESPACE_URL, name))
print(uuid.uuid5(uuid.NAMESPACE_URL, name))
print(uuid.uuid5(uuid.NAMESPACE_X500, name))
print(uuid.uuid5(uuid.NAMESPACE_X500, name))
```

```
v1 UUID
24d846f6-e191-11ec-84c1-3c7d0a1adc9b
24d847c8-e191-11ec-84c1-3c7d0a1adc9b
24d84822-e191-11ec-84c1-3c7d0a1adc9b

v3 UUID
0ec04bb3-005f-3cc0-9801-e016212df158
0ec04bb3-005f-3cc0-9801-e016212df158
3a87ce19-2fda-3caf-9ba6-36ff1d1e2ada
3a87ce19-2fda-3caf-9ba6-36ff1d1e2ada

v4 UUID
8d0b8965-47ee-4bd7-919e-7d22e523ae17
4969b128-6696-4fcd-8c83-17753036d31f
2b1abb46-7321-45cc-a58a-a1e728ecace6

v5 UUID
d7092d86-aa34-5a33-b9bd-ed5d05128903
d7092d86-aa34-5a33-b9bd-ed5d05128903
2c6f6c3d-a1c6-5261-8e94-8d78229fef1e
2c6f6c3d-a1c6-5261-8e94-8d78229fef1e
```

# 사용처

## 특징

```
UUID 표준에 따라 이름을 부여하면 고유성을 완벽하게 보장할 수는 없지만 실제 사용상에서 중복될 가능성이 거의 없다고 인정되기 때문에 많이 사용되고 있다.
```

- 식별자로 사용할 수도 있다.

## v4 UUID의 충돌 확률?

> The short answer is no. With the sheer number of possible combinations (2^128), it would be almost impossible to generate a duplicate unless you are generating trillions of IDs every second, for many years.

> Thus, the probability to find a duplicate within 103 trillion version-4 UUIDs is one in a billion.  
> 따라서 103조개의 v4 UUID 내에서 중복을 찾을 확률은 10억분의 1이다.

- 하지만 mission critical한 부분(예를 들면 금융 서비스의 돈 관련 로직의 식별자)에 사용한다면 unique 제약을 걸어야한다.
  - 희박하지만 중복 UUID가 생성될 가능성이 있기 때문!

# Use case, Retry key of LINE Bot messages

- https://developers.line.biz/en/docs/messaging-api/retrying-api-request/#flow-of-api-request-retry
- LINE Bot API를 통해 유저에게 라인 메시지를 보낼 수 있다.
- 이 때 `X-Line-Retry-Key` 라는 헤더를 추가하고 value로 UUID를 보낼 수 있다.
- LINE Bot API 서버는 retry key로 전달받은 UUID를 특정 라인 메신지의 식별자로 사용한다.
- 중복된 retry key를 받은 경우 이미 발송한 메시지로 판단해 409 CONFLICT로 응답하고 메시지를 보내지 않는다.

# v5 UUID with open source contribution

## 기존에는 LINE Bot API를 호출하기 전 v4 UUID를 생성해 header에 담아서 요청했고, 별도로 생성된 UUID를 DB에 저장하진 않았음

- 하지만 이 경우 매우 희박한 확률이지만 중복 UUID가 생성될 가능성이 있음
- 그리고 LINE Bot API의 409 CONFLICT 로직을 의미있게 사용할 수 없음
- 예를 들어 최초로 메시지를 발송할 때 생성된 UUID를 a라고 가정해보면,
- LINE Bot API 호출 후 Timeout이 발생했으나 실제로 메시지가 전송처리 되는 경우가 생길 수 있음.
- 그러나 Timeout으로 응답을 받았기 때문에 해당 메시지에 대해서 재전송을 시도하고,
- 이 때 최초 발송 시 사용했던 UUID를 RetryKey로 재요청을 하지 않는다면 동일 메시지가 중복발송됨.

## DB에 메시지마다 생성한 UUID를 저장하면 어떨까?

저장해서 관리할 필요가 없는 데이터를 테이블 column을 추가하면서까지 저장하는 것은 오버헤드, DB에 저장하지 않고 해결할 수 있을까?

- 특정 라인 메시지에 대한 고정된 UUID를 RetryKey로 사용할 수 있다면 문제를 해결할 수 있음 → v3, v5 UUID를 사용하자!
- 실제로 LINE Bot API를 호출하는 모듈은 메시지 본문(json)을 몇 가지 메타데이터와 함께 kafka로 받아서 처리하고 있음
- 메타데이터에서 실제 json을 저장하는 테이블의 PK를 입력값으로 넣어 UUID를 생성하면 메시지별로 고유한 UUID를 생성할 수 있음!
- v3에 사용하는 MD5는 굉장히 오래된 MD 알고리즘, 충돌 및 복호화 가능성이 높음

## Java의 STL의 UUID 클래스는 v3, v4 UUID 생성 메서드만을 지원한다.

- v5 UUID는 지원하지 않아 필요하다면 스펙대로 직접 구현해야 한다.
- 이를 위해 maven에 UUID Generator라는 의존성이 있다.
  - https://mvnrepository.com/artifact/com.fasterxml.uuid/java-uuid-generator/4.0.1
- 최신버전 기준 보안 취약점이 발견된 하위 의존성을 사용하고 있다.
- 필요한 것은 v5 버전의 UUID를 생성하는 것 뿐이였고 이를 위해 보안 취약점이 발견된 의존성을 추가하는 것은 조금 미묘...
- 다양한 Java 튜토리얼 글이 올라오는 baeldung에 UUID 관련 포스트를 확인
  - https://www.baeldung.com/java-uuid

## baeldung의 샘플 코드를 관리하는 repository에 있는 v5 UUID를 생성할 수 있는 클래스 코드를 사용하기로 결정

- https://github.com/eugenp/tutorials
- https://github.com/eugenp/tutorials/blob/master/core-java-modules/core-java-uuid/src/main/java/com/baeldung/uuid/UUIDGenerator.java
- 로컬 테스트 후 문제 없음을 확인하고 적용!
- 하기 전에 unused imports 제거 및 IDE에서 지원하는 code inspection를 통한 code cleaning 작업을 진행
  - 작업 후 PR로 올림
  - https://github.com/eugenp/tutorials/pull/12110
  - merge 되었음!

# 3줄 요약

UUID는 스펙과 버전이 있다.
RetryKey 로직을 적절하게 사용해 중복된 메시지 발송에 대한 처리를 LINE Bot API 서버에게 맡길 수 있다.
Java UUID 클래스에는 v3, v4 UUID를 생성하는 메서드만 지원한다. v5 UUID를 생성하려면 직접 구현하거나 레퍼런스 코드를 사용하자.

# References

- https://en.wikipedia.org/wiki/Universally_unique_identifier#Collisions
- https://mattmk.tistory.com/31
- https://python.flowdas.com/library/uuid.html
- https://www.uuidtools.com/uuid-versions-explained
- https://www.sohamkamani.com/uuid-versions-explained/
