---
layout  : wiki
title   : Must Have 코드팩토리의 플러터 프로그래밍 - 20 ~ 21
date    : 2024-04-14 20:58:00 +0900
updated : 2024-04-14 20:58:00 +0900
tag     : book
resource: 
toc     : true
public  : true
parent  : book
latex   : false
---
* TOC
{:toc}

## Must Have 코드팩토리의 플러터 프로그래밍 20 ~ 21

### 20장 [Project #4] 파이어베이스 연동하기

### P.604 (20.4)

테스트하기 : 최종 완성본 결과

```dart
// ... Firebase 적용 부분 기록 ...

// lib/main.dart
import 'package:calendar_scheduler/screen/home_screen.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/material.dart';
import 'package:intl/date_symbol_data_local.dart';
import 'package:calendar_scheduler/database/drift_database.dart';
import 'package:get_it/get_it.dart';
import 'package:calendar_scheduler/provider/schedule_provider.dart';
import 'package:calendar_scheduler/repository/schedule_repository.dart';
import 'package:provider/provider.dart';
import 'package:calendar_scheduler/firebase_options.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );

  await initializeDateFormatting();

  final database = LocalDatabase();

  final repository = ScheduleRepository();
  final scheduleProvider = ScheduleProvider(repository: repository);

  GetIt.I.registerSingleton<LocalDatabase>(database);

  runApp(
    MaterialApp(
      debugShowCheckedModeBanner: false,
      home: HomeScreen(),
    ),
  );
}

// lib/component/schedule_bottom_sheet.dart
import 'package:flutter/material.dart';
import 'package:calendar_scheduler/component/custom_text_field.dart';
import 'package:calendar_scheduler/const/colors.dart';
import 'package:calendar_scheduler/model/schedule_model.dart';
import 'package:uuid/uuid.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

class ScheduleBottomSheet extends StatefulWidget {
  final DateTime selectedDate;

  const ScheduleBottomSheet({
    required this.selectedDate,
    Key? key,
  }) : super(key: key);

  @override
  State<ScheduleBottomSheet> createState() => _ScheduleBottomSheetState();
}

class _ScheduleBottomSheetState extends State<ScheduleBottomSheet> {
  final GlobalKey<FormState> formKey = GlobalKey();

  int? startTime; 
  int? endTime; 
  String? content; 

  @override
  Widget build(BuildContext context) {
    final bottomInset = MediaQuery.of(context).viewInsets.bottom;

    return Form(
      key: formKey, 
      child: SafeArea(
        child: Container(
          height: MediaQuery.of(context).size.height / 2 +
              bottomInset, 
          color: Colors.white,
          child: Padding(
            padding:
                EdgeInsets.only(left: 8, right: 8, top: 8, bottom: bottomInset),
            child: Column(
              children: [
                Row(
                  children: [
                    Expanded(
                      child: CustomTextField(
                        label: '시작 시간',
                        isTime: true,
                        onSaved: (String? val) {
                          startTime = int.parse(val!);
                        },
                        validator: timeValidator,
                      ),
                    ),
                    const SizedBox(width: 16.0),
                    Expanded(
                      child: CustomTextField(
                        label: '종료 시간',
                        isTime: true,
                        onSaved: (String? val) {
                          endTime = int.parse(val!);
                        },
                        validator: timeValidator,
                      ),
                    ),
                  ],
                ),
                SizedBox(height: 8.0),
                Expanded(
                  child: CustomTextField(
                    label: '내용',
                    isTime: false,
                    onSaved: (String? val) {
                      content = val;
                    },
                    validator: contentValidator,
                  ),
                ),
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: () => onSavePressed(context),
                    style: ElevatedButton.styleFrom(
                      primary: PRIMARY_COLOR,
                    ),
                    child: Text('저장'),
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }

  void onSavePressed(BuildContext context) async {    if (formKey.currentState!.validate()) {
      formKey.currentState!.save();

      final schedule = ScheduleModel(
        id: Uuid().v4(),
        content: content!,
        date: widget.selectedDate,
        startTime: startTime!,
        endTime: endTime!,
      );

      await FirebaseFirestore.instance
          .collection(
        'schedule',
      )
          .doc(schedule.id)
          .set(schedule.toJson());

      Navigator.of(context).pop();
    }
  }

  String? timeValidator(String? val) {
    if (val == null) {
      return '값을 입력해주세요';
    }

    int? number;

    try {
      number = int.parse(val);
    } catch (e) {
      return '숫자를 입력해주세요';
    }

    if (number < 0 || number > 24) {
      return '0시부터 24시 사이를 입력해주세요';
    }

    return null;
  } 

  String? contentValidator(String? val) {
    if (val == null || val.length == 0) {
      return '값을 입력해주세요';
    }

    return null;
  } 
}

// lib/screen/home_screen.dart
import 'package:calendar_scheduler/model/schedule_model.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:flutter/material.dart';
import 'package:calendar_scheduler/component/main_calendar.dart';
import 'package:calendar_scheduler/component/schedule_card.dart';
import 'package:calendar_scheduler/component/today_banner.dart';
import 'package:calendar_scheduler/component/schedule_bottom_sheet.dart';
import 'package:calendar_scheduler/const/colors.dart';
import 'package:get_it/get_it.dart';
import 'package:calendar_scheduler/database/drift_database.dart';
import 'package:provider/provider.dart';
import 'package:calendar_scheduler/provider/schedule_provider.dart';

class HomeScreen extends StatefulWidget {
  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  DateTime selectedDate = DateTime.utc(
    DateTime.now().year,
    DateTime.now().month,
    DateTime.now().day,
  );

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      floatingActionButton: FloatingActionButton(
        backgroundColor: PRIMARY_COLOR,
        onPressed: () {
          showModalBottomSheet(
            context: context,
            isDismissible: true,
            isScrollControlled: true,
            builder: (_) => ScheduleBottomSheet(
              selectedDate: selectedDate,
            ),
          );
        },
        child: Icon(
          Icons.add,
        ),
      ),
      body: SafeArea(
        child: Column(
          children: [
            MainCalendar(
              selectedDate: selectedDate,
              onDaySelected: (selectedDate, focusedDate) =>
                  onDaySelected(selectedDate, focusedDate, context),
            ),
            SizedBox(height: 8.0),
            StreamBuilder<QuerySnapshot>(
              stream: FirebaseFirestore.instance
                  .collection(
                'schedule',
              )
                  .where(
                'date',
                isEqualTo:
                '${selectedDate.year}${selectedDate.month}${selectedDate.day}',
              )
                  .snapshots(),
              builder: (context, snapshot) {
                return TodayBanner(
                  selectedDate: selectedDate,
                  count: snapshot.data?.docs.length ?? 0,
                );
              },
            ),
            SizedBox(height: 8.0),
            Expanded(
              child: StreamBuilder<QuerySnapshot>(
                stream: FirebaseFirestore.instance
                    .collection(
                      'schedule',
                    )
                    .where(
                      'date',
                      isEqualTo:
                          '${selectedDate.year}${selectedDate.month}${selectedDate.day}',
                    )
                    .snapshots(),
                builder: (context, snapshot) {
                  if (snapshot.hasError) {
                    return Center(
                      child: Text('일정 정보를 가져오지 못했습니다.'),
                    );
                  }

                  if (snapshot.connectionState == ConnectionState.waiting) {
                    return Container();
                  }

                  final schedules = snapshot.data!.docs
                      .map(
                        (QueryDocumentSnapshot e) => ScheduleModel.fromJson(
                            json: (e.data() as Map<String, dynamic>)),
                      )
                      .toList();

                  return ListView.builder(
                    itemCount: schedules.length,
                    itemBuilder: (context, index) {
                      final schedule = schedules[index];

                      return Dismissible(
                        key: ObjectKey(schedule.id),
                        direction: DismissDirection.startToEnd,
                        onDismissed: (DismissDirection direction) {
                          FirebaseFirestore.instance
                              .collection('schedule')
                              .doc(schedule.id)
                              .delete();
                        },
                        child: Padding(
                          padding: const EdgeInsets.only(
                              bottom: 8.0, left: 8.0, right: 8.0),
                          child: ScheduleCard(
                            startTime: schedule.startTime,
                            endTime: schedule.endTime,
                            content: schedule.content,
                          ),
                        ),
                      );
                    },
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }

  void onDaySelected(
    DateTime selectedDate,
    DateTime focusedDate,
    BuildContext context,
  ) {
    setState(() {
      this.selectedDate = selectedDate;
    });
  }
}
```

