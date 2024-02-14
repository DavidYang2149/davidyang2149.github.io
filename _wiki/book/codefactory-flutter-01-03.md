---
layout  : wiki
title   : Must Have 코드팩토리의 플러터 프로그래밍 - 01 ~ 03
date    : 2024-02-10 21:35:00 +0900
updated : 2024-02-14 21:59:00 +0900
tag     : book
toc     : true
public  : true
parent  : book
latex   : false
---
* TOC
{:toc}

## Must Have 코드팩토리의 플러터 프로그래밍 01 ~ 03

### 01장 다트 마스터하기

#### P.47 (1.3.4~1.3.5)

var 타입 : 변수에 값이 들어가면 자동으로 타입을 추론.

dynamic 타입 : `var`과 비슷하나 타입에 대한 변경이 가능 함.

var 타입과 dynamic 타입의 **공통점** : 값 선언 이후, 값을 재설정할 수 있음.

var 타입과 dynamic 타입의 **차이점** : 값 선언 이후, 타입 변경이 가능한지가 다름

Q. 001 : `var` 혹은 `dynamic`을 사용하는 것을 지양해야 하는가? 언제 사용할 수 있을까?

```dart
void main() {
  var varName = 'dart';
  print(varName);
  
  varName = 'Dart!';
  print(varName);

  varName = 1;
  print(varName); // Error: A value of type 'int' can't be assigned to a variable of type 'String'.

  dynamic dynamicName = 'dart';
  print(dynamicName);
  
  dynamicName = 1;
  print(dynamicName);
}
```

#### P.48 (1.3.6)

final : 런타임 상수

const : 빌드타임 상수

final과 const의 공통점 : 값 선언 이후, 원시 값을 변경할 수 없음.

final과 const의 차이점 : final은 런타임(실행 시)에 선언 가능. const는 빌드 타임에 값이 선언되어야 함.

```dart
void main() {
  final DateTime finalNow = DateTime.now();
  print(finalNow);
  
  const DateTime constNow = DateTime.now();
  print(constNow); // Error: Cannot invoke a non-'const' constructor where a const expression is expected.
}
```

#### P.53 (1.4.1)

List 타입의 fold()함수 : reduce와 실행되는 논리는 같지만 return되는 타입이 어떤 것이든 상관 없음.

```dart
void main() {
  List<String> blackPinkList = ['리사', '지수', '제니', '로제'];
  final allMembers = blackPinkList.fold<int>(0, (value, element) {
    print('value : ${value} \ element : ${element}  \ element.length : ${element.length}');
    return value + element.length;
  });
  
  print(allMembers);

  // 실행결과 
  // value : 0  element : 리사   element.length : 2
  // value : 2  element : 지수   element.length : 2
  // value : 4  element : 제니   element.length : 2
  // value : 6  element : 로제   element.length : 2
  // 8
}
```

#### P.57 (1.5.2)

null 관련 연산자 `??=` : 기존 값이 null일 경우 값이 주입 됨.(독특하고 기특해보이는 녀석)

```dart
void main() {
  var message = null;
  print(message);
  
  message ??= '드... 드리겠습니다!';
  print(message);
}
```

#### P.60 (1.6.2)

switch문 : break 키워드가 생략가능 함. continue를 사용하면 이어서 점프가 가능 함.

```dart
void main() {
  var command = 'PENDING';
  switch (command) {
    case 'CLOSED':
      print('executeClosed();');
    case 'PENDING':
      print('executePending();');
    case 'APPROVED':
      print('executeApproved();');
    case 'DENIED':
      print('executeDenied();');
    case 'OPEN':
      print('executeOpen();');
    default:
      print('executeUnknown();');
  }

  var continueCommand = 'OPEN';
  switch (continueCommand) {
    case 'OPEN':
      print('executeOpen();');
      continue newCase; // Continues executing at the newCase label.

    case 'DENIED': // Empty case falls through.
    case 'CLOSED':
      print('executeClosed();'); // Runs for both DENIED and CLOSED,

    newCase:
    case 'PENDING':
      print('executeNowClosed();'); // Runs for both OPEN and PENDING.
  }
}
```

