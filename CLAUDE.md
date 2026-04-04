# 블로그 작업 가이드 (Claude용)

## 스택
- Jekyll + Chirpy 테마 (v7.5), GitHub Pages 커스텀 Actions 워크플로우로 배포
- 포스트 위치: `_posts/`, 파일명 형식: `YYYY-MM-DD-제목-슬러그.md`
- 배포 흐름: `main` 브랜치 push → GitHub Actions 빌드 → `blog.payable.kr` 배포

## 포스트 작성

### 프론트매터
```yaml
---
title: "포스트 제목"
date: YYYY-MM-DD HH:MM:SS +0900
categories: [카테고리1, 카테고리2]
tags: [태그1, 태그2, 태그3]
---
```

### 날짜 규칙
- **반드시 과거 날짜/시간을 사용한다.** Jekyll은 미래 타임스탬프의 포스트를 빌드에서 제외한다.
- `_config.yml`에 `future: true`가 설정되어 있지만, 날짜 자체를 올바르게 설정하는 것이 우선이다.
- 타임존은 `+0900` (KST) 사용. 특별한 이유가 없으면 시간은 `00:00:00`으로 설정.
- 예: 오늘이 2026-04-05이면 → `date: 2026-04-04 00:00:00 +0900`

### 카테고리와 태그
- **소문자**를 사용한다. Chirpy 테마는 링크 생성 시 Jekyll의 `slugify` 필터를 적용해 소문자로 변환하고(`CTF` → `/categories/ctf/`), jekyll-archives는 `:name` 그대로 페이지를 생성한다. Linux 빌드 서버는 대소문자를 구분하므로 대소문자가 다르면 깨진 링크로 처리된다.
- 하이픈이 포함된 태그는 괜찮다: `blind-sql-injection`, `ctf-writeup`

### 포스트 내 예시 URL
- 펜스드 코드 블록(` ``` `)이나 인라인 백틱(`` ` ``) 안의 URL은 `<code>` 태그로 렌더링되어 htmlproofer가 검사하지 않으므로 안전하다.
- 가짜 URL을 본문 일반 텍스트에 그냥 쓰지 않는다. kramdown이 자동 링크로 변환할 수 있다.

## 빌드 및 CI

- htmlproofer v5 (`html-proofer ~> 5.0`)가 Actions의 "Test site" 단계에서 실행된다.
- 현재 설정: `--disable-external`(외부 URL 검사 생략) + `|| true`(htmlproofer 오류가 있어도 빌드 실패하지 않음).
- htmlproofer 오류가 발생하면 GitHub Actions의 "Test site" 단계 로그에서 구체적인 깨진 링크를 확인한다.

## 사이트 구조

- `_tabs/` — 사이드바 탭 페이지. 현재 `about.md`와 `archives.md`만 있음 (categories/tags 탭은 의도적으로 제거됨).
- `_config.yml` 주요 설정: `theme_mode: dark`, `future: true`, `timezone: Asia/Seoul`
- 파비콘: `assets/img/favicons/`에 필요한 파일 모두 존재
