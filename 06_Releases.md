---
title: Releases
author: Leo
date: 2024-11-01
layout: post
---

프로젝트에서 GPC를 실행하는 주요 방법은, Gitlab-ci파일에서 직접 docker image를 실행하는 방법입니다.

## 사용방법

```yml
apply:
  # "latest" 나 "unstable" 라벨을 사용하면 가장 최신 버전을 사용할 수 있습니다.
  image: registry.gitlab.com/grouperenault/gitlab-project-configurator:latest

  # "stable" 라벨을 사용하면 릴리즈 버전을 사용할 수 있습니다.
  image: registry.gitlab.com/grouperenault/gitlab-project-configurator:stable

```

### 주의사항

> 우리는 이전 버전을 유지 관리하지 않는다는 점을 유의해 주시기 바랍니다. 
> 예를 들어, v2의 개발이 완료된 후에는 v1에 대한 버그 수정이나 백포트가 없습니다. 
> v3 개발 중에 v2를 분기하는 것은 옵션이지만, GPC의 다양한 버전을 너무 많이 유지 관리하는 데 노력을 들이고 싶지는 않습니다.

간단히 말해서, 버그 수정은 주로 HEAD에서 진행되며 새로운 버전과 함께 릴리스될 것입니다.

여러 가지 "채널"의 이미지가 제공됩니다:

| Docker 이미지 채널      | 예시         | 설명                                                                 |
|---------------------|------------|--------------------------------------------------------------------|
| Unstable/Latest 라벨 | `unstable` | "master"에서의 개발을 따릅니다.                                       |
|                     | `latest`   |                                                                    |
| Stable 라벨         | `stable`   | EPIC 종료 시 릴리스 태그가 지정됩니다.                                   |
| 시맨틱 버전 릴리스   | `1.0.0`    | 각 안정 버전에 대한 라벨입니다. <br> 불안정한(개발) 버전은 시맨틱 버전 라벨이 **아닙니다**. |
|                     | `1.1.0`    |                                                                    |
|                     | `1.1`      |                                                                    |
|                     | `1`        |                                                                    |
| 날짜 버전            | `c19.04.10`| 각 성공적인 빌드 후에 게시됩니다. <br> 예시: `c19.10`은 2019년 10월의 최신 빌드를 사용합니다. |
|                     | `c19.04`   |                                                                    |
|                     | `c19`      |                                                                    |
