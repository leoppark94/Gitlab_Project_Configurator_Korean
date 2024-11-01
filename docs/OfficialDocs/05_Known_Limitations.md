---
title: 05. Known_Limitations
author: Leo
date: 2024-11-01
layout: default
parent: OfficialDocs
---
알려진 한계점에 대해서 다루고 있는 문서입니다.

다만 정확한 최신상황을 반영하기 위해서는 [공식 문서](https://grouperenault.gitlab.io/gitlab-project-configurator/docs/known_limitations.html)를 참조해주세요.

## Approvers

GPC가 사용자를 승인자로 추가할 때, 해당 사용자가 GitLab의 적격 승인자가 아닐 경우 승인자 목록에 아무런 영향을 미치지 않을 수 있습니다.

[문서 참조](https://docs.gitlab.com/ee/user/project/merge_requests/merge_request_approvals.html#eligible-approvers) 

사용자는 다음 조건을 만족할 경우 프로젝트의 승인자로 추가될 수 있습니다:

- 프로젝트의 멤버일 경우
- 프로젝트의 직속 부모 그룹의 멤버일 경우
- 공유를 통해 프로젝트에 접근 권한이 있는 그룹의 멤버일 경우

## 레이블 관리

GPC는 프로젝트에 레이블을 추가하거나 업데이트할 수 있지만, GPC 설정에 포함되지 않은 기존 레이블은 제거하지 않습니다.

> **참고:** 이는 실제로 알려진 제한 사항이라기보다는 의도된 동작입니다.

레이블은 다양한 용도로 사용될 수 있으므로, GPC 외부에서 정의할 수 있도록 허용합니다.

## 배지

레이블과 마찬가지로, 기존 배지를 변경하지 않습니다.

## 외부 유저

GPC가 외부 사용자를 승인자, 멤버, 보호된 태그 또는 브랜치에 추가할 때 몇 가지 제한 사항이 있습니다. GPC는 사용자를 프로젝트에 추가하기 위해 사용자 ID를 찾아야 하지만, 비관리자 토큰으로는 이를 허용하지 않습니다. 만약 이 외부 사용자가 프로젝트의 멤버라면, GPC는 그를 승인자로 추가하거나 보호된 브랜치에 병합/푸시할 권한을 부여할 수 있습니다.

> **참고:** 해결책은 외부 사용자로 그룹을 생성하고, 이 그룹을 승인자, 멤버 등으로 추가하는 것입니다.
