---
title: GPC 한국어판
layout: default
---
## Welcome to Gitlab Project Configurator

GPC는 하나 이상의 Gitlab 프로젝트 설정을 자동으로 구성할 수 있도록 해줍니다.

![GPC_Logo](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/_images/gpc-icon.png)

프로젝트 구성을 버전 관리된 구성으로 정의할 수 있으며, IaC 처럼 gitlab의 설정을 관리할 수 있도록 합니다.(대표적으로 terraform 처럼)

공식 문서를 확인하고 싶으시면 **좌측 사이드바의 OfficialDocs 를 클릭해 주세요**

### GPC의 목적

- 한 번에 많은 Gitlab 프로젝트를 구성할 수 있습니다.
- 프로젝트에 적용하기 전에 변경 사항이 Gitlab에서 지원되는지 테스트 할 수 있습니다.
- 모든 프로젝트의 구성을 유지하도록 보장합니다.
- 일부 구성을 쉽게 합니다 (ex. JIRA 플러그인 또는 Webhook).
- Gitlab의 CI 토큰과 같은 중요한 요소를 갱신/업데이트합니다.
- 모든 프로젝트에 대해 파생을 허용하며, 모든 내용은 YAML로 설명되어 주석과 추적 가능성을 추가할 수 있습니다.
- stdout, HTML 보고서 또는 JSON 파일과 같은 여러 방법으로 실행 결과를 제공합니다.

GPC는 다른 설정에는 손을 대지 않고 변경하고 싶은 일부 설정만 변경합니다.  
이는 "모두가 원하는 대로 하는" 것과 "모든 프로젝트가 이러저러 해야 한다"는 사이의 원활한 전환을 가능하게 합니다.  
