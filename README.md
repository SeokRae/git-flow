# GitFlow Best Practices

## 참고

- [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)
- [Understanding the GitHub flow](https://guides.github.com/introduction/flow/)

## Git Basic

> **깃 플로우 모범 사례**

![git flow](images/gitflow.png)

- Developers must implement all tests before they commit code.
- Stand up CI automation and trigger builds for every pull request so that bad changes can be rejected.
- Ensure that all tests are executed and linting and static code analysis is done for every PR.
- Implement CI practices. All developers branch from the trunk, make changes, and submit PRs back to the trunk.
    - The branches are removed in less than 24 hours.

## The main branches

- 중앙 저장소에는 수명이 두 가지 주요 분기가 있다.
    - master
    - develop

- `origin/master`는 `HEAD`의 소스 코드가 항상 `프로덕션 준비 상태`를 반영하는 주요 분기로 간주한다.
- `origin/develop`는 `HEAD`의 소스 코드가 항상 `다음 릴리스에 대해 가장 최근에 제공된 개발 변경 사항이 있는 상태`를 반영하는 주요 분기로 간주한다.
    - 어떤 사람들은 이것을 `통합 분기(integration branch)`라고 부르고, 여기에서 automatic nightly builds 가 빌드됩니다.(이건 무슨 말이지?)

- `develop branch`의 소스 코드가 안정적인 지점에 도달하고, 릴리스할 준비가 되면 모든 변경 사항을 어떻게든 다시 마스터에 **병합**한 다음 릴리스 번호로 `태그`를 지정해야 한다.
- 변경 사항이 다시 마스터로 **병합**될 때 새로운 프로덕션 릴리스로 정의 할 수 있다.

## Supporting branches

- master 및 develop 과 같은 `주요 분기`는 개발 팀 구성원 간의 병렬 개발을 지원한다.
- 기능을 쉽게 추적할 수 있고, 프로덕션 릴리즈를 준비하고, 라이브 프로덕션 문제를 신속하게 수정하는 데 도움이 되는 다양한 `지원 분기`를 사용한다.
- 메인 브랜치와 달리 지원 분기는 결국 제거될 것이기 때문에 제한된 수명을 가지고 있다.

- 다양한 유형의 분기
    - **Feature branches**
    - **Release branches**
    - **Hotfix branches**

- 이러한 각 분기는 특정 목적을 가지고 있으며, 어떤 분기가 원래 분기가 될 수 있고, 어떤 분기가 병합 대상이 되어야 하는지에 대한 엄격한 규칙이 적용된다.

### Feature branches

- `develop` 분기로부터 새롭게 분기될 수 있고, develop으로 병합된다.
- **분기 명명 규칙**
    - `master`, develop, release-*, or hotfix-* 를 제외한 모든 것

- `기능(feature) 분기`는 향후 릴리즈 또는 먼 미래 릴리즈의 새 기능을 개발하는데 사용된다.
- 기능 개발을 시작할 때 이 기능이 통합될 대상 릴리즈는 그 시점에서 잘 알려지지 않을 수 있다.
- 기능 분기의 본질은 기능이 개발 중인 동안 존재하고 결국에는 결국에는 다시 develop으로 병합되거나 폐기된다.
    - 병합되는 경우는 새 기능을 다음 릴리즈에 확실하게 추가 하는 경우
    - 폐기하는 경우는 해당 기능을 적용하지 않는 경우

> Creating a feature branch

- 새로운 기능에 대한 작업을 시작할 때 develop에서 분기한다.

```shell
git checkout -b feature develop 
```

> Incorporating a finished feature on develop

- 완성된 기능은 다음 릴리즈에 추가하기 위하여 develop 분기에 merge 될 수 있다.

```shell
git checkout

git merge --no--ff feature xxx

git branch -d feature feature

git push origin develop
```

- `--no-ff` 플래그는 merge가 fast-forward로 수행될 수 있는 경우에도 병합이 항상 새 커밋 개체를 생성하도록 한다.
- 기능 분기로부터 개발을 위해 `분기된 정보가 손실되는 것을 방지`하고, 기능을 개발한 모든 커밋을 **그룹화**할 수 있다.
- 후자의 경우 git 기록에서 어떤 커밋들이 함께 기능을 구현했는지 확인할 수가 없다.
- 모든 로그 메시지를 수동으로 읽어야 하는 문제가 생긴다.
- 전체 기능을 되돌리는 것은 후자의 경우 골치아프므로 `--no-ff` 플래그를 사용하면 쉽게 수행할 수 있다.
- 몇 개의 커밋 개체를 더 생성하는 것처럼 보이지만 그러한 비용보다 이점이 더 크다.

![merge](images/no-ff.png)

### Release branches

- `develop branch`에서 분기되어 `develop` 또는 `master branch`로 merge해야 한다.
- 분기 명명 규칙
    - `release-*`

- 릴리즈 분기는 새로운 프로덕션 릴리즈를 준비하기 위해 지원한다.
- 사소한 버그 수정 및 릴리즈에 대한 메타 데이터 준비(버전 번호, 빌드 날짜 등)을 허용한다.
- `release branch`에서 이 모든 작업을 수행하면 `develop branch`가 다음 큰 릴리스의 기능을 받을 수 있도록 지워진다. 

- `develop`에서 새 `release branch`를 분기하는 순간은 `develop branch`가 새 `release`의 원하는 상태를 반영할 때입니다.
- 이 시점에서 개발을 지속적으로 하기 위해서는 적어도 `빌드 예정 릴리즈(release-to-be-built)`를 대상으로 하는 모든 `feature`를 `merge`해야 한다.
- 향후 `release`를 대상으로 하는 모든 `feature`는 그렇지 않을 수도 있는데 이는 `release branch`가 분기 될 때까지 기다려야 한다.


> Creating a release branch

> Finishing a release branch

### Hotfix branches

> Creating the hotfix branch

> Finishing a hotfix branch 

