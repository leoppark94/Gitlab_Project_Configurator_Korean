---
title: 01. Qucik Guide
author: Leo
date: 2024-10-29
layout: default
parent: OfficialDocs
---

다움 문서는 [document](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/readme.html) 를 기반으로 만들어졌습니다.
최신 내용을 반영하기 위해 정확한 내용은 위 링크를 참고해주세요.

## 특징

- 하나 또는 여러 구성 파일에 저장된 여러 프로젝트를 구성합니다.
- 설정은 여러개의 파일로 나눌 수 있습니다.
- 설정 프로필을 사용하여 설정을 리팩토링할 수 있습니다.

일반적으로 프로젝트는 모든 parameter를 정의하여 개별적으로 구성되지만, GPC는 더 나은 방법을 제공합니다.  
프로젝트의 구성을 프로필로 분리할 수 있으며, 다른 프로필로부터 상속받을 수도 있습니다.  
또한, 프로젝트 수준에서 customizing이 가능합니다.

설정 예시:

- 프로젝트 `My App Rules` 프로필은 모든 프로젝트에 대해 정의된 규칙인 `Our Project Rules` 프로필을 따릅니다.
- 모든 프로젝트에서 "Fast-Forward Merge Commit"과 같은 일부 설정을 통합하고 싶기 때문에, `Our Project Rules` 프로필은 `Our Company Rules` 프로필을 상속받습니다.
- 하지만 `My App Rules`은 변수를 추가해야 하는 등의 특정 설정이 필요함으로, 커스터마이징을 하게 됩니다.

![RULES1](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/_images/gpc-usecase.png)

또한 GPC는 변수를 Secret 으로 저장하고 이러한 변수가 필요한 프로젝트에만 배포하는 데 사용할 수 있습니다.  

![RULES2](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/_images/gpc-workflow.png)

보다 더 나은 이미지를 보고싶으시면 [참조문서](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/readme.html) 링크를 접속하여 보시기 바랍니다.  

## 미리보기 모드 (Preview mode)
GPC에서는 Preview 모드를 제공하여 잠재적으로 호환성을 깨뜨릴 수 있는 변경 사항을 미리 체험할 수 있습니다. 이 모드는 `--preview` 파라미터를 정의하거나 `GPC_PREVIEW` 환경 변수를 설정하여 활성화할 수 있습니다.

### Preview 모드에서의 변경 사항

Preview 모드에서는 다음과 같은 변경 사항이 적용됩니다:

1. **상속 중 리스트 확장**:
    - 기본 규칙(base rule)이 리스트 형태로 변수를 정의한 경우, 상속받는 규칙(inherited rule)은 기본 규칙의 변수 리스트를 그대로 이어받으면서 추가로 자신이 정의한 변수도 포함하게 됩니다.
    - 예를 들어, 기본 규칙이 변수 리스트를 `[variables, protected_branches,…]`로 정의하고, 상속받는 규칙이 `[protected_tags]`를 추가로 정의하면, 상속받는 규칙은 `[variables, protected_branches,protected_tags,…]`의 리스트를 가지게 됩니다.

2. **리스트 항목 제거 불가**:
    - 이 모드에서는 상속받은 규칙에서 리스트 항목을 제거할 수 있는 방법이 없습니다.
    - 예를 들어, 기본 규칙에서 `[variables, protected_branches,…]`를 정의했을 때, 상속받는 규칙에서 `[variables]`를 제거하고 싶어도 제거할 수 없습니다.
    - 만약 리스트 항목을 제거해야 한다면, 규칙 상속 계층 구조를 다시 설계해야 합니다.

### 사용법

새로운 프로젝트를 만들고 (예시 경로 path/to/a/group/project-config) 다음 파일을 생성합니다.

해당 파일을 만든 이후에 추가로 해야 하는 작업은 아래에 작성되어 있습니다.

- workflow를 정의한 `.gitlab-ci.yml`

