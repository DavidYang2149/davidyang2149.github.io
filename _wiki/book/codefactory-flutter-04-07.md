---
layout  : wiki
title   : Must Have 코드팩토리의 플러터 프로그래밍 - 04 ~ 07
date    : 2024-02-16 22:58:00 +0900
updated : 2024-02-16 22:58:00 +0900
tag     : book
resource: 
toc     : true
public  : true
parent  : book
latex   : false
---
* TOC
{:toc}

## Must Have 코드팩토리의 플러터 프로그래밍 04 ~ 07

### 04장 다트 3.0 신규 문법

#### P.108 (4.3)

switch문 : 신규 문법 기능 살펴보기.

#### P.108 (4.3.1)

표현식 기능 : 표현식(expression)으로 사용 가능.

```dart
void main() {
  String todayKR = '월요일';

  String todayEN = switch (todayKR) {
    '월요일' => 'Monday',
    '화요일' => 'Tuesday',
    '수요일' => 'Wednesday',
    '목요일' => 'Thursday',
    '금요일' => 'Friday',
    '토요일' => 'Saturday',
    '일요일' => 'Sunday',
    _ => 'Not Found', // '_'는 default와 동일
  };

  print(todayEN); // Monday
}
```

#### P.111 (4.3.4)

보호구문 : when 키워드를 사용할 수 있음.

```dart
void main() {
  (int a, int b) val = (1, -1);

  switch (val) {
    case (1, _) when val.$2 > 0:
      print('1, _');
      break;
    default:
      print('default');
  }
}
```

### 05장 플러터 입문하기

개요 및 환경설정 + 아이폰 실제 기기 사용하기(P.138) 참조.

### 06장 기본 위젯 알아보기

다양한 위젯 알아보기.

#### P.142 (6.1)

##### 부모 위젯

- Container 위젯 : 자식 위젯을 담기 위한 위젯.

- GestureDetector 위젯 : 플러터에서 제공하는 **제스처 기능**을 위해 사용되는 위젯.

- SizedBox 위젯 : 높이와 너비를 지정하기 위한 위젯.

##### 자식 위젯

- Column 위젯 : children 매개변수에 입력된 위젯들을 **세로**로 배치.

- Row 위젯 : children 매개변수에 입력된 위젯들을 **가로**로 배치.

- ListView 위젯 : 리스트를 구현할 때 사용.

참조 : [공식 문서 - 위젯 정보](https://docs.flutter.dev/ui/widgets)

#### P.147 (6.3)

텍스트 관련 위젯 : Text를 다루는 위젯.

RichText : 다양한 텍스트 스타일을 적용할 수 있는 [위젯](https://api.flutter.dev/flutter/widgets/RichText-class.html).

참조 : [공식 문서 - 텍스트 위젯](https://docs.flutter.dev/ui/widgets/text)

### 07장 앱을 만들려면 알아야 하는 그 밖의 지식

#### P.146 (7.4.1)

플러터에서는 Screen이라는 용어(=프론트엔드의 Page 레벨)를 사용.

### 참고

이 글은 골든래빗 《Must Have 코드팩토리의 플러터 프로그래밍 2판》의 스터디 내용 입니다.

### 스터디

Q. 

A. 
