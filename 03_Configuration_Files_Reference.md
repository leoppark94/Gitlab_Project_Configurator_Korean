---
title: Configuration Files Reference
author: Leo
date: 2024-10-29
layout: post
---

이 문서는 GPC에서 사용할 수 있는 프로젝트 설정 파일을 설명합니다.

이 파일을은 priviate 프로젝트에 주로 보관되며, 직접 사용자가 해당 변경사항을 적용할 수 있더라도, Pipeline을 통해 실행해 주는것이 좋습니다.

예시에서 설명했던 `common_policies.yml` 파일과 `my_config.yml`파일에 사용했던 모든 내용에 대해 다루고 있습니다.

정확한 내용은 [공식문서](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/config_files.html#enable-disable-wiki-lfs-snippets-issues-ci-cd) 를 참고해주세요.

해당값들은 YAML파일 뿐 아니라 JSON이나 TOML로 관리하여도 상관없습니다.

구성은 여러 섹션으로 나뉩니다:

- "rules" 또는 "policies": 규칙의 집합이 정의되고 이름이 지정됩니다. 여러 개의 "정책"을 정의할 수 있으며, 상속을 사용할 수도 있습니다.

- projects configuration: 프로젝트 또는 프로젝트 목록에 대해 정책이 적용되며, 예외가 있을 수 있습니다("custom_rules" 사용).

- groups configuration: 그룹 또는 그룹 목록에 대해 정책이 적용되며, 예외가 있을 수 있습니다("custom_rules" 사용).

"rules" 또는 "policies"은 GitLab 프로젝트에 적용될 설정의 집합입니다. 각 규칙은 고유한 이름을 가져야 합니다.

"project configuration"은 동일한 규칙을 따라야 하는 프로젝트의 집합입니다. "Group Configurations"도 마찬가지입니다.

![CONFIGURE_IMAGE](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/_images/rules-projects-config.png)

---

Tip: `null`은 변경하지 않는다는 의미로 사용됩니다

---

## 전체적인 구성 파일 예시

전체적인 파일 구조를 확인하기 위해서 다음 [공식 demo Repo](https://gitlab.com/grouperenault/gpc_demo/gpc-demo-config/-/tree/master?ref_type=heads) 를 참조해주세요.

`common_policies.yml` 과 `my_config.yml`파일을 참조하면 됩니다.

## 상위 노드

구성은 서로 다른 상위 노드 아래에 여러 부분으로 나뉘어지며, 여러 파일로 나눌 수도 있습니다.

예를들어 `high_node.yml` 

```yml
# Define groups of variables
variable_profiles:
  ...

# Define groups of members
member_profiles:
  ...

# Define group of labels
label_profiles:
  ...

# Define rules for projects
projects_rules:
  ...

# Apply rules on projects
projects_configuration:
  ...

# Define rules for groups
groups_rules:
  ...

# Apply rules for groups
groups_configuration:
  ...
```

같이 되어있는 파일을 여러 파일로 분할하여 필요한 항목을 `include`를 이용하여 불러올 수 잇습니다.

아래 예시처럼 분리할 수 있습니다.

`variables.yml`

```yml
variable_profiles:
  ...
```

`members.yml`

```yml
member_profiles:
  ...
```

`project_rules01.yml`

```yml
projects_rules:
  ...
```

`project_rules02.yml`

```yml
projects_rules:
  ...
```

`project_config01.yml`

```yml
projects_configuration:
  ...
```

`project_config02.yml`

```yml
projects_configuration:
  ...
```

`global_config.yml`

```yml
include:
  - variables.yaml
  - members.yaml
  - project_rules01.yaml
  - project_rules02.yaml
  - project_config01.yaml
  - project_config02.yaml
```

위와 같이 구성하게 되면 `global_config.yaml` 파일은 다른 파일의 모든 내용을 merge 하여 포함하게 됩니다.

## Project Rules

`projects_rules`(혹은 `groups_rules`) 상위 노드에 구성되는 항목들에 대한 설명입니다. 각각의 규칙은 반드시 `rule_name`을 가지고 있어야합니다.

### 강제 기본 브랜치 생성

`force_create_default_branch` 옵션을 사용하면 현재 기본값에서 기본 브랜치를 강제로 생성하거나(브랜치가 존재하지 않는 경우) 저장소가 비어 있는 경우 초기 커밋을 생성할 수 있습니다.

```yml
projects_rules:
  - rule_name: myteam_master_rule
    default_branch: master
    force_create_default_branch: true
```

### Null 처리

GPC에서 사용 가능한 대부분의 규칙에서 기본값은 특별한 의미를 갖는 `null` 로 설정됩니다.

`null` 은 변경을 하지 말라는 의미로 사용됩니다(현재 설정 계속 유지)

만약 현재 project 설정이 적용되어있고, 업데이트하고싶지 않으면

```yml
projects_configuration:
  - paths:
      - groupname/my_project01
    rule_name: default_policy
    custom_rules:
      protected_tags: null
```

위와같이 설정하면 됩니다.

이는 `default_policy` 라는 규칙이 프로젝트에 적용되지만 보호된 태그가 규칙에서 강제 적용되더라도 그대로 유지하는 방법을 보여줍니다.

### Rule 상속

`inherits_from` 키워드를 이용해서 다른 규칙에서 상속해올 수 있습니다.

```yml
projects_rules:
  - rule_name: base_policy
    default_branch: master
    permissions:
      visibility: internal

  - rule_name: extended_policy
    inherits_from: base_policy
    permissions:
      visibility: private
```

위 예시코드에서 `Extended_policy`의 가시성은 비공개로 설정되지만 `default_branch`는 `base_policy`의 가시성을 상속받아 사용합니다.

### 다중 Rule 상속

Rule 은 `inherits_from` 키워드를 사용하여 다른 Rule 에서 상속할 수 있습니다.

```yml
projects_rules:
  - rule_name: first_base_policy
    default_branch: master
    permissions:
      visibility: internal
  - rule_name: scnd_base_policy
    permissions:
      visibility: private
    mergerequests:
      merge_method: ff

  - rule_name: extended_policy
    inherits_from:
      - first_base_policy
      - scnd_base_policy
    push_rules:
      remove: true
```

이 예시 코드 에서 `Extended_policy`는 비공개로 설정되지만(`scnd_base_policy`가 `first_base_policy` Rule 을 재정의하기 때문에) `default_branch`는 `first_base_policy`의 항목을 사용합니다.

이 구성은 `Extended_policy`와 동일합니다.

```yml
projects_rules:
  - rule_name: extended_policy
    default_branch: master
    permissions:
      visibility: private
    mergerequests:
      merge_method: ff
    push_rules:
      remove: true
```

### Schema의 하위 서브셋 실행

구성 스키마의 속성 중 일부만 적용하는 것도 가능합니다.  
`--executor` 옵션을 사용하고 적용하고자 하는 속성(또는 쉼표로 구분된 속성 목록)을 지정해야 합니다.  
예시: (project_members, members, protected_branches, protected_tags, variables, labels, approval_rules, mergerequests, approval_settings, jira, badges, pipelines_email, push_rules, runners, schedulers, deploy_keys).

`gpc --mode apply --executor members`

일부 속성은 종속성을 가지고 있습니다. 예를들어 멤버가 있어야 `approval_rules` 를 적용할 수 있다는 것 같은게 있을 수 있습니다.

### Threadpools 사용하기

다량의 프로젝트를 구성해야할 경우 `--max-workers` 옵션을 사용해서 스레드 풀에 사용할 작업자 수를 정의할 수 있습니다.

`gpc --mode apply --max-workers 8`

### Project 권한 설정

1. Project 가시성(visibility)

```yml
projects_rules:
  - rule_name: my_rule_name
    permissions:
      visibility: internal
```

프로젝트 가시성은 `public`, `internal`, `private`, `null` 을 사용할 수 있습니다.

2. 사용자의 권한 요청 승인 유무

```yml
projects_rules:
  - rule_name: my_rule_name
    permissions:
      request_access_enabled: true
```

3. Wiki, LFS, Snippets, Issues, CI/CD

```yml
projects_rules:
  - rule_name: my_rule_name
    permissions:
      wiki_access_level: disabled
      lfs_enabled: disabled
      issues_access_level: disabled
      snippets_access_level: disabled
      container_registry_access_level: disabled
      builds_access_level: disabled
      merge_requests_access_level: disabled
      packages_enabled: false
```

위 yml파일처럼 구성할 수 있습니다.

위 내용은 프로젝트에서 사용할 수 있는 기능을 키고 끄는데 사용이 가능합니다.

builds_access_level이 CI/CD를 관리합니다.

4. 그외 다양한 항목들

보다 자세히 알고싶으시면 [공식문서](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/config_files.html) 를 참고하세요

```yml
projects_rules:
  - rule_name: my_rule_name 
    # 모든 항목은 project에 적용되는 항목들입니다.
    releases_access_level: enabled # releases 사용 유무
    infrastructure_access_level: enabled # infrastructure 사용 유무
    feature_flags_access_level: enabled # feature flags 사용 유무
    environments_access_level: enabled # environments  사용 유무
    monitor_access_level: enabled # Monitor 사용 유무
    pages_access_level: enabled # Page 사용 유무
    analytics_access_level: enabled # Analytics 사용 유무 (이하 사용유무 생략)
    forks_access_level: enabled # Fork
    security_and_compliance_access_level: enabled # Security and Compliance
    issues_access_level: enabled # issues
    repository_access_level: enabled # Repository
    merge_requests_access_level: enabled # MR
    wiki_access_level: enabled # Wiki
    builds_access_level: enabled # Build
    snippets_access_level: enabled # snippets
    container_registry_access_level: enabled # Container Registry
    model_experiments_access_level: enabled # Model Experiments
    model_registry_access_level: enabled # Model Registry
    requirements_access_level: enabled # Requirements
```

### Project members

```yml
member_profiles:
  - name: mytest_master_approvers
    role: maintainers
    members:
      - calamity.desperados
      - path/to/your/group

projects_rules:
  - rule_name: default_policy
    project_members:
      profiles:
        - mytest_master_approvers
      members:
        - role: developers
          name: calamity.desperados
        # list are also supported
        - role: maintainers
          names:
            - billy.thekid
            - buffalo.bill

  - rule_name: another_policy
    project_members:
      profiles:
        - mytest_master_approvers
```

프로젝트 멤버는 `project_members` 섹션에 정의할 수 있으며, 이는 프로파일 리스트 또는 멤버 리스트로 직접 지정할 수 있습니다.

- **프로파일(profiles)**: `member_profiles` 노드에 정의된 프로파일 리스트를 사용합니다. 이 경우, 지정된 멤버들은 프로파일에 정의된 역할을 갖게 됩니다.
  
- **멤버(members)**: 추가적인 멤버를 직접 정의할 수도 있으며, 이름과 역할을 설정할 수 있습니다. 멤버가 이미 프로파일에 설정되어 있는 경우, 직접 정의된 역할이 우선 적용됩니다.

위 예시에서는 `calamity.desperados` 사용자가 `developers` 역할을 갖게 됩니다.

> **참고**  
> 프로젝트의 직접 멤버를 추가하거나 제거할 수 있으며, 이는 프로젝트의 소유자가 아닌 경우에만 가능합니다. 이는 프로젝트가 포함된 그룹의 멤버가 그룹 권한을 유지한다는 것을 의미합니다.

사용 가능한 역할은 다음과 같습니다: `reporters`, `developers`, `guests`, `maintainers`, `owners`.

### Project Lables


여러개의 프로젝트 레이블을 정의할 수 있습니다

- **name**: 레이블 이름
- **color**: 레이블 색상

레이블 프로파일을 정의하고 이를 프로젝트 규칙에 적용할 수도 있습니다.

- **profile**: 프로파일 이름

> **중요**  
> 이 설정에 정의되지 않은 기존 레이블은 제거되지 않습니다.

### Access Token 설정

어떤 프로젝트가 `CI_JOB_TOKEN` 을 사용하여 귀하의 프로젝트에 접근할 수 있는지를 관리하고, 이 프로젝트의 `CI_JOB_TOKEN` CI/CD 변수를 사용하여 인증된 API 요청으로 접근할 수 있는 프로젝트를 선택하세요.

```yml
projects_rules:
  - rule_name: my_rule_name
    token_access:
      allow_access_with_ci_job_token: true
      allowed_projects:
        - group/project1
        - group/project2
      limit_ci_job_token_access: true
      limited_projects:
        - group/project3
        - group/project4
```

### Merge Requests Settings

MR은 다음 예시와 같은 방법으로 설정할 수 있습니다

```yml
projects_rules:
  - rule_name: my_rule_name
    mergerequests:
      merge_method: ff # ff(Fast-forward), merge(Merge commit), rebase_merge(Rebase), null
      squash_option: allow # Squash 사용 유무 (do not allow, allow, encourage, required 옵션 사용가능)
      only_allow_merge_if_pipeline_succeeds: true # 파이프라인 통과한 경우에만 머지 (true, false, null)
      only_allow_merge_if_all_discussions_are_resolved: true # 모든 discussions가 해결되야 머지 가능(true, false, null)
      resolve_outdated_diff_discussions: true # 자동 discussions 해결(Resolve)기능 켜기 끄기 (true, false, null)
      merge_pipelines_enabled: true # 머지 파이프라인 (프리미엄만 가능, true, false, null)
      merge_trains_enabled: true # 머지 트레인 (프리미엄만 가능, true, false, null)
      printing_merge_request_link_enabled: true # push 했을 때 command line에서 MR 링크 출력
      remove_source_branch_after_merge: true # Merge 된 뒤 소스 브랜치 삭제
      default_template: some string # MR 템플릿
```

### Merge Request Approval Settings

MR 허용은 다음과 같이 설정할 수 있습니다

일부 기능은 프리미엄만 지원되며

 `License-Check` and `Coverage-Check` 를 이용해서 사용가능유무를 확인하여 프로필을 구성해야 합니다.

```yml
# 아래에서 approvers 에 프로필을 입력하기 위해 미리 생성
member_profiles:
  - name: mytest_master_approvers
    role: maintainers
    members:
      - someone
      - path/to/your/group

projects_rules:
  - rule_name: my_rule_name
    approvers:
      minimum: 1 # Merge 에 필요한 승인 인원수
      can_override_approvals_per_merge_request: true # 병합 요청별로 필요한 승인자/승인 업데이트를 방지
      remove_all_approvals_when_new_commits_are_pushed: false # 새 커밋이 푸시되면 병합 요청의 모든 승인이 제거되도록 할 수 있습니다
      enable_self_approval: false # 셀프 승인 가능 유무
      enable_committers_approvers: false # 커밋을 한 사람이 승인할 수 있는지 유무
      
      members: # 승인을 할 수 있는 멤버를 지정할 수 있습니다(그룹도 가능). 삭제하고 싶다면 [] 를 입력하면 됩니다.
        - some.user

      profiles: # 프로필을 통해서도 승인자를 지정할 수 있습니다.
        - mytest_master_approvers
spell correctly
# 프리미엄만 가능
# 여러개의 승인 규칙을 지정할 수 있습니다.
approval_rules:
  - name: rule1
    profiles:
      - profile1_approvers
    members:
      - some_member
    minimum: 2
    protected_branches:
      - master
  - name: rule2
    profiles:
      - another_profile
    members:
      - another_member
    minimum: 8
    protected_branches:
      - shadow
  - name: rule3
    minimum: 4
    protected_branches:
      - a_branch_approved_by_default_eligible_approver
  - name: Coverage-Check # inbuilt rule
    minimum: 4
    profiles:
      - profile1_approvers
    members:
      - some_member
  - name: License-Check # inbuilt rule
    minimum: 4
    profiles:
      - profile1_approvers
    members:
      - some_member

# approval_rules와 approval_settings를 사용할 때는 approvers 섹션을 사용하지 않도록 주의해야 합니다(반대의 경우도 마찬가지입니다).
approval_settings:
    can_override_approvals_per_merge_request: true
    remove_all_approvals_when_new_commits_are_pushed: false
    enable_self_approval: false
    enable_committers_approvers: false
```

### Repository 설정

```yml
projects_rules:
  - rule_name: my_rule_name
    default_branch: master # 기본 브랜치 설정: 프로젝트당 하나의 기본 브랜치만 허용합니다. 해당 브랜치가 없으면 GPC는 아무 작업도하지않습니다
    description: my description # 프로젝트 설명
    ci_config_path: my_ci_config_file.yml # CI에 사용할 .ci파일
    deploy_keys:
      - id: 123456 # deploy key ID
        can_push: true # deploy key 푸쉬 가능 유무
    auto_cancel_pending_pipelines: enabled  # pending된 파이프라인 자동 취소
    # 커밋메세지 템플릿 지정 방법
    merge_commit_template: Merge %{source_branch} into %{target_branch} 
    # 스쿼시 커밋 템플릿
    squash_commit_template: |
        %{title}

        %{url}
        %{co_authored_by}
    # 이슈 템플릿
    issues_template: Describe the issue.
    # 보호된 브랜치 정책 설정 ( 관련 추가설정은 예시코드 아래에 있습니다 )
    protected_branches:
      - pattern: master # 와일드카드(*)를 사용하여 브랜치 집합을 구성할 수 있습니다. 이 값을 반드시 큰따옴표로 묶어 주세요! (예시 "stable/*")
        allowed_to_merge: maintainers # maintainers , developers, None 사용가능
        allowed_to_push: none # maintainers , developers, None 사용가능
        allow_force_push: false # (선택사항) force푸쉬가 가능한지 지정합니다.
        code_owner_approval_required: true # (선택사항) CODEOWNERS 파일의 항목과 일치하는 경우 이 브랜치로의 푸시를 방지

    # 보호된 태그
    # `protected_tags`가 정의되어 있고(즉, null 값이 아닌 경우), GPC(Global Project Configuration)에 구성되지 않은 모든 기존 보호 태그는 제거됩니다.
    protected_tags:
      - pattern: "master" # 와일드카드(*)를 사용하여 브랜치 집합을 구성할 수 있습니다. 이 값을 반드시 큰따옴표로 묶어 주세요! (예시 "stable/*")
        allowed_to_create: maintainers

    # 저장소를 복제할 때 GitLab에서 가져올 최대 변경 사항 수를 설정합니다
    # 얕게 할수록 복제 속도가 빨라집니다
    ci_git_shallow_clone: 50

```

> `member_profiles` 섹션에서 승인자(approvers)의 경우 역할(role)은 필수 항목이 아닙니다. 그러나 `project_members` 섹션에서 프로파일을 사용하고자 한다면, 역할을 정의해야 합니다.

복잡한 approval 설정의 예시는 아래와 같습니다.

```yml
  member_profiles:
    - name: mytest_master_approvers
      role: maintainers
      members:
        - roger.duvel
        - path/to/group2

    - name: mytest_profiles
      role: maintainers
      members:
        - roger.duvel
        - ricardo.corona
        - path/to/group/profiles

projects_rules:
  - rule_name: my_rule_name
    protected_branches:
      - pattern: master
        allowed_to_merge:
          role: maintainers
          profiles:
            - mytest_profiles
          members:
            - jean.kwak
            - path/to/group
        allowed_to_push: none
        allow_force_push: false # optional
        code_owner_approval_required: true # optional
```

- `protected_branches = null`(기본값)는 GPC가 이 값을 수정하지 않음을 의미합니다.
- `protected_branches = []`는 기존 보호 브랜치를 모두 제거합니다.

GitLab의 제한으로 인해 상속된 그룹을 `allowed_to_merge` 또는 `allowed_to_push` 멤버로 직접 추가할 수 없습니다.  
그러나 해당 그룹의 직접 멤버들(상속된 멤버가 아닌)은 추가할 수 있습니다. 이러한 멤버를 추가하고자 한다면 `apply_group_best_effort` 옵션을 사용할 수 있습니다.

> 주의! 여러 그룹을 추가할 수 있지만, 너무 많을 경우 속도가 느려질 수 있습니다.

예시:

```yml
projects_rules:
  - rule_name: my_rule_name
    apply_group_best_effort: true
    protected_branches:
      - pattern: master
        allowed_to_merge:
          role: maintainers
          members:
            - path/to/inherited/group
        allowed_to_push: none
        allow_force_push: false # optional
        code_owner_approval_required: true # optional
```

### CI/CD설정

```yml
variable_profiles:
  COMMON_VARIABLES:
    # 변수를 직접 할당하거나, 환경에서 가져올 수 있습니다(프로젝트)
    - name: SOME_VARIABLE_NAME
      value: some variable value
      protected: true
    - name: PRIVATE_VARIABLE
      value_from_envvar: SOMETHING_PRIVATE
      protected: true


projects_rules:
  - rule_name: my_rule_name
    # GPC에서 변수 값을 동일한 정규 표현식으로 검사하려면 현재 구성 프로젝트에서 보호되지 않은 변수와 함께 value 또는 value_from_envvar를 사용해야 합니다 (하지만 대상 변수는 보호될 수 있습니다).
    # 숫자 값으로 시작하는 변수를 정의하거나 사용할 수 없습니다! 모든 변수는 영문자로 시작해야 합니다.
    variables:
      - name: VARIABLE_NAME # 변수의 이름입니다. 대문자를 사용하는 것이 좋습니다.
        value: value for VARIABLE_NAME # 변수에 할당된 값으로, 구성 파일에 직접 설정됩니다.
        protected: true # (선택사항/기본값: False): 이 변수가 보호된 변수인지 여부를 정의합니다. 보호된 변수는 보호된 브랜치에서만 사용할 수 있습니다.
        variable_type: env_var # (선택사항)변수의 유형입니다. 기본값 (env_var)은 일반적인 환경 변수이며, 또는 파일(file)로 지정할 수 있습니다 (자세한 내용은 여기를 참조하세요)
      - name: MY_VARIABLE_MASKED
        value: masked_variable
        variable_type: env_var
        masked: true  # (선택사항) 이 변수가 마스킹된 변수인지 여부를 정의합니다. 값은 정규 표현식과 일치해야 합니다 (마스킹된 변수 문서를 참조하세요).
      - name: VALUE_FROM_ENV_VAR
        value_from_env_var: SOMETHING # 구성 프로젝트에 정의된 환경 변수에서 값을 가져옵니다.
  - rule_name: default_project_policy
    variables:
      - import: COMMON_VARIABLES  # "변수 프로필"에서 하나 이상의 변수를 정의하고 import 키워드를 사용하여 규칙에서 이를 가져올 수 있습니다.
  - rule_name: another_policy
    variables:
      - import: COMMON_VARIABLES
# 프로필에 대해 러너(runners)를 활성화하거나 비활성화할 수 있습니다.
# 이 러너들은 특정 러너(specific runners)여야 합니다. 공유 러너(shared runners)를 활성화/비활성화하는 것은 불가능합니다.
# 공유 러너를 정의하면 경고 메시지가 표시되지만, 특정 러너는 업데이트됩니다.
# 러너가 일시 중지된 상태일 경우, GPC는 이를 다시 시작할 수 없습니다. 일시 중지된 러너를 활성화하면 해당 러너는 활성화되지만 여전히 일시 중지 상태로 남아 있습니다.
runners:
  - runner_id: 666
    enabled: true

schedulers:
  - name: schedule_toto # 스케쥴러 이름
    branch: master # 스케쥴러가 사용할 브랜치
    cron: "0 4 1 * *" # 크론 문법으로 작성한 동작 시간
    enabled: true # (선택사항) 활성화 여부, 따로 설정 안할 경우 현재 설정 유지 / false
    tz: UTC # 타임존(크론)
    variables: # 파이프라인에서 사용할 변수를 설정하는게 가능합니다.
      - import: SET_OF_VARIABLES
      - name: OTHER_VARIABLE
        value: other value
      - name: var_2
        value_from_envvar: ENVVAR_VALUE
```

### 프로젝트 설정

```yml
badges:
  # 파이프라인 뱃지
  - name: pipeline
    link_url: "%{gpc_gitlab_url}/%{project_path}/commits/%{default_branch}"
    image_url: "%{gpc_gitlab_url}/%{project_path}/badges/%{default_branch}/pipeline.svg"
  # 커버리지 뱃지
  - name: coverage
    link_url: "http://some.server/path/%{project_path}/%{default_branch}/coverage/index.html"
    image_url: "%{gpc_gitlab_url}/%{project_path}/badges/%{default_branch}/coverage.svg"

# 아티팩트를 유지하는 기능을 비활성화하거나 활성화합니다.
artifacts:
  keep_latest_artifact: false


integrations:
  # Jira서비스를 설정할 수 있습니다.
  # 자세한 설정은 원본 문서를 참조하세요.
  jira:
    url: 'https://my.jira.server'
    jira_issue_transition_id: 101
    disabled: false
    username: my bot
    password_from_envvar: ENVVAR_PASSWORD
    trigger_on_commit: false
    trigger_on_mr: false
    comment_on_event_enabled: false
  
  # 프로젝트에 알림을 이메일로 보내도록 설정할 수 있습니다.
  pipelines_email:
    recipients: # (필수) 수신할 이메일주소
      - user1.name@someserver.com
      - user2.name@otherserver.com
    notify_only_broken_pipelines: false # (선택사항) 손상된 파이프라인에 대해 알림을 보냅니다.
    notify_only_default_branch: true # (선택사항) 오직 기본 브랜치의 활동만 알림을 보낼지 설정합니다.
    pipeline_events: true # (선택사항): 파이프라인 이벤트 알림을 활성화 합니다.
    disabled: false # (선택사항): 해당 알림을 비활성화 합니다.

# 푸쉬 알람 설정
# 세부적인 내용은 원본 링크를 확인해주세요
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

```

기본적으로 GPC로 설정되었음을 보여주는 뱃지가 설정됩니다

다음 사진과 같습니다

![GPC_BADGE](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/_images/gpc-badge.png)

### Keep 설정

기존 프로젝트의 구성을 유지하기 위해 목록으로 정의된 몇 가지 특정 `keep_...` 키워드가 존재합니다.

다음은 그 전체 목록입니다:

- `keep_existing_schedulers`
- `keep_existing_protected_branches`
- `keep_existing_protected_tags`
- `keep_existing_variables`
- `keep_existing_members`

이 키워드들은 기존 값을 삭제하지 않고 새로운 스케줄러, 보호된 브랜치, 태그, 변수 등을 추가할 수 있도록 해줍니다.

```yml
keep_existing_schedulers: true
schedulers:
  - name: schedule_toto
    branch: master
    cron: "0 4 1 * *"
    enabled: true
    tz: UTC  # by default
    variables:
      - name: OTHER_VARIABLE
        value: other value
      - name: var_2
        value_from_envvar: ENVVAR_VALUE

keep_existing_protected_branches: true
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

keep_existing_protected_tags: true
protected_tags:
  - pattern: master
    allowed_to_create: maintainers
  - pattern: protected_tag
    allowed_to_create:
      role: maintainers
      users:
        - robert.tripel
        - jean.kwak

keep_existing_members: true
project_members:
  members:
    - role: developers
      name: user_toto

keep_existing_variables: true
variables:
  - name: ANOTHER_VARIABLE_NAME
    value: another value
```

이 예시에서는 `keep` 옵션이 `true`로 설정되어 있기 때문에, 기존 프로젝트의 변수와 멤버가 유지됩니다.

`false`로 설정하면, 프로젝트 구성은 지정된 GPC 구성으로 덮어쓰게 됩니다.

예를 들어, `keep_existing_variables`가 `true`로 설정되어 있고 지정된 변수 중 하나가 프로젝트에 이미 존재하는 경우, 해당 변수는 덮어쓰여집니다!

## Group Rules

그룹 멤버를 정의하려면 `group_members` 섹션에서 프로필 목록이나 멤버 목록을 구성할 수 있습니다.

- **profiles**: `member_profiles` 노드에 정의된 프로필 목록입니다. 정의된 멤버는 프로필에 정의된 역할을 갖습니다.

- **members**: 추가 멤버를 정의할 수 있으며, 이름과 역할을 구성합니다. 멤버가 이미 프로필에 설정되어 있는 경우, 역할은 멤버에 의해 재정의됩니다.

이전 예시에서 사용자 `calamity.desperados`는 `developers` 역할을 갖게 됩니다.

사용 가능한 역할은 다음과 같습니다: `reporters`, `developers`, `guests`, `maintainers`, `owners`.

```yml
member_profiles:
  - name: mytest_master_approvers
    role: maintainers
    members:
      - calamity.desperados
      - path/to/your/group

groups_rules:
  - rule_name: default_policy
    group_members:
      profiles:
        - mytest_master_approvers
      members:
        - role: developers
          name: calamity.desperados
        # list are also supported
        - role: maintainers
          names:
          - billy.thekid
          - buffalo.bill
```

### Group Variables

자세한 정보는 프로젝트 변수를 참조하세요.

그룹 변수는 변수 프로필 및 프로젝트 변수에서 사용 가능한 모든 다른 옵션을 지원합니다.

```yml
variable_profiles:
  COMMON_VARIABLES:
    # Either, set variables directly
    # from this yml file
    - name: SOME_VARIABLE_NAME
      value: some variable value
      protected: true
    # or set variables from environment
    # variables of this project
    - name: PRIVATE_VARIABLE
      value_from_envvar: SOMETHING_PRIVATE
      protected: true

groups_rules:

  - rule_name: default_group_policy
    keep_existing_variables: true
    variables:
      - import: COMMON_VARIABLES
      - name: A_VARIABLE_NAME
        value: a_direct_value
        protected: true
      - name: SOME_PASSWORD
        value_from_envvar: SOME_PASSWORD
        protected: true
        masked: true
```

## Configurations

"구성(Configuration)"이란 프로젝트나 그룹에 규칙을 적용하는 것을 의미합니다. 규칙에서 벗어나야 하는 좋은 (또는 나쁜) 이유가 있을 수 있기 때문에, 프로젝트(또는 프로젝트 그룹)에서 규칙을 변경하여 "사용자 지정(Customization)"을 적용할 수 있는 기능이 있습니다.

이 변형이 더 높은 유지 보수 비용을 감수할 가치가 있는지는 유지 관리자의 판단에 맡겨집니다.

### Project configuration

`projects_configuration` 섹션에서는 어떤 정책을 어느 프로젝트에 적용할지를 선택할 수 있습니다.

예시 :

```yml
include:
    # your policies can be splitted into several files
    - policies.yml
    - policies_1.yml

projects_configuration:
  # It is possible to configure a list of projects
  - paths:
    - groupname/my_project01
    - groupname/my_project02
    rule_name: default_policy

  # or groups of projects, need to use recursive keyword for that
  - paths:
    - groupname/my_subgroup01
    - groupname/my_subgroup02
    recursive: true
    rule_name: another_policy

  # You can use globbing
- paths:
    - groupname/some/path/*/*_to_include
    - groupname/some/path/*/project_*
    recursive: true


    rule_name: another_policy
```

각 프로젝트 또는 프로젝트 그룹에 대해 정의된 정책을 재정의하여 `custom_rules`를 적용할 수도 있습니다.

```yml
include: policies.yml

projects_configuration:
  - paths:
      - groupname/my_project01
    rule_name: default_policy
    custom_rules:
      variables: null  # if the policy wanted to change add some variables,
                       # we actually want to keep existing variables for
                       # this project
      mergerequests:
        only_allow_merge_if_all_discussions_are_resolved: false

  - paths:
      - groupname/my_project02
    rule_name: default_policy
    custom_rules:
      protected_tags: null  # ignore change on protected tags
      mergerequests:
        merge_method: rebase_merge  # force rebase merge
        only_allow_merge_if_pipeline_succeeds: false
```

각 프로젝트 또는 프로젝트 그룹에 대해 여러 정책을 사용하여 적용하기 전에 병합할 수 있습니다.

```yml
include: policies.yml

projects_configuration:
  - paths:
      - groupname/my_project01
    rule_name:
      - default_policy
      - another_policy
```

특정 프로젝트에 대해 rule_name 없이 custom_rules를 사용하여 별도의 규칙을 생성하지 않고도 특정 규칙을 정의할 수 있습니다

```yml
include: policies.yml

projects_configuration:
  - paths:
      - groupname/my_project01
    custom_rules:
      mergerequests:
        merge_method: rebase_merge  # force rebase merge
        only_allow_merge_if_pipeline_succeeds: false
```

프로젝트 포함 및 제외방법:

`paths` 키는 규칙이 적용될 프로젝트 또는 그룹을 정의합니다.

전체 그룹 또는 프로젝트 경로를 사용할 수 있으며, 정규 표현식을 사용할 수도 있습니다. 정규 표현식은 `^`로 시작하고 `$`로 끝나야 합니다.

```yml
projects\_configuration:
  - paths:
    - groupname/my_group01
    - groupname/my_group02
    excludes:
      - ^.*demo$
      - ^groupname\/my_group01\/test_.*$
    recursive: true
    rule_name: another_policy
```


> 중요
> 제외 정규식에서 각 슬래시 `/`를 이스케이프하는 것을 잊지 마세요!
> 프로젝트 경로가 "a/path"일 경우, 정규 표현식은 "^a\/path$"가 됩니다.

기본적으로 프로젝트 목록은 재귀적이지 않습니다. 하위 그룹을 활성화하려면 `recursive: true`를 정의하세요.

`excludes` 키워드를 사용하여 일부 프로젝트를 구성에서 제외할 수도 있습니다.

전체 하위 그룹을 제외하려면 다음 패턴을 사용하세요:

```yml
- paths:
  - groupname/my_group
  excludes:
    - ^groupname\/my_group\/test_.*$
```

주어진 예제는 `my_group` 그룹에 포함된 모든 프로젝트를 구성하되, 이름이 `test_`로 시작하는 프로젝트는 제외합니다.

### Apply on all projects

정의된 정책을 사용자가 접근할 수 있는 모든 프로젝트에 적용할 수도 있습니다.

이를 위해 "paths" 필드에 "/"를 정의해야 합니다.

```yml
include: policies.yml

projects_configuration:
  - paths:
      - /
    rule_name: default_policy
    custom_rules:
      variables: null  # null means "keep them as they are"
      mergerequests:
        only_allow_merge_if_all_discussions_are_resolved: false
```

여기서 설명하는 대부분의 값은 기본값으로 `null`을 가지며, 이는 GPC에서 "건드리지 않음"을 의미합니다.

리스트의 경우에도 `null`은 여전히 "건드리지 않음"을 의미하지만, 빈 리스트(예: `approvers: []`)는 "모두 제거"를 뜻한다는 점을 알아두어야 합니다.

### Customization

때때로 규칙을 적용해야 하지만, 특정 한두 개의 설정을 제외하거나 특정 값을 강제해야 할 필요가 있을 수 있습니다. 이는 일반적으로 바람직하지 않은 관행으로 볼 수 있습니다.

```yml
projects_configuration:
  - paths:
      - some/project
    rule_name: default_policy
    custom_rules:
      protected_tags: null  # ignore change on protected tags
      mergerequests:
        merge_method: rebase_merge  # force rebase merge
```

프로젝트 `some/project`에 대해 `default_policy` 규칙을 적용하되, `protected_tags` 설정은 변경하지 않고, `merge_method`를 `merge_rebase`로 강제하고 싶습니다.

이는 규칙에 다른 방식으로 설정되어 있더라도 적용됩니다.

### "Not ssen yet"

기본적으로 GPC는 매우 직관적입니다. 각 설정이 순차적으로 실행되므로, 동일한 프로젝트에 규칙이 두 번 적용되는 것을 기본적으로 방지하지 않습니다.

이러한 동작을 비활성화하려면 `not_seen_yet_only: true`를 설정하여 이미 구성된 프로젝트를 제외할 수 있습니다.

예를 들어, 그룹 내 모든 프로젝트에 단일 규칙을 적용하되 특정 프로젝트는 제외하고자 할 때 유용합니다.

다음은 `groupname/my_group01` 아래의 모든 프로젝트에 `general_policy`를 적용하되, `groupname/my_group01/my_project05`는 제외하고 `exception_policy`를 적용하는 방법을 보여줍니다.

```yml
projects_configuration:
  - paths:
      - groupname/my_group01/my_project05
    rule_name: exception_policy

  - paths:
        - groupname/my_group01
    recursive: true
    not_seen_yet_only: true
    rule_name: general_policy
```

### Group configuration

프로젝트 구성과 관련하여, 그룹을 구성할 수 있는 특정 섹션이 제공됩니다.

```yml
include:
    - policies.yml
    - policies_1.yml

groups_configuration:
  # It is possible to configure a list of groups
  - paths:
    - groupname/my_group01
    - groupname/my_group02
    rule_name: default_policy
```