### 21장 [Project #5] JWT를 이용한 인증하기

### P.655 (21.5)

테스트하기 : 최종 완성본 결과

```dart
// ... JWT 적용 부분 기록 ...

// lib/screen/auth_screen.dart
import 'package:calendar_scheduler/component/login_text_field.dart';
import 'package:calendar_scheduler/const/colors.dart';
import 'package:calendar_scheduler/database/drift_database.dart';
import 'package:calendar_scheduler/provider/schedule_provider.dart';
import 'package:calendar_scheduler/screen/home_screen.dart';
import 'package:dio/dio.dart';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class AuthScreen extends StatefulWidget {
  const AuthScreen({Key? key}) : super(key: key);

  @override
  State<AuthScreen> createState() => _AuthScreenState();
}

class _AuthScreenState extends State<AuthScreen> {
  GlobalKey<FormState> formKey = GlobalKey<FormState>();

  String email = '';
  String password = '';

  @override
  Widget build(BuildContext context) {
    final provider = context.watch<ScheduleProvider>();

    return Scaffold(
      body: Padding(
        padding: const EdgeInsets.symmetric(horizontal: 16.0),
        child: Form(
          key: formKey,
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              Align(
                alignment: Alignment.center,
                child: Image.asset(
                  'assets/img/logo.png',
                  width: MediaQuery.of(context).size.width * 0.5,
                ),
              ),
              const SizedBox(height: 16.0),
              LoginTextField(
                onSaved: (val) {
                  email = val!;
                },
                validator: (val) {
                  if (val?.isEmpty ?? true) {
                    return '이메일을 입력해주세요.';
                  }

                  RegExp reg = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$');

                  if (!reg.hasMatch(val!)) {
                    return '이메일 형식이 올바르지 않습니다.';
                  }
                  return null;
                },
                hintText: '이메일',
              ),
              const SizedBox(height: 8.0),
              LoginTextField(
                onSaved: (val) {
                  password = val!;
                },
                obscureText: true,
                validator: (val) {
                  if (val?.isEmpty ?? true) {
                    return '비밀번호를 입력해주세요.';
                  }

                  if (val!.length < 4 || val.length > 8) {
                    return '비밀번호는 4~8자 사이로 입력 해주세요!';
                  }
                  return null;
                },
                hintText: '비밀번호',
              ),
              const SizedBox(height: 16.0),
              ElevatedButton(
                style: ElevatedButton.styleFrom(
                  backgroundColor: SECONDARY_COLOR,
                ),
                onPressed: () async {
                  onRegisterPress(provider);
                },
                child: Text('회원가입'),
              ),
              ElevatedButton(
                style: ElevatedButton.styleFrom(
                  backgroundColor: SECONDARY_COLOR,
                ),
                onPressed: () async {
                  onLoginPress(provider);
                },
                child: Text('로그인'),
              ),
            ],
          ),
        ),
      ),
    );
  }

  onRegisterPress(ScheduleProvider provider) async {
    if (!saveAndValidateForm()) {
      return;
    }

    String? message;

    try {
      await provider.register(
        email: email,
        password: password,
      );
    } on DioError catch (e) {
      message = e.response?.data['message'] ?? '알 수 없는 오류가 발생했습니다.';
    } catch (e) {
      message = '알 수 없는 오류가 발생했습니다.';
    } finally {
      if (message != null) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text(message),
          ),
        );
      } else {
        Navigator.of(context).push(
          MaterialPageRoute(
            builder: (_) => HomeScreen(),
          ),
        );
      }
    }
  }

  onLoginPress(ScheduleProvider provider) async {
    if (!saveAndValidateForm()) {
      return;
    }

    String? message;

    try {
      await provider.login(
        email: email,
        password: password,
      );
    } on DioError catch (e) {
      message = e.response?.data['message'] ?? '알 수 없는 오류가 발생했습니다.';
    } catch (e) {
      message = '알 수 없는 오류가 발생했습니다.';
    } finally {
      if (message != null) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content: Text(message),
          ),
        );
      } else {
        Navigator.of(context).push(
          MaterialPageRoute(
            builder: (_) => HomeScreen(),
          ),
        );
      }
    }
  }

  bool saveAndValidateForm() {
    if (!formKey.currentState!.validate()) {
      return false;
    }

    formKey.currentState!.save();

    return true;
  }
}

// lib/main.dart
import 'package:calendar_scheduler/repository/auth_repository.dart';
import 'package:calendar_scheduler/screen/auth_screen.dart';
import 'package:calendar_scheduler/screen/home_screen.dart';
import 'package:flutter/material.dart';
import 'package:intl/date_symbol_data_local.dart';
import 'package:calendar_scheduler/database/drift_database.dart';
import 'package:get_it/get_it.dart';
import 'package:calendar_scheduler/provider/schedule_provider.dart';
import 'package:calendar_scheduler/repository/schedule_repository.dart';
import 'package:provider/provider.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await initializeDateFormatting();

  final scheduleRepository = ScheduleRepository();
  final authRepository = AuthRepository();
  final scheduleProvider = ScheduleProvider(
    scheduleRepository: scheduleRepository,
    authRepository: authRepository,
  );

  runApp(
    ChangeNotifierProvider(
      create: (_) => scheduleProvider,
      child: MaterialApp(
        debugShowCheckedModeBanner: false,
        home: AuthScreen(),
      ),
    ),
  );
}

// lib/const/colors.dart
import 'package:flutter/material.dart';

const PRIMARY_COLOR = Color(0xFF0DB2B2);
final LIGHT_GREY_COLOR = Colors.grey[200]!;
final DARK_GREY_COLOR = Colors.grey[600]!;
final TEXT_FIELD_FILL_COLOR = Colors.grey[300]!;
const SECONDARY_COLOR = Color(0xFF335CB0);
const ERROR_COLOR = Colors.red;

// lib/component/login_text_field.dart
import 'package:calendar_scheduler/const/colors.dart';
import 'package:flutter/material.dart';

class LoginTextField extends StatelessWidget {
  final FormFieldSetter<String?> onSaved;
  final FormFieldValidator<String?> validator;
  final String? hintText;
  final bool obscureText;

  const LoginTextField({
    required this.onSaved,
    required this.validator,
    this.obscureText = false,
    this.hintText,
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return TextFormField(
      onSaved: onSaved,
      validator: validator,
      cursorColor: SECONDARY_COLOR,
      // true일 경우 텍스트 필드에 입력된 값이 보이지 않도록 설정
      // 비밀번호 텍스트필드를 만들때 유용
      obscureText: obscureText,
      decoration: InputDecoration(
        // 텍스트 필드에 아무것도 입력 안했을때 보여줄 수 있는 힌트 문자
        hintText: hintText,
        // 활성화된 상태의 보더
        enabledBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(8.0),
          borderSide: BorderSide(
            color: TEXT_FIELD_FILL_COLOR,
          ),
        ),
        // 포커스된 상태의 보더
        focusedBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(8.0),
          borderSide: BorderSide(
            color: SECONDARY_COLOR,
          ),
        ),
        // 에러 상태의 보더
        errorBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(8.0),
          borderSide: BorderSide(
            color: ERROR_COLOR,
          ),
        ),
        // 포커스된 상태에서 에러가 났을때 보더
        focusedErrorBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(8.0),
          borderSide: BorderSide(
            color: ERROR_COLOR,
          ),
        ),
      ),
    );
  }
}

// lib/repository/auth_repository.dart
import 'dart:convert';
import 'dart:io';

import 'package:dio/dio.dart';

class AuthRepository {
  final _dio = Dio();
  final _targetUrl = 'http://${Platform.isAndroid ? '10.0.2.2' : 'localhost'}:3000/auth';

  Future<({String refreshToken, String accessToken})> register({
    required String email,
    required String password,
  }) async {
    final result = await _dio.post(
      '$_targetUrl/register/email',
      data: {
        'email': email,
        'password': password,
      },
    );

    return (refreshToken: result.data['refreshToken'] as String, accessToken: result.data['accessToken'] as String);
  }

  Future<({String refreshToken, String accessToken})> login({
    required String email,
    required String password,
  }) async {
    final emailAndPassword = '$email:$password';
    Codec<String, String> stringToBase64 = utf8.fuse(base64);

    final encoded = stringToBase64.encode(emailAndPassword);

    final result = await _dio.post(
      '$_targetUrl/login/email',
      options: Options(
        headers: {
          'authorization': 'Basic $encoded',
        },
      )
    );

    return (refreshToken: result.data['refreshToken'] as String, accessToken: result.data['accessToken'] as String);
  }

  Future<String> rotateRefreshToken({
    required String refreshToken,
  }) async {
    // Refresh Token을 Header에 담아서 Refresh Token 재발급 URL에 요청을 보냅니다.
    final result = await _dio.post(
      '$_targetUrl/token/refresh',
        options: Options(
          headers: {
            'authorization': 'Bearer $refreshToken',
          },
        )
    );

    return result.data['refreshToken'] as String;
  }

  Future<String> rotateAccessToken({
    required String refreshToken,
  }) async {
    // Refresh Token을 Header에 담아서 Access Token 재발급 URL에 요청을 보냅니다.
    final result = await _dio.post(
      '$_targetUrl/token/access',
        options: Options(
          headers: {
            'authorization': 'Bearer $refreshToken',
          },
        )
    );

    return result.data['accessToken'] as String;
  }
}

// lib/provider/schedule_provider.dart
import 'package:calendar_scheduler/model/schedule_model.dart';
import 'package:calendar_scheduler/repository/auth_repository.dart';
import 'package:calendar_scheduler/repository/schedule_repository.dart';
import 'package:flutter/material.dart';
import 'package:uuid/uuid.dart';

class ScheduleProvider extends ChangeNotifier {
  final ScheduleRepository scheduleRepository;
  final AuthRepository authRepository;

  String? accessToken;
  String? refreshToken;

  DateTime selectedDate = DateTime.utc(
    DateTime.now().year,
    DateTime.now().month,
    DateTime.now().day,
  );
  Map<DateTime, List<ScheduleModel>> cache = {};

  ScheduleProvider({
    required this.scheduleRepository,
    required this.authRepository,
  }) : super() {}

  updateTokens({
    String? refreshToken,
    String? accessToken,
  }) {
    if (refreshToken != null) {
      this.refreshToken = refreshToken;
    }

    if (accessToken != null) {
      this.accessToken = accessToken;
    }

    notifyListeners();
  }

  Future<void> login({
    required String email,
    required String password,
  }) async {
    final resp = await authRepository.login(
      email: email,
      password: password,
    );

    updateTokens(
      refreshToken: resp.refreshToken,
      accessToken: resp.accessToken,
    );
  }

  logout(){
    accessToken = null;
    refreshToken = null;
    cache = {};
    notifyListeners();
  }

  Future<void> register({
    required String email,
    required String password,
  }) async {
    final resp = await authRepository.register(
      email: email,
      password: password,
    );

    updateTokens(
      refreshToken: resp.refreshToken,
      accessToken: resp.accessToken,
    );
  }

  rotateToken({
    required String refreshToken,
    required bool isRefreshToken,
  }) async {
    if (isRefreshToken) {
      final token = await authRepository.rotateRefreshToken(refreshToken: refreshToken);

      this.refreshToken = token;
    } else {
      final token = await authRepository.rotateAccessToken(refreshToken: refreshToken);

      accessToken = token;
    }

    notifyListeners();
  }

  void getSchedules({
    required DateTime date,
  }) async {
    final resp = await scheduleRepository.getSchedules(
      date: date,
      accessToken: accessToken!,
    );

    print(resp);

    cache.update(date, (value) => resp, ifAbsent: () => resp);

    notifyListeners();
  }

  void createSchedule({
    required ScheduleModel schedule,
  }) async {
    final targetDate = schedule.date;
    final uuid = Uuid();

    final tempId = uuid.v4();
    final newSchedule = schedule.copyWith(
      id: tempId,
    );

    cache.update(
      targetDate,
      (value) => [
        ...value,
        newSchedule,
      ]..sort(
          (a, b) => a.startTime.compareTo(
            b.startTime,
          ),
        ),
      ifAbsent: () => [newSchedule],
    );

    notifyListeners();

    try {
      final savedSchedule = await scheduleRepository.createSchedule(
        schedule: schedule,
        accessToken: accessToken!,
      );

      cache.update(
        targetDate,
        (value) => value
            .map((e) => e.id == tempId
                ? e.copyWith(
                    id: savedSchedule,
                  )
                : e)
            .toList(),
      );
    } catch (e) {
      cache.update(
        targetDate,
        (value) => value.where((e) => e.id != tempId).toList(),
      );
    }

    notifyListeners();
  }

  void deleteSchedule({
    required DateTime date,
    required String id,
  }) async {
    final targetSchedule = cache[date]!.firstWhere(
      (e) => e.id == id,
    );

    cache.update(
      date,
      (value) => value.where((e) => e.id != id).toList(),
      ifAbsent: () => [],
    );

    notifyListeners();

    try {
      await scheduleRepository.deleteSchedule(
        id: id,
        accessToken: accessToken!,
      );
    } catch (e) {
      cache.update(
        date,
        (value) => [...value, targetSchedule]..sort(
            (a, b) => a.startTime.compareTo(
              b.startTime,
            ),
          ),
      );
    }

    notifyListeners();
  }

  void changeSelectedDate({
    required DateTime date,
  }) {
    selectedDate = date;
    notifyListeners();
  }

}

// lib/component/today_banner.dart
import 'package:calendar_scheduler/const/colors.dart';
import 'package:calendar_scheduler/provider/schedule_provider.dart';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class TodayBanner extends StatelessWidget {
  final DateTime selectedDate;  // ➊ 선택된 날짜
  final int count;  // ➋ 일정 개수

  const TodayBanner({
    required this.selectedDate,
    required this.count,
    Key? key,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    final provider = context.watch<ScheduleProvider>();

    final textStyle = TextStyle(  // 기본으로 사용할 글꼴
      fontWeight: FontWeight.w600,
      color: Colors.white,
    );

    return Container(
      color: PRIMARY_COLOR,
      child: Padding(
        padding: EdgeInsets.symmetric(horizontal: 16.0, vertical: 8.0),
        child: Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            Text(
              '${selectedDate.year}년 ${selectedDate.month}월 ${selectedDate.day}일',  // “년 월 일” 형태로 표시
              style: textStyle,
            ),
            Row(
              children: [
                Text(
                  '$count개',  // 일정 개수 표시
                  style: textStyle,
                ),
                const SizedBox(width: 8.0,),
                GestureDetector(
                  onTap: (){
                    provider.logout();

                    Navigator.of(context).pop();
                  },
                  child: Icon(
                    Icons.logout,
                    color: Colors.white,
                    size: 16.0,
                  ),
                ),
              ],
            ),
          ],
        ),
      ),
    );
  }
}

// lib/repository/schedule_repository.dart
import 'dart:async';
import 'dart:io';

import 'package:calendar_scheduler/model/schedule_model.dart';
import 'package:dio/dio.dart';

class ScheduleRepository {
  final _dio = Dio();
  final _targetUrl = 'http://${Platform.isAndroid ? '10.0.2.2' : 'localhost'}:3000/schedule';

  Future<List<ScheduleModel>> getSchedules({
    required String accessToken,
    required DateTime date,
  }) async {
    final resp = await _dio.get(
      _targetUrl,
      queryParameters: {
        'date': '${date.year}${date.month.toString().padLeft(2, '0')}${date.day.toString().padLeft(2, '0')}',
      },
      options: Options(
        headers: {
          'authorization': 'Bearer $accessToken',
        },
      ),
    );

    return resp.data
        .map<ScheduleModel>(
          (x) => ScheduleModel.fromJson(
            json: x,
          ),
        )
        .toList();
  }

  Future<String> createSchedule({
    required String accessToken,
    required ScheduleModel schedule,
  }) async {
    final json = schedule.toJson();

    final resp = await _dio.post(
      _targetUrl,
      data: json,
      options: Options(
        headers: {
          'authorization': 'Bearer $accessToken',
        },
      ),
    );

    return resp.data?['id'];
  }

  Future<String> deleteSchedule({
    required String accessToken,
    required String id,
  }) async {
    final resp = await _dio.delete(
      _targetUrl,
      data: {
        'id': id,
      },
      options: Options(
        headers: {
          'authorization': 'Bearer $accessToken',
        },
      ),
    );

    return resp.data?['id'];
  }
}
```

### 참고

이 글은 골든래빗 《Must Have 코드팩토리의 플러터 프로그래밍 2판》의 스터디 내용 입니다.

### 스터디

Q. 

A. 
