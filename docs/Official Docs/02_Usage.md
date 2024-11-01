---
title: Usage
author: Leo
date: 2024-10-29
layout: default
---

다음 문서는 [document](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/usage.html) 를 기반으로 만들어졌습니다.  

사실상 번역본이라고 보면 됩니다.

이 내용은 참고만 하고 정확한 최신 내용을 반영하기 위해 정확한 내용은 위 링크를 참고해주세요.

## 개요

GPC는 사용하기 꽤 쉽습니다(적어도 문서에서는).

YAML(또는 JSON) 파일에 정의된 규칙을 적용하기 위해 스케줄러를 구성할 비공개 GitLab 프로젝트를 생성하기만 하면 됩니다.

프로젝트를 “비공개”로 유지하는 한, 아무도 credential 이나 secret 을 볼 수 없습니다.

![GPC_Config](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/_images/gpc-workflow1.png)  

GPC를 이용해 구성한 프로젝트는 다음과 같은 Badge가 생깁니다.

![GPC_BADGE](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/_images/gpc-badge.png)

GPC에는 두 가지 실행 모드가 있습니다:

- 드라이 런(dry-run): 이 모드는 프로젝트의 병합 요청에서 실행됩니다. GPC는 프로젝트를 변경하지 않고 가능한 한 많은 테스트를 수행합니다. 예를 들어, 이는 변수가 규정된 정규 표현식을 준수하지 않는 경우 병합하기 전에 실패하도록 합니다.
- 적용(apply): 이것은 주요 실행 모드로, master 에서 실행 할 수 있습니다. 이 모드는 가능한 한 많은 변경 사항을 적용하려고 하며, 실행 후 모든 변경 사항과 실패를 보고합니다.

## 시작하기전에

