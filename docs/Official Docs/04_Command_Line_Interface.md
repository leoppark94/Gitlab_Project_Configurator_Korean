---
title: Command Line Interface
author: Leo
date: 2024-11-01
layout: default
---

이것은 명령줄 인터페이스 도움말입니다. CLI는 로컬 테스트와 개발 중에 유용할 수 있지만, Gitlab-Project-Configurator의 주요 사용처는 Gitlab CI 파이프라인입니다.

GPC는 실행을 구성하기 위해 많은 매개변수를 받을 수 있습니다. 이는 직접적인 명령줄 매개변수(`--something`), 환경 변수(`GPC_SOMETHING`) 또는 `gpc.cfg` 파일의 '기본' 설정(`something = somevalue`)으로 설정할 수 있습니다.

추가적인 정보는 [공식 문서](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/cli.html)를 확인해주세요.

'기본값' 파일은 `--defaults-file`로 구성할 수 있으며 기본값은 `gpc.cfg`입니다.

기본값 파일은 아래와 같습니다.

```yml
# 색상 출력을 설정합니다. 허용되는 값:
#   no: 색상 사용 안 함
#   auto: CI가 TTY가 있는 컬러 터미널인지 확인
#   force: TTY가 없어도 색상 강제 사용
color = "force"

# Python-Gitlab 설정 파일
gitlab_cfg = "~/.gitlab.cfg"

# JSON 보고서 파일의 기본 이름
report_file = "report-gpc.json"
# HTML 보고서 파일의 기본 이름
report_html = "report-gpc.html"

# 기본 Gitlab 서버 URL
gitlab_url = "https://gitlab.com"

# Sentry DSN (데이터 소스 이름)
sentry_dsn = ""

# 자세한 로그를 위한 GELF 엔드포인트 구성
gelf_host = ""
# GELF 출력의 로그 레벨 설정
gelf_level = "DEBUG"

# 메일 알림을 보내기 위한 smtp 서버
smtp_server = ""
# smtp 포트
smtp_port = "25"

# 'GPC에 의해 구성됨' 배지 URL 설정
configured_by_gpc_badge_url = "https://img.shields.io/badge/Configured%20by-GPC-green.svg"

# 프로젝트에 유지할 외부 배지 이미지 URL 목록
accepted_external_badge_image_urls = [""]
```
