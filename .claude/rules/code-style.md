---
paths:
  - "_posts/**/*.md"
---

# Code Style Guide

## 포스트 파일명

```
yyyy-MM-dd-대분류-타이틀.md
```

- 구분자는 하이픈(`-`) 사용 (언더스코어 `_` 사용 금지)
- 예: `2024-03-08-iOS-UIKit-layout-guide.md`

## Front Matter

```yaml
---
title: 제목
author: totoroto
date: yyyy-MM-dd HH:MM:SS +0900
categories: [대분류]
tags: [태그1, 태그2]
render_with_liquid: false
pin: false
---
```

- `date` 타임존은 항상 `+0900` (서울)
- `render_with_liquid: false` — 코드 예제에 liquid 문법과 충돌하는 `{{`, `{%` 등이 포함될 경우 반드시 추가
- `pin: true` — 상단 고정이 필요한 포스트에만 사용

## 본문 구조

- 제목은 `##`부터 시작 (h1은 front matter의 `title`이 대신함)
- 하위 섹션은 `###`, `####` 순으로 계층 구성

## 코드 블록

코드 블록에는 반드시 언어를 명시한다.

````markdown
```swift
let name = "totoroto"
```
````

- 들여쓰기는 **4 spaces** 사용

## 이미지

- 경로: `assets/img/posts/대분류/파일명.png`
- 예: `assets/img/posts/iOS/autolayout-example.png`
- 삽입 시 alt 텍스트 작성

```markdown
![설명](assets/img/posts/iOS/autolayout-example.png)
```
