---
layout: post
title: "Typescript에서 객체 순회하기"
description: ""
date: 2021-01-15
tags: [typscript]
comments: true
---

# TL;DR

- 순회하는 객체의 value만 필요한 경우(객체가 추가적인 key를 가질 수 없다고 단언할 수 있다면): Object.entries
- 순회하는 객체의 key와 value가 모두 필요한 경우
    - 객체가 prototype pollution으로부터 안전하고, key를 추가적으로 가질 수 없다고 단언할 수 있다: key의 타입을 좁히자
    - prototype polution으로부터 안전하다고 단언할 수 있다: 제네릭 활용
    - 완전 방어적으로 작성하고 싶다: 타입 가드 활용

# 문제 발생

다음과 같은 코드는 자바스크립트에서는 가능했지만 타입스크립에서는 오류를 발생한다

```tsx
interface Person {
  name: string;
  age: number;
}

function iterate(person: Person) {
  for(const key in person) {
    console.log(person[key]); // 에러
  }
  Object.keys(person).forEach(key => {
    console.log(person[key]); // 에러
  })
}
// 둘 다 다음과 같은 에러 발생
// Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'Person'.
//  No index signature with a parameter of type 'string' was found on type 'Person'
```

- Person 타입에 index signature가 없기 때문에 string type의 `key` 변수로 person에 접근할 수 없기 때문

`for...in obj` 문이나 `Object.keys(obj)`가 obj의 key로 좁혀진 타입의 배열이 아닌 string의 배열을 반환하는 이유는 obj는 타입스크립트 안에서 알 수 없는 더 많은 키를 런타임에서 가질 수 있기 때문이다(derived 클래스가 사용될 경우 등). 자세한 내용은 [다음](https://github.com/microsoft/TypeScript/pull/12253#issuecomment-263132208) 참조.

따라서 객체를 타입스크립트에서 순회하기 위해서는 몇 가지 추가적인 과정을 거쳐야 한다.

# 해결 방법

## key의 타입 좁히기

key의 타입을 좁힘으로서 이런 문제를 해결할 수 있다.

```tsx
function iter(person: Person) {
  let key: keyof Person;
  for(key in person) { // let key = "name" | "age"
    console.log(person[key]) // OK
  }
}
```

근데 이 방법은 두 가지의 문제를 가진다

1. prototype pollution: 객체의 prototype에 오염이 발생할 경우 for...in 문이 의도치 않게 작동할 수 있다. 자세한 내용은 [이 글](https://youngdo212.github.io/2020-06-13/for-in/)을 참고하자.
2. 객체에 추가적인 key가 들어올 경우: 타입스크립트의 Structural Typing으로 인해 발생할 수 있는 문제.

2번 문제의 경우 다음과 같은 상황을 예상해 볼 수 있다.

```tsx
const wrongPerson = {
	name: 'wrong',
	age: 0,
	notExpectedProp: true,
}
iter(wrongPerson); // OK
```

타입스크립트는 그 유래가 어떻게 됐든, 구조가 같다면 같은 타입으로 인식한다. wrongPerson 변수도 Person타입과 마찬가지로 name: string, age: number의 타입을 가지므로 Person 타입으로 인식된다. 이것은 iter 함수 내부에서 value를 사용할 때 문제가 발생할 수 있다.

```tsx
iter(wrongPerson)

function iter(person: Person) {
  let key: keyof Person;
  for(key in person) {
    const value = person[key] // const value: string | number
		if(typeof value === 'number') {
			console.log(value.isNaN());
		} else {
			console.log(value.length)); // 타입스크립트에서는 에러가 없지만 런타입에서 에러 발생
		}
  }
}
```

value는 string 또는 number 타입으로 인식되어 위 코드는 타입스크립트에서 에러가 발생하지 않는다. 하지만 런타임 단계에서는 에러가 발생한다. wrongPeron은 boolean 타입의 notExpectedProp 속성도 가지고 있으므로 `value.length` 코드를 사용할 수 없다.

이를 방지하기 위해서 다른 방식으로 타입을 좁힐 필요가 있다. 두 가지 방법이 존재한다

## Generic 이용하기

제네릭을 이용하면 key도 좁힐 수 있을 뿐더러 value의 잘못된 사용까지 막을 수 있다

```tsx
function bar<T extends Person>(person: T): void {
  for(const key in person) { // const key: Extract<keyof T, string>
    const value = person[key]; // const value: T[Extract<keyof T, string>]
  }
}
```

key가 T의 키 값으로 좁혀졌기 때문에 `person[key]` 를 사용할 수 있다.

또한 value의 타입은 T의 타입을 모르기때문에 함부로 사용할 수 없으므로 반드시 하위 코드에서 value의 타입을 좁히는 로직이 추가될 것이다.

다만 이 방법은 여전히 for...in 문을 사용하고 있기 때문에 prototype pollution으로 부터 안전하지 않다.

## 타입 가드 이용하기: assertion function를 통해

custom assertion function을 이용해도 해결이 가능하다. 이 방법은 prototype pollution으로 부터도 안전하나다.

```tsx
function assertHasOwnProperty<
  T extends Record<PropertyKey, unknown>,
  K extends PropertyKey
>(obj: T, prop: K): asserts obj is T & Record<K, unknown> {
  if (!Object.prototype.hasOwnProperty.call(obj, prop)) {
    throw new Error(`hasOwnProperty error`);
  }
}

function iter(person: Person) {
  Object.keys(person).forEach(key => {
    assertHasOwnProperty(person, key);
    const value = person[key]; // const value: unknown
  })
}
```

작성하는 코드가 길어지기는 하지만, prototype 오염으로 부터 안전하다. 또한 value의 타입은 unknown이기 때문에 타입을 좁히지 않는 한 함부로 value를 사용할 수 없다.

## 순회하는 객체의 value만 필요할 경우

객체의 나열가능한 value값만 필요한 경우 해결법은 단순하다. Object.entries를 이용하면 된다.

```tsx
function iter(person: Person) {
  for(const [, value] of Object.entries(person)) { // for in이 아니라 for of
    console.log(value); // OK: value is any
  }
}
```

Object.entries를 통해 반환되는 배열은 이미 객체의 값을 보장하고 있으므로, key를 이용해 참조할 필요가 없어서 안전하다. 하지만 이 방법은 value가 any 타입을 가지게 되어 자유자재로 조작이 가능하다. 따라서 객체가 추가적인 속성을 가지지 않는다고 단언할 수 있을 때만 사용하는 것을 추천한다.

# 결론

결국 순환하는 객체를 얼만큼 신뢰할 수 있을지에 따라서 사용하는 방법이 달라질 수 있다. 상황에 따라 적절한 방법을 적용하는 것이 중요할 것 같다

참고자료

- [https://fettblog.eu/typescript-better-object-keys/](https://fettblog.eu/typescript-better-object-keys/)
- [https://effectivetypescript.com/2020/05/26/iterate-objects/](https://effectivetypescript.com/2020/05/26/iterate-objects/)
- [https://github.com/microsoft/TypeScript/pull/12253#issuecomment-263132208](https://github.com/microsoft/TypeScript/pull/12253#issuecomment-263132208)
- [https://fettblog.eu/typescript-hasownproperty/](https://fettblog.eu/typescript-hasownproperty/)
- [https://youngdo212.github.io/2020-06-13/for-in/](https://youngdo212.github.io/2020-06-13/for-in/)