참조 : [switch문 공식문서](https://dart.dev/language/branches#switch-statements)

#### P.63 (1.7.1)

네임드 파라미터 : 사용법 숙지해보기

```dart
int addTwoNumbers({ required int a, int b = 3 }) { return a + b; }

void main() {
  print(addTwoNumbers(a: 1, b: 2));
  print(addTwoNumbers(a: 1));
}
```

포지셔널 파라미터 : 독특한 녀석

```dart
int addNumbers(int a, { required int b, int c = 3 }) { return a + b + c; }

void main() {
  print(addNumbers(1, b: 1));
  print(addNumbers(1, b: 1, c: 2));
}
```

### 02장 다트 객체지향 프로그래밍

#### P.74 (2.2)

this 키워드 : 클래스에 종속된 값을 지칭할 때 사용. 함수 내부에 같은 이름의 변수가 없으면 **this 키워드**를 생략할 수 있음.

```dart
class Idol {
  String name = '블랙핑크';

  void sayName() {
    print('저는 ${this.name}입니다'); // OK
    print('스마트한 저는 ${name}입니다'); // OK!
  }
}
```

#### P.76 (2.2.2)

네임드 생성자 : 네임드 파라미터와 비슷한 개념. 클래스를 생성하는 방법을 명시하고 싶을 때 사용.

```dart
class Idol {
  final String name;
  final int membersCount;

  Idol(String name, int membersCount) : this.name = name, this.membersCount = membersCount;

  // 네임드 생성자 방식
  Idol.fromMap(Map<String, dynamic> map) : this.name = map['name'], this.membersCount = map['membersCount'];

  void sayName() {
    print('저는 ${this.name}입니다. 인원은 ${this.membersCount}명 입니다.');
  }
}

void main() {
  Idol blackPink = Idol('블랙핑크', 4);
  blackPink.sayName();
  
  Idol bts = Idol.fromMap({ 'name' : 'BTS', 'membersCount' : 7 }); // 네임드 생성자 방식
  bts.sayName();
}
```

#### P.78 (2.2.3)

프라이빗 변수 : '_'로 변수명을 지음.

```dart
class Idol {
  String _name;

  Idol(this._name);
}

void main() {
  Idol blackPink = Idol('블랙핑크');
  print(blackPink._name); // 현재는 같은 파일이라 접근이 가능 / 만약 클래스가 다른 파일에 설정되어 있다면 접근 불가
}
```

#### P.84 (2.6)

믹스인 : 특정 클래스에 원하는 기능들만 골라 넣음. `mixin {변수명} on {대상}`로 선언하고 `with`로 사용.

Q. 002 : `mixin`을 왜 사용해야 하는가? 상속으로 대체할 수 있지 않은가?

```dart
class Idol {
  final String name;
  final int membersCount;

  Idol(String name, int membersCount) : this.name = name, this.membersCount = membersCount;

  // 네임드 생성자 방식
  Idol.fromMap(Map<String, dynamic> map) : this.name = map['name'], this.membersCount = map['membersCount'];

  void sayName() {
    print('저는 ${this.name}입니다. 인원은 ${this.membersCount}명 입니다.');
  }
}

mixin IdolSingMixin on Idol {
  void sing() {
    print('${this.name}가 노래를 부릅니다.');
  }
}

class BoyGroup extends Idol with IdolSingMixin {
  BoyGroup(super.name, super.memberCount);

  void sayBoyGroup() {
    print('남자 아이돌입니다.');
  }
}

void main() {
  BoyGroup bts = BoyGroup('BTS', 7);
  bts.sayName();
  bts.sayBoyGroup();
  
  bts.sing();
}
```

#### P.87 (2.8)

제네릭 : 제네릭 문자를 배워보기

- T : 변수 타입
- E : 리스트 내부 요소들의 타입
- K : 키를 표현
- V : 값을 표현

### 03장 다트 비동기 프로그래밍

#### P.97 (3.4)

Stream : 지속적으로 값을 반환받을 때 사용. (JavaScript의 Generator와 비슷)

```dart
import 'dart:async';

void main() {
  final controller = StreamController();
  final stream = controller.stream;
  
  final streamListener1 = stream.listen((value) {
    print(value);
  });
  
  controller.sink.add(1);
  controller.sink.add(2);
  controller.sink.add(3);
  controller.sink.add(4);
}
```

### 참고

이 글은 골든래빗 《Must Have 코드팩토리의 플러터 프로그래밍 2판》의 스터디 내용 입니다.

### 스터디

Q. [P.47 (1.3.4~1.3.5)] `var` 혹은 `dynamic`을 사용하는 것을 지양해야 하는가? 언제 사용할 수 있을까?

A. 어떤 타입이 들어올지 예측하기 어려울 때 사용.

Q. [P.84 (2.6)] `mixin`을 왜 사용해야 하는가? 상속으로 대체할 수 있지 않은가?

A. log 기능에서 사용할 수 있음. (상속은 다중 상속이 안되기 때문에...)

Q. 추상클래스 vs 인터페이스

A. 차이점이 무엇일까? - 좀 더 조사해보기.