```yaml
stages:
  - test
  - apply

image: registry.gitlab.com/grouperenault/gitlab-project-configurator:latest

variables:
  # GPC_GITLAB_TOKEN 은 GPC가 GitLab API와 상호 작용할 때 사용하는 토큰으로 "maintainer" 접근 권한을 가진 계정의 토큰이어야 합니다.
  # 프로젝트 설정에서 --gitlab-token에 해당하는 값을 정의해야 합니다.
  GPC_COLOR: force  # '--color force' 콘솔의 출력에 색상을 줍니다.
  GPC_CONFIG: my_config.yml  # '--config my_config.yml' 랑 같은 옵션을 지정
  GPC_REPORT_FILE: report-gpc.json # '--report-file report-gpc.json' 와 같은 옵션을 지정
  GPC_REPORT_HTML: report-gpc.html # 'GPC의 HTML 형식의 리포트 파일 경로를 지정 `report-gpc.html` 로 리포트를 생성하도록 합니다.'
  GPC_DUMP_MERGED_CONFIG: merged-config-gpc.html # GPC가 병합된 설정을 덤프할 파일 경로 `merged-config-gpc.html`
  GPC_PREVIEW: "1"  # GPC의 Preview 모드를 활성화


dry_run:
  stage: test
  script:
    - gpc --diff # 변경사항을 확인합니다
  artifacts:
    paths: # 파이프 라인 실행 도중 생성된 파일을 저장하도록 함
      - $GPC_REPORT_FILE
      - $GPC_REPORT_HTML
    expire_in: 30 days

# 더 많은 옵션으로 실행하도록 함
dry_run:verbose:
  stage: test
  script:
    - gpc --verbose
  artifacts:
    paths:
      - $GPC_REPORT_FILE
      - $GPC_REPORT_HTML
    expire_in: 30 days


apply:
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

- 일반적으로 적용할 정책을 `common_policies.yml` 에 지정해 줍니다.
GitLab CI/CD 파이프라인에서 사용할 내용들이 담겨있습니다.

```yaml
# 변수 설정
variable_profiles: 
  # 보호된 변수들을 생성
  SOME_PROTECTED:
    - name: SOME_PROTECTED_VALUE
      value: some_value
      protected: true
    - name: SOME_PROTECTED_PASSWORD
      value_from_envvar: SOME_PROTECTED_PASSWORD
      protected: true
      masked: true # 마스킹하여 출력되지 않도록 함

# Gitlab의 멤버를 설정
member_profiles:
  - name: mytest_master_approvers
    role: maintainers
    members:
      - roger.duvel
      - path/to/a/subgroup # maintainer 역할을 할 subgroup 경로 지정
  - name: mytest_profiles
    role: developers
    members:
      - roger.duvel
      - ricardo.corona

projects_rules:
  ##############################################################################
  ## DEFAULT POLICY FOR ALL PROJECTS
  ##############################################################################
  - rule_name: default_policy

    default_branch: master
    force_create_default_branch: true # 기본 브랜치를 생성하지 않은 경우 기본 브랜치를 강제로 생성하고, 빈 리포지토리인 경우 초기 커밋을 생성

    description: the project description
    project_members:
      profiles:
        - mytest_master_approvers
        - mytest_profiles
      members:
        - role: developers
          name: toto.desperados
        - role: developers
          name: roger.duvel

    keep_existing_schedulers: true
    schedulers:
        - name: schedule_toto
          branch: master
          cron: "0 4 1 * *" # 크론 표현식으로 스케줄을 정의, 매월 1일 오전 4시에 실행
          enabled: true
          tz: UTC
          variables:
              - import: ANY_VARIABLE
              - name: OTHER_VARIABLE
                value: other value
              - name: var_2
                value_from_envvar: ENVVAR_VALUE

    approval_rules:
    # Merge Request 에 대한 승인 규칙과 설정
        - name: rule1
          profiles:
            - mytest_master_approvers
          members:
            - roger.duvel
          minimum: 2
          # 보호된 branch id 나 name
          protected_branches:
            - some_branch
        - name: rule2
          profiles:
            - mytest_master_approvers
          members:
            - regis.duval
          minimum: 1
          protected_branches:
            - another_branch

    approval_settings:
    # 해당 옵션을 사용하면 "approvers" 의 "options" 을 대체하는 연관이 있음. 두개 옵션을 동시에 사용하는것 X
      can_override_approvals_per_merge_request: true
      remove_all_approvals_when_new_commits_are_pushed: false
      enable_self_approval: false
      enable_committers_approvers: false

    approvers:
    # 하나의 고유한 승인 규칙만 필요할 때 사용
          profiles:
            - mytest_master_approvers
          members:
            - roger.duvel
          minimum: 2
          options:
            can_override_approvals_per_merge_request: true
            remove_all_approvals_when_new_commits_are_pushed: false

    keep_existing_protected_branches: false
    protected_branches:
      - pattern: master
        allowed_to_merge:
            role: maintainers
            profiles:
                - mytest_profiles
            members:
                - robert.tripel
                - jean.kwak
        allowed_to_push: maintainers
        allow_force_push: false
        allowed_to_unprotect: maintainers # allowed_to_unprotect 필드를 명시하지 않을 경우 기본값으로 maintainers로 설정 developers와 admins 도 권한 획득 가능.
        code_owner_approval_required: true

    keep_existing_protected_tags: false
    protected_tags:
      - pattern: master
        allowed_to_create: maintainers
      - pattern: protected_tag
        allowed_to_create:
          role: maintainers
          users:
            - robert.tripel
            - jean.kwak

    permissions:
      visibility: private # 공개 설정
      request_access_enabled: true
      wiki_access_level: disabled # 위키
      issues_access_level: disabled # 이슈
      snippets_access_level: disabled # 스니펫
      lfs_enabled: false # LFS 설정

    # 머지 설정
    mergerequests:
      only_allow_merge_if_all_discussions_are_resolved: true # 모든 discussions가 해결되야 함
      only_allow_merge_if_pipeline_succeeds: true # 파이프라인이 성공해야만 함
      resolve_outdated_diff_discussions: false # 자동 diff 해결
      printing_merge_request_link_enabled: true
      merge_method: ff # fast-forward
      squash_option: do not allow # 병합 시 commit squash 금지
      merge_pipelines_enabled: false # 병합 파이프라인 false(프리미엄만 필요)
      merge_trains_enabled: false # Merge train 사용 여부(프리미엄 필요)

    keep_existing_variables: false
    variables:
      # 변수 프로파일을 가져오도록 합니다.
      - import: ANY_VARIABLE
      - import: SOME_PROTECTED
      # 커스텀 변수는 여기서 가져오고 설정할 수 있습니다.
      - name: ANOTHER_VARIABLE_NAME
        value: another value
      - name: A_HIDDEN_VARIABLE
        value_from_envvar: SOME_SECRET_NAME   # 프로젝트 설정에서 설정된 secret 변수
        masked: true
        protected: true

    runners:
      - runner_id: 666
        enabled: true

    integrations:
      jira:
        url: 'https://jira.server'
        jira_issue_transition_id: 101,102
        username: my bot
        disabled: false
        password_from_envvar: JIRA_PASSWORD  # 프로젝트 설정에서 설정된 secret 변수
        trigger_on_commit: true

    push_rules:
        remove: false
        dont_allow_users_to_remove_tags: true
        member_check: true
        prevent_secrets: true
        commit_message: '^(.+)$'
        commit_message_negative: 'truc'
        branch_name_regex: '^(.+)$'
        author_mail_regex: '^(.+)$'
        prohibited_file_name_regex: '^(.+)$'
        max_file_size: 500
        reject_unsigned_commits: false

    nestor:
      enable: true

  - rule_name: default_internal_policy
    inherits_from: default_policy
    permissions:
      visibility: internal
