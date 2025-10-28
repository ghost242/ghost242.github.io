---
layout: post
title: 타입시스템에 대해서
subtitle: Basic of type system in Programming
parent: Haskell
has_toc: true
comments: true
categories: ["Programming", "Haskell"]
tag: ["Haskell", "Functional programming", "Data type"]
---

## Abstract

프로그래밍 언어에서 타입 시스템은 프로그래머가 1보다 큰 값을 다루기 시작할 때 이미 중요했던 개념이다. 왜냐하면 데이터의 타입은 한정된 메모리와 명령어의 길이로 인해 수학적으로는 무한인 수의 크기에 제약을 두는 것이기 때문이다.

일반적으로 널리 보급된 데스크탑 CPU의 명령어셋인 amd64는 정수를 최대 $2^{64}-1$ 까지 다룰 수 있다. 실수는 Floating point라고 하는 방식으로 다루는데, 이런 특징으로 인해 3.9999999가 4와 같다거나 하는 등의 많은 에러 케이스가 발생하기도 한다.

이 외에도 컴퓨터가 보다 범용적으로 사용법이 확장되면서 숫자를 응용해서 인간의 언어를 다룰 수 있도록 계속 데이터 표현 방식이 확장되었다. 이 포스트에서는 Haskell이 다루는 데이터 타입에 대해 정리해보려 한다.

## Type

Haskell에서 다루는 일반적인 자료구조(Primitive type)으로는 Integer