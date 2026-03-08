# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Jekyll 기반 개인 블로그. [Chirpy 테마](https://github.com/cotes2020/jekyll-theme-chirpy) (v7.4.1) 사용. GitHub Pages에 자동 배포됨. 언어 설정은 한국어(ko-KR), 주제는 iOS 개발 학습 기록.

## Commands

```bash
# 의존성 설치
bundle

# 로컬 개발 서버 실행 (http://localhost:4000)
bundle exec jekyll s

# 프로덕션 빌드
bundle exec jekyll b -d "_site"

# SCSS 린트
npm test
# 또는
npx stylelint _sass/**/*.scss

# SCSS 린트 자동 수정
npm run fixlint
```

## Architecture

### 포스트 작성 규칙
- 파일 위치: `_posts/YYYY-MM-DD-title.md`
- 필수 front matter:
  ```yaml
  ---
  title: 제목
  date: YYYY-MM-DD HH:MM:SS +0900
  categories: [카테고리]
  tags: [태그1, 태그2]
  ---
  ```
- 선택 front matter: `pin: true`, `render_with_liquid: false`, `toc: false`, `comments: false`

### 디렉터리 구조
- `_posts/` — 블로그 포스트 (Markdown)
- `_drafts/` — 미발행 초안 (comments 자동 비활성화)
- `_layouts/` — HTML 레이아웃 템플릿 (default, post, home, page 등)
- `_includes/` — 재사용 HTML 컴포넌트 (sidebar, topbar, comments, toc 등)
- `_sass/` — SCSS 스타일시트
  - `addon/` — 공통 스타일 및 변수
  - `colors/` — 다크/라이트 테마 색상
  - `layout/` — 페이지별 레이아웃 스타일
  - `variables-hook.scss` — 테마 변수 커스터마이즈용 파일
- `_javascript/` — 빌드 전 JS 소스 (commons/, utils/)
- `_data/` — YAML 데이터 (locales/, authors.yml, contact.yml 등)
- `assets/` — 정적 파일 (이미지, 컴파일된 JS)

### 주요 설정 파일
- `_config.yml` — Jekyll 설정 (사이트 정보, Google Analytics, Giscus 댓글, PWA 등)
- `_sass/variables-hook.scss` — CSS 변수 오버라이드로 테마 커스터마이즈
- `_data/locales/ko-KR.yml` — 한국어 UI 텍스트

### 배포
`main` 브랜치 push 시 GitHub Actions (`.github/workflows/pages-deploy.yml`)가 자동으로 빌드하여 GitHub Pages에 배포.