```

- 커스텀 프로필 정책을 `my_config.yml` 에 지정해 줍니다.

```yaml
include: common_policies.yml # common_policies.yml 파일을 포함하여 공통 정책을 가져옵니다.

projects_configuration:

  ##################################################################################################
  # 'Config' project: 기본정책이지만, 비공개로 되어 다른 변수를 덮어쓰지 않도록 합니다.
  ##################################################################################################
  - paths:
      - path/to/a/group/project-config
    custom_rules:
      variables: null  # overide X
      permissions:
        visibility: private
    rule_name: default_policy

  ##################################################################################################
  # 메인 프로젝트에 대한 기본 정책
  ##################################################################################################
  - paths:
      - path/to/a/project
      - path/to/another/project
      - path/to/a/group
    excludes:
      - path/to/a/group/excluded/project
      - ^path\/to\/exclude\/with\/a\/.*regex$
    rule_name: default_policy
    not_seen_yet_only: true  # 이전에 선언한 경로는 기본 정책에서 제외되도록 합니다.
    recursive: true
    custom_rules:
      variables:
        - import: SOME_PROTECTED
```

다음 환경변수들을 설정해주어야 합니다.

- `GPC_GITLAB_TOKEN`(not protected): Gitlab API 를 사용하는데 사용됩니다(maintainer 이상의 권한이 필요합니다)
- `value_from_envvar` 를 사용하는 모든 것들: `MY_PROTECTED_VARIABLE` 와 그외 변수들
- `master` 브랜치에 스케쥴을 매 3시간 마다 설정: `30 */3 * * *`

TIP: Config 프로젝트 정책이 "private" 인지 확인하세요. 프로젝트를 "internal" 또는 "public" 상태로 유지하면 토큰이 유출될 수 있습니다.

### 실행해보기

clone repository:  

```shell
git clone https://gitlab.com/grouperenault/gitlab-project-configurator.git
```

install:  

```shell
make dev
```

정적 분석, 코드 및 단위 테스트를 실행

```shell
make style check tests  # or make sct
```

의존성 업데이트  

```shell
make update
```
