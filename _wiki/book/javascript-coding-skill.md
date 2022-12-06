---
layout  : wiki
title   : 자바스크립트 코딩의 기술
date    : 2022-12-06 21:35:00 +0900
updated : 2022-12-06 21:35:00 +0900
tag     : 
toc     : true
public  : true
parent  : book
latex   : false
---
* TOC
{:toc}

## 자바스크립트 코딩의 기술

자바스크립트 코딩의 기술 책읽기 후기입니다.

```js
// const taxRate = 0.1;
// const shipping = 5.00;
// let total = 100 + (100 * taxRate) + shipping;
// total = 200;

// const shipping2 = [];
// // 메모리
// // 기초자료형: 값이 변하지 않는 메모리 영역 => 숫자, 문자
// // 참조자료형: 다른 메모리 주소를 참조하는 영역 => 배열, 객체

// // []
// shipping2.push('일등');
// // ['일등']
// return `구매 금액은 ${total}입니다`;

// Tip2
// 모르는 단어, 문장
// 어휘적 유효 범위
// 블록 유효 범위

// Tip3
// 갑자기 난이도가 올라감
// 모르는 단어, 문장
// 호이스팅
// 클로저
// REPL
// 고차 함수
// 즉시 실행 함수

// Tip4
// 템플릿 리터럴

// function getProvider() {
//   // do something
// }
// const image = 'pic';
// const widthInt = 1024;

// 'https://' + getProvider() + '/' + image + '?width=' + widthInt;
// // VS
// `https://${getProvider()}/${image}?width=${widthInt}`;

// Tip5
// 또 난이도가 급격히 올라감
// 모르는 단어, 문장
// 컬렉션
// 맵, 세트, 위크맵, 위크셋
// 이터러블
// 키-값 저장소

// Tip6
// includes
// 어떻게 사용하나?
// const sections = ['contact', 'shipping'];

// const lessions = ['eng', 'jpn', 'cha', 'ger']
// const filters = ['eng', 'jpn'];

// const result = lessions.filter(x => filters.includes(x));

// console.log(result);

// Tip7
// 모르는 단어, 문장
// 조작(mutation), 부수 효과(side effect)
// 주의사항: 펼침연산자는 깊은 복사가 아닙니다.
```