1. 설정을 하기 위한 private 프로젝트를 생성합니다. 예를들어 "myteamname-projects-config".  
이 project를 가능하면 private으로 유지하세요, 왜냐하면 이 프로젝트는 다른 프로젝트에 사용되는 credential을 가지고 있기 때문입니다.
    - Gitlab membership 구조에 주의하세요0
    - 최상위 그룹에 추가된 사람은 개인 프로젝트를 포함해 모든 하위 프로젝트에 자동으로 추가됩니다.
    - 다른 사람의 역할을 내려보낼 수 없습니다. [참조](https://docs.gitlab.com/ee/user/permissions.html)
    - 사용자가 그룹의 프로젝트와 프로젝트 자체에 모두 속해 있는 경우 가장 높은 권한이 사용됩니다.

2. 관리하려는 프로젝트에 대한 "소유자" 권한이 있는 Gitlab 토큰이 필요합니다.
3. 이메일 주소가 연결은 필수입니다. 이메일 주소가 없으면 Gitlab이 연결되지 않을 수 있습니다.
4. IPN 에 대한 Gitlab 액세스 토큰을 `GPC_GITLAB_TOKEN` 이라는 프로젝트 변수에 설정합니다.
    ![GITLAB_TOKEN](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/_images/gpc-token.png)

    요구되는 권한은 다음과 같습니다.

    | Feature                                | Minimum Permission |
    |----------------------------------------|--------------------|
    | General settings, merge requests       | Maintainer         |
    | Protected Branches and tags permission | Owner              |

    토큰에 있어야 하는 권한은 사용하려는 기능에 따라 다릅니다.  

    기본적으로 모든 프로젝트의 owner 토큰을 사용하는 것이 좋습니다.  

    그러나 기존 IPN의 토큰을 재사용하려는 경우 멤버십 수준은 GPC로 구성하려는 기능에 따라 다릅니다.  

5. 다음 섹션에 있는 방법으로 configure파일을 만듭니다.
6. 정상적으로 동작하면 Gitlab 스케쥴러를 만듭니다(하루에 두번이면 충분합니다)

## 권장하는 CI integration 방법

구성 파일에 대해 GPC를 실행할 .gitlab-ci.yml 파일을 생성해야 합니다.

```yaml
stages:
  - test
  - apply

image: registry.gitlab.com/grouperenault/gitlab-project-configurator:latest
# 주의: 여기서 "stable"을 사용하여 더 잘 검증된 버전을 사용할 수 있지만, 구버전입니다.  
# Stable은 각 에픽의 마지막에 업데이트됩니다. "latest" (또는 "unstable")이 가장 최신 버전입니다.

variables:
  GPC_GITLAB_URL: https://gitlab.com  # --gitlab-url ... 과 같은 동작
  # GPC_GITLAB_TOKEN 이 환경변수로 있어야 합니다. --gitlab-token ... 과 같은 역할을 합니다.
  GPC_REPORT_FILE: report-gpc.json # '--report-file report-gpc.json' 과 같은 역할을 합니다.
  GPC_REPORT_HTML: report-gpc.html  # '--report-html report-gpc.html' 과 같은 역할을 합니다. (선택사항 이며 html 레포트를 생성하도록 합니다)
  GPC_CONFIG: my_config.yml  # '--config my_config.yml' 과 같은 역할을 합니다.
  GPC_COLOR: force  # '--color force' 와 같습니다.

test:config:
  stage: test
  script:
    - gpc --diff
  artifacts:
    paths:
      - $GPC_REPORT_FILE
      - $GPC_REPORT_HTML
    expire_in: 30 days

apply:config:
  stage: apply
  only:
    - master
    - schedules
    - triggers
  script:
    - gpc --mode apply
  artifacts:
    paths:
      - $GPC_REPORT_FILE
      - $GPC_REPORT_HTML
    expire_in: 30 days
```

위 yaml파일을 통해 생성한 pipeline은 다음 이미지와 같습니다

![PIPELINE_EXAMPLE](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/_images/gpc-pipeline.png)  

- `test:config`: Gitlab server 를 변경하지 않지만, 어떤게 변경되는지 보여줍니다. `--diff` 파라미터를 사용하면 수정되지 않은 건 출력되지 않습니다.  
해당 단계는 병합 전 테스트용도로 사용되며, MR(merge request) 때 사용됩니다.  
- `apply:config` 실제로 Gitlab server 에 변경사항을 적용합니다. 오직 `master` 브랜치에서만 작동하도록 합니다.

 dry run mode의 출력결과는 아래 사진과 같습니다.

 ![DIFF_EXAMPLE](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/_images/gpc-dry-run-output.png)

주의!

- 프로젝트 설정에서 `GPC_GITLAB_TOKEN` 환경 변수를 정의했는지 확인하세요. 이는 구성하려는 모든 프로젝트에 대해 "owner" 역할을 가진 사용자의 GitLab 개인 액세스 토큰입니다. 가능하면 최상위 그룹의 "owner"로 설정하세요.

## Email 알림 설정

GPC프로젝트가 변경사항이 있을 때 알림이 가도록 이메일을 설정할 수 있습니다.
프로젝트에서 환경 변수 `GPC_WATCHERS`를 정의하고 이메일 주소를 세미콜론으로 구분하여(예시: `leoppark94@gmail.com;foo@bar.com`) 설정해야 합니다.  
“watchers” 는 “A change has been made by gpc” 라는 제목의 메일을 받게 됩니다.

![EMAIL-EXAMPLE](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/_images/gpc-email-report.png)

또한 GPC script로 지정할 수도 있습니다.

`$ gpc -m apply -c my_config.yml --watchers leoppark94@gmail.com;foo@bar.com`

## 리포트(Report)

두 종류의 report를 생성할 수 있습니다.  
다음 파라미터를 사용하세요!  
`--report-file report.json` 혹은 `--report-html report.html`  

JSON파일은 다음과 같은 구조를 가지고 있습니다  
![JSON_REPORT_EXAMPLE](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/_images/gpc-json-report.png)

HTML로 출력하면 좀더 보기좋게 출력됩니다.  
![HTML_REPORT_EXAMPLE](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/_images/gpc-html-report.png)

## 2.5. 미리보기 모드

GPC는 설정에서 새로운 동작을 활성화하는 선택 옵션을 제공합니다. 이 옵션은 향후 릴리스에서 표준 옵션이 될 예정이지만, 기존 설정을 깨지 않기 위해 기본적으로 비활성화되어 있습니다.  

미리보기 모드는 환경 변수 `GPC_PREVIEW=1`을 정의하거나 `--preview` parameter를 사용하여 활성화할 수 있습니다.

현재 미리보기 모드는 규칙이 상속될 때 리스트가 병합되는 방식을 변경합니다. 현재 규칙이 다른 규칙을 상속할 때, 딕셔너리는 병합되지만 리스트는 대체됩니다.  

이는 `Gitlab-CI`가 딕셔너리(변수, 태그 등)를 병합하고 리스트(스크립트, 규칙 등)를 대체하는 방식을 모방한 것입니다.

미리보기 모드에서는 이제 리스트의 내용을 대체하는 대신 "확장"합니다. 이를 통해 기본 규칙에서 기본값을 정의하고 모든 상속된 규칙이 자동으로 해당 값을 가져오도록 할 수 있습니다.

이 방식은 "파괴적인 변경"을 초래할 수 있기 때문에 주의가 필요합니다(상속된 규칙은 리스트에서 기존 항목을 제거할 수 없습니다)

- **현재 동작 방식**:
  - 딕셔너리: 병합됨 (첫 번째 규칙의 값이 두 번째 규칙에 추가되거나 덮어씌워짐).
  - 리스트: 대체됨 (첫 번째 규칙의 리스트가 두 번째 규칙의 리스트로 완전히 바뀜).

- **미리보기 모드에서의 동작 방식**:
  - 딕셔너리: 병합됨 (동일).
  - 리스트: 확장됨 (첫 번째 규칙의 리스트 항목들이 두 번째 규칙의 리스트에 추가됨).

미리보기 모드 비활성화 (기본값):

```yaml
projects_rules:
- rule_name: root_policy
  project_members:
    members:
      - some.name

projects_rules:
  - rule_name: derived_policy
    inherits_from: root_policy
    project_members:
      members:
        - some.name  # need to recopy
                    # since lists are
                    # replaced
        - another.person
```

미리보기 모드 활성화(`GPC_PREVIEW=1`):

```yaml
projects_rules:
  - rule_name: root_policy
    project_members:
      members:
        - some.name

projects_rules:
  - rule_name: derived_policy
    inherits_from: root_policy
    project_members:
      members:
        - another.person
        # some.name is automatically added
```
