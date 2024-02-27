---
layout  : wiki
title   : Must Have 코드팩토리의 플러터 프로그래밍 - 08 ~ 09
date    : 2024-02-21 19:58:00 +0900
updated : 2024-02-21 19:58:00 +0900
tag     : book
resource: 
toc     : true
public  : true
parent  : book
latex   : false
---
* TOC
{:toc}

## Must Have 코드팩토리의 플러터 프로그래밍 08 ~ 09

### 08장 [Project] 블로그 웹 앱

#### P.194 (8.1.3)

안드로이드와 iOS 네이티브 설정 : 인터넷 권한과 http 프로토콜 권한 설정이 필요함.

#### P.199 (8.2.2)

권한 및 네이티브 설정하기 : 안드로이드 및 iOS 설정 방법

#### 안드로이드

```xml
<!-- android/app/src/main/AndroidManifest.xml 설정 -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET"/>
    <!-- ... 생략 ... -->
</manifest>
```

```java
// android/app/src/build.gradle
android {
  // ... 생략 ...
  // compileSdkVersion flutter.compileSdkVersion
  compileSdkVersion 33
  // ... 생략 ...
  
  defaultConfig {
    // minSdkVersion flutter.minSdkVersion
    minSdkVersion 20
  }
  // ... 생략 ...
}
```

#### iOS

```xml
<!-- ios/Runner/Info.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <!-- ... 생략 ... -->
  <key>NSAppTransportSecurity</key>
  <dict>
    <key>NSAllowsLocalNetworking</key>
    <true/>
    <key>NSAllowsArbitraryLoadsInWebContent</key>
    <true/>
  </dict>
  <!-- ... 생략 ... -->
</dict>
</plist>
```

### P.213 (8.5)

테스트하기 : 최종 완성본 결과

```dart
// lib/main.dart
import 'package:blog_web_app/screen/home_screen.dart';
import 'package:flutter/material.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();

  runApp(
    MaterialApp(
      home: HomeScreen(),
    ),
  );
}

// lib/screen/home_screen.dart
import 'package:flutter/material.dart';
import 'package:webview_flutter/webview_flutter.dart';

class HomeScreen extends StatelessWidget {
  WebViewController webViewController = WebViewController()
    ..loadRequest(Uri.parse('https://blog.codefactory.ai/'))
    ..setJavaScriptMode(JavaScriptMode.unrestricted);

  HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Colors.orange,
        title: const Text('Code Factory'),
        centerTitle: true,
        actions: [
          IconButton(
            icon: Icon(Icons.home),
            onPressed: () {
              webViewController
                  .loadRequest(Uri.parse('https://blog.codefactory.ai'));
            },
          ),
        ],
      ),
      body: WebViewWidget(
        controller: webViewController,
      ),
    );
  }
}
```

### 09장 [Project] 전자액자

### P.220 (9.1.1)

위젯 생명주기

#### StatelessWidget

1. StatelessWidget 빌드.

2. 생성자 실행.

3. build() 함수 실행.

4. 위젯 화면 렌더링.

* StatelessWidget에 매개변수가 변경될 경우, 인스턴스를 아예 새로 생성한 후 기존 인스턴스를 대체해서 변경사항을 화면에 반영함.

#### StatefulWidget

3가지 생명주기가 존재함.

##### 상태 변경이 없는 생명주기

1. StatefulWidget 생성자 실행.

2. createState() 함수 실행 : StatefulWidget과 연동되는 State 생성.

3. initState() 함수 실행 : State 생성 순간에 단 한번만 실행됨.

4. didChangeDependencies() 함수 실행 : BuildContext가 제공됨, State 값이 변경되면 재실행됨.

5. State 상태가 dirty로 설정됨(build()가 재실행돼야 하는 상태).

6. build() 함수 실행 및 UI 반영.

7. State 상태 clean으로 변경.

8. 위젯이 위젯 트리에서 사라지며 deactivate()가 실행(State가 일시적 또는 영구적으로 삭제됨).

9. dispose()가 실행됨(위젯이 영구적으로 삭제됨).

##### StatefulWidget 생성자의 매개변수가 변경됐을 때 생명주기

1. StatefulWidget 생성자 실행.

2. State의 didupdateWidget()함수 실행.

3. State 상태가 dirty로 설정됨(build()가 재실행돼야 하는 상태).

4. build() 함수 실행 및 UI 반영.

5. State 상태 clean으로 변경.

##### State 자체적으로 build()를 재실행할 때 생명주기

1. setState() 함수 실행.

2. State 상태가 dirty로 설정됨(build()가 재실행돼야 하는 상태).

3. build() 함수 실행 및 UI 반영.

4. State 상태 clean으로 변경.

### P.223 (9.1.2)

Timer : 특정 시간이 지난 후에 일회성 또는 지속적으로 함수를 실행

```dart
Timer.periodic(
  Duration(seconds: 3), // 주기
  (Timer timer) {}, // 콜백 함수
);
```

### P.224 (9.2)

사전 준비 : [이미지 경로](https://github.com/codefactory-co/flutter-golden-rabbit-novice-v2)

### P.231 (9.4.2)

상태바 색상 변경하기

```dart
import 'package:flutter/material.dart';
/// 상태바 변경을 위한 import
import 'package:flutter/services.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    /// 상태바를 라이트(흰색)으로 변경
    SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle.light);

    return Scaffold(
      body: PageView(
        children: [1, 2, 3, 4, 5]
            .map((number) => Image.asset('asset/image/image_$number.jpeg',
                fit: BoxFit.cover))
            .toList(),
      ),
    );
  }
}
```

### P.237 (9.5)

테스트하기 : 최종 완성본 결과

```dart
// lib/main.dart
import 'package:image_carousel/screen/home_screen.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(
    MaterialApp(
      home: HomeScreen(),
    ),
  );
}

// lib/screen/home_screen.dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

class HomeScreen extends StatefulWidget {
  const HomeScreen({Key? key}) : super(key: key);

  @override
  _HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  final PageController pageController = PageController();

  @override
  void initState() {
    super.initState();

    Timer.periodic(Duration(seconds: 3), (timer) {
      int nextPage = pageController.page!.toInt() + 1;

      if (nextPage == 5) {
        nextPage = 0;
      }

      pageController.animateToPage(
        nextPage,
        duration: Duration(milliseconds: 500),
        curve: Curves.ease,
      );
    });
  }

  @override
  Widget build(BuildContext context) {
    SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle.light);

    return Scaffold(
      body: PageView(
        controller: pageController,
        children: [1, 2, 3, 4, 5]
            .map((number) => Image.asset('asset/image/image_$number.jpeg',
                fit: BoxFit.cover))
            .toList(),
      ),
    );
  }
}
```

### 참고

이 글은 골든래빗 《Must Have 코드팩토리의 플러터 프로그래밍 2판》의 스터디 내용 입니다.

### 스터디

Q. 

A. 
