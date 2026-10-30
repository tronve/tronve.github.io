---
title: GitHub Pages와 Chirpy로 블로그 만들기
date: 2026-10-30 10:00:00 +0900
categories: [개발]
tags: [github-pages, jekyll, chirpy, ruby, github-actions]
---

나이츠 프론티어의 첫 미션으로 학습 기록을 남길 블로그를 만들었다. 앞으로 7개월 동안 미션 결과물과 공부한 내용을 여기에 쌓을 예정이라, 만드는 과정과 막혔던 지점을 먼저 정리해 둔다.

## GitHub Pages는 어떻게 동작하나

GitHub Pages는 저장소에 있는 정적 파일(HTML, CSS, JS)을 웹사이트로 호스팅해 주는 기능이다. 저장소 이름을 `<아이디>.github.io`로 만들면 그 주소가 그대로 사이트 주소가 된다.

정적 파일만 올릴 수 있기 때문에 보통 정적 사이트 생성기를 함께 쓴다. 이 블로그는 다음 흐름으로 만들어진다.

1. 마크다운으로 글을 쓴다 (`_posts/`)
2. push하면 GitHub Actions 워크플로가 실행된다
3. Jekyll이 마크다운과 테마를 HTML로 빌드한다
4. 빌드 결과가 GitHub Pages로 배포된다

서버를 직접 운영하지 않아 관리할 것이 거의 없고, 글 하나하나가 git 커밋으로 남는다는 점이 마음에 들었다.

## 왜 Chirpy인가

정적 사이트 생성기로는 Jekyll, Hugo, Astro 등을 비교했고, 그중 Jekyll의 [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) 테마를 골랐다. 미션에서 요구한 카테고리 구성이 기본 기능으로 들어 있고, 태그, 검색, 다크 모드, 목차도 따로 만들 필요가 없다. 보안 블로그에서 많이 쓰여 참고할 자료가 많다는 점도 컸다.

## 구축 과정

### 1. 템플릿으로 저장소 만들기

`cotes2020/chirpy-starter` 저장소에서 "Use this template"로 `tronve.github.io` 저장소를 만들었다. Fork 대신 템플릿을 쓰면 테마는 gem으로 받아 쓰고 내 저장소에는 설정과 글만 남아서 관리가 단순해진다.

무료 요금제에서는 비공개 저장소의 Pages가 동작하지 않아 공개 저장소로 만들었다.

### 2. 배포 방식을 GitHub Actions로 바꾸기

Settings → Pages에서 Source를 "Deploy from a branch"가 아니라 **GitHub Actions**로 바꿔야 한다. Chirpy는 템플릿에 포함된 워크플로로 직접 빌드하고 배포하기 때문에, 이 설정을 빠뜨리면 제대로 된 화면이 나오지 않는다.

### 3. `_config.yml` 설정

`url`, `lang: ko-KR`, `timezone: Asia/Seoul`, 제목과 소개, 프로필 이미지를 설정했다. 이메일은 페이지에 그대로 노출되어 수집 봇의 대상이 될 수 있어 적지 않았다.

> `_config.yml`은 `jekyll serve`가 실행 중이어도 자동으로 반영되지 않는다. 수정 후에는 서버를 껐다가 다시 켜야 한다.
{: .prompt-info }

## 트러블슈팅: Ruby 4.0에서 bundle install 실패

로컬에서 미리 보기 위해 `bundle install`을 실행하자 다음 오류가 났다.

```text
Because jekyll-theme-chirpy >= 7.1.0 depends on Ruby ~> 3.1
  and Gemfile depends on jekyll-theme-chirpy ~> 7.6,
  Ruby ~> 3.1 is required.
So, because current Ruby version is = 4.0.9,
  version solving has failed.
```

처음에는 "3.1 이상이 필요한데 4.0.9가 더 높으니 괜찮지 않나?"라고 생각했다. 원인은 `~>` 연산자였다. RubyGems의 pessimistic version constraint로, `~> 3.1`은 `>= 3.1, < 4.0`을 뜻한다. 마지막 자리의 버전 상승만 허용하고, 그 앞자리가 바뀌는 메이저 업그레이드는 막는다. 메이저 버전이 바뀌면 호환성이 깨질 수 있으니 의존성 쪽에서 미리 선을 그어 둔 것이다.

Ruby 3.4를 따로 설치하고 경로 우선순위를 바꿔 해결했다.

```bash
brew install ruby@3.4
echo 'export PATH="$(brew --prefix ruby@3.4)/bin:$PATH"' >> ~/.zshrc
# 터미널을 다시 연 뒤
ruby -v          # 3.4.x 확인
bundle install
bundle exec jekyll serve
```

`ruby@3.4`처럼 버전이 붙은 Homebrew 패키지는 keg-only로 설치되어 시스템 경로에 자동으로 연결되지 않기 때문에 PATH에 직접 넣어야 한다.

로컬 Ruby 버전은 실제 배포에 영향을 주지 않는다는 점도 알게 됐다. 사이트는 GitHub Actions가 워크플로 파일에 지정된 Ruby 버전으로 따로 빌드하고, 로컬 환경은 미리 보기 용도다.

> 맥에서 만든 `Gemfile.lock`을 커밋한다면 `bundle lock --add-platform x86_64-linux`로 Actions의 리눅스 환경을 플랫폼에 추가해 둬야 한다.
{: .prompt-tip }

## 카테고리 구성

미션에서 정한 6개 카테고리로 구성했다.

- 개발
- CTF/Wargame
- Bug Bounty
- 블로그/기술문서
- 논문/컨퍼런스
- 공모전/자격증

Chirpy는 카테고리를 따로 만드는 대신 글 머리말의 `categories` 값으로 자동 생성한다. 그래서 글이 하나도 없는 카테고리는 화면에 나타나지 않고, 카테고리마다 첫 글을 하나씩 작성했다.

```markdown
---
title: 글 제목
date: 2026-10-30 10:00:00 +0900
categories: [개발]
tags: [github-pages, jekyll]
---
```

> `date`가 현재보다 미래 시각이면 그 글은 빌드에서 빠진다. 시간대(`+0900`)를 꼭 붙여야 한다.
{: .prompt-warning }

## 앞으로

자기소개 페이지와 디자인을 더 다듬고, 미션을 수행할 때마다 과정을 기록할 계획이다.
