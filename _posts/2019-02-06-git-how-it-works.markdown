---
layout: post
title: 'Git the inner parts'
date: 2019-02-06 15:00:00 +09:00
categories: 'git'
published: true
---

## Git은 어떻게 동작하는 것일까?

직접 프로젝트를 만들어 Git의 동작 원리를 파악해보자.

먼저, 프로젝를 생성한다.

```bash
mkdir git-test
cd git-test
```

이제, 해당 프로젝트에 디렉토리를 만들어본다.

```bash
mkdir data
echo "Hello, World!" > data/text.txt
```

```bash
git init
tree -a
```

```
.
├── .git
│   ├── HEAD
│   ├── branches
│   ├── config
│   ├── description
│   ├── hooks
│   ├── info
│   │   └── exclude
│   ├── objects
│   │   ├── info
│   │   └── pack
│   └── refs
│       ├── heads
│       └── tags
└── data
    └── text.txt
```

```
git add data/text.txt
```

1. `.git/objects` 폴더 내부에 새로운 blob 파일을 생성한다.
   해당 blob 파일은 `data/text.txt` 파일의 압축된 컨텐츠를 포함한다. Hashing으로 진행되기 때문에 원본을 독자적으로 나타내는 작은 양의 코드로 변환된다. 예를들어, 아까 생성한 `text.txt` 파일의 내용은 `3f72385e49fbffe041da6cd0b7e79e9dd8fde7c754`로 hashing되며, 첫 두 글짜는 폴더명, 나머지는 파일명으로 사용된다.
   `.git/objects/3f/7238`
1. `git add`를 통해 파일이 `index`에 추가된다. `index`는 Git이 트래킹을 해야하는 파일들을 전부 가지고 있는 리스트의 개념이다.

```
echo "1234" > data/numbers.txt
```

```
$ git add data
```

Working Copy와 index가 있는데, Working copy는 내가 현재 작업하고 있는 것, index는 git에서 인식하는 최신본. `git add` 해주기 전까진 이전 파일로 알고 있게된다.

## 커밋하기

```
$ git commit -m "test1"
[master (root-commit) 8cf03c2] test1
 2 files changed, 2 insertions(+)
 create mode 100644 data/numbers.txt
 create mode 100644 data/text.txt
```

커밋의 단계

1. 커밋되는 프로젝트 컨텐츠의 버전을 나타내는 Tree Graph를 생성한다.
2. 커밋 객체를 생성한다
3. 현재 브랜치를 새로운 커밋 객체를 향하게 한다.

### Tree Graph 생성

깃은 `index`로부터 tree graph를 생성함으로써 프로젝트의 현재 상태를 기록한다. 해당 tree graph는 프로젝트 내의 모든 파일들의 장소와 컨텐츠를 기록한다.

Blob와 Tree로 이루어짐
Blob는 `git add` 명령어를 통해 저장된다. 파일의 컨텐츠를 나타낸다.

Tree는 커밋이 되었을 때 저장된다. Tree는 Working copy에서의 경로를 나타낸다.

```
$ git cat-file -p 8cf03c2
tree d5a5b8
author immigration9 <projgotham18@gmail.com> 1549439777 +0900
committer immigration9 <projgotham18@gmail.com> 1549439777 +0900

test1
```

- tree: 타입
- d5a5b...: hash
- author / committer: 작성자 - 이메일 - 타임스탬프
- test1: commit message

```
$ git cat-file -p d5a5b8
040000 tree b27100	data
```

- tree가 나타내는 것은 directory가 기준임을 알 수 있다.
- blob는 해당 파일의 컨텐츠를 볼 수 있다.

```
$ git cat-file -p b27100
100644 blob 97b595	numbers.txt
100644 blob 3fa0d4	text.txt
```

### HEAD

HEAD는 현재 어떤 브랜치에 있는지를 알려주는 파일이다.

```
$ cat .git/HEAD
ref: refs/heads/master

$ cat .git/refs/heads/master
8cf03c
```

### 추가 Commit

```
$ echo "87654321" > data/numbers.txt
$ git add data/numbers.txt
$ git commit -m "test2"
[master 50c19b6] test2
 1 file changed, 1 insertion(+), 1 deletion(-)
```

```
$ git cat-file -p 50c19b6
tree f38d57
parent 8cf03c
author immigration9 <projgotham18@gmail.com> 1549442502 +0900
committer immigration9 <projgotham18@gmail.com> 1549442502 +0900

test2
```

추가 커밋시 발생 순서

1. 'test2'를 기준으로 index에 컨텐츠에 대한 tree graph를 생성
2. 커밋 객체 생성 (test2)
3. HEAD를 새로운 커밋으로 이동

변경되지 않은 `data/text.txt` 파일은 이전 커밋의 파일을 그대로 이용하고,
변경된 `data/numbers.txt` 파일은 새로운 커밋의 파일을 이용한다.

결국, 객체는 불변적이고, Refs만이 변형적이다.

### 커밋 Checkout

커밋을 Checkout하는 것은 다소 의아할 수 있긴한데 (일반적으로는 branch를 Checkout하기 때문에), 우선 진행해보자.

```
$ git checkout 50c19b6
Note: checking out '50c19b6'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 50c19b6 test2
```

`detached HEAD` 상태에 들어왔다. 이제 변화를 줘서 새롭게 커밋을 해보자.

```
$ echo "3" > data/numbers.txt
$ git add data/numbers.txt
$ git commit -m "test3"
[detached HEAD 2e04bb9] test3
 1 file changed, 1 insertion(+), 1 deletion(-)
```

브랜치가 다소 이상하게 표현되어 있는 것을 알 수 있다. 이제 새로운 브랜치를 생성해보자.

### Branch 생성

```
$ git branch prod
$ cat .git/refs/heads/prod
2e04bb
```

이제, HEAD가 test3 커밋을 가리키고 있는 상태에서, 기존의 `master` 브랜치는 test2 커밋을 가리키고, `prod` 브랜치는 test3 커밋을 HEAD와 동일하게 가리킨다.

### Branch 체크아웃

```
$ git checkout master
Previous HEAD position was 2e04bb9 test3
Switched to branch 'master'
```

1. 커밋 내용을 Working copy에 쓴다
2. 커밋 내용을 index에 쓴다
3. HEAD를 체크아웃 대상으로 옮긴다.

### Merging

```
$ git checkout prod
Switched to branch 'prod'

$git merge master
Already up to date.
```

아무일도 일어나지 않는 이유는?

- 커밋은 변경된 사항들의 종합이다. 즉, 조상(ancestor)이 후손(descendent)으로 머지될 경우, 깃은 아무 일도 하지 않는다.

### 후손을 Merge

이제 반대로, 조상 브랜치에서 후손 브랜치를 머지하도록 한다.

```
$ git checkout master
Switched to branch 'master'

$ git merge prod
Updating 50c19b6..2e04bb9
Fast-forward
 data/numbers.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

- Fast-forward: 후손이 조상으로 머지될 경우, 히스토리에는 변경사항이 없지만, HEAD가 가리키고 있는 위치가 달라진다.

### 서로 다른 두 개의 커밋을 머지

`master` 브랜치에 test4 커밋 생성

```
$ echo "13579" > data/numbers.txt
$ git add data/numbers.txt
$ git commit -m 'test4'
[master 9c2d6ae] test4
 1 file changed, 1 insertion(+), 1 deletion(-)
```

`prod` 브랜치에 test-prod3 커밋 생성

```
$ git checkout prod
$ echo "Goodbye, World" > data/text.txt
$ git add data/text.txt
$ git commit -m "test-prod3"
[prod fffe746] test-prod3
 1 file changed, 1 insertion(+), 1 deletion(-)
```

커밋은 여러개의 부모 커밋을 가질 수 있다.
Merge commit을 사용하면 분기되어 있는 구조(lineage)가 통합될 수 있다.

```
$ git merge master -m "test-merge"
Merge made by the 'recursive' strategy.
 data/numbers.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

- Git에는 베이스 커밋이 존재한다: Git은 수신자나 송신자에만 존재하는 베이스 커밋으로부터 변화된 파일에 대한 머지를 자동으로 해결한다.

1. Merge 수신자와 송신자에 의해 생성된 변경점들을 조합하는 diff를 만든다.
2. diff 값을 Working copy에 적용
3. diff 값을 index에 적용
4. 업데이트 된 index 값을 커밋
5. 새로운 커밋으로 HEAD를 이동

![git-merge-flow](../_images/git-merge-flow.png)

### 같은 파일을 변경하는 브랜치 머지

동일한 브랜치 유지를 위해 머지부터 시행한다.

```
$ git checkout master
Switched to branch 'master'
 ~/git/git-test   master  git merge prod
```

우선 prod 브랜치 내용 변경 후 커밋 작업

```
$ git checkout prod
Switched to branch 'prod'
$ echo '5' > data/numbers.txt
$ git add data/numbers.txt
$ git commit -m 'test-prod5'
[prod 74c5b8c] test-prod5
 1 file changed, 1 insertion(+), 1 deletion(-)
```

그 다음, master 브랜치 내용 변경 후 커밋 작업

```
$ git checkout master
Switched to branch 'master'
$ echo '6' > data/numbers.txt
$ git add data/numbers.txt
$ git commit -m 'test6'
[master 7a4c8aa] test6
 1 file changed, 1 insertion(+), 1 deletion(-)
```

이제 머지 작업을 진행하면, master와 prod는 각각 test6와 test-prod5의 커밋을 최신으로 가지고 있으며, 베이스 커밋으로는 test-merge를 갖게 된다.

머지를 진행해보자

```
$ git merge prod
Auto-merging data/numbers.txt
CONFLICT (content): Merge conflict in data/numbers.txt
Automatic merge failed; fix conflicts and then commit the result.
```

Conflict이 발생하여 진행이 되지 않는다. `data/numbers.txt` 파일은 아래와 같이 변경되어 있는 것을 확인할 수 있다.

```
<<<<<<  HEAD
6
=======
5
>>>>>>  prod
```

위에 있는 항목이 현재 브랜치에서 온 것이고, 아래 있는 항목이 머지 대상 브랜치에서 온 값이다.

아래와 같이 수정을 진행하자.

1. Working copy에 있는 Conflict를 해결한다.
2. index에 있는 Conflict를 해결한다

```
$ echo '11' > data/numbers.txt
$ git add data/numbers.txt
```

마지막으로 `git merge`를 해주면 끝난다.

### Merge conflict 발생시 어떤 방식으로 돌아가는 것일까?

위 상황을 재현해보자.
master 브랜치와 prod 브랜치에 다른 값을 넣고 merge를 돌려본다.

```
$ git ls-files --stage
100644 7ed6ff 1	data/numbers.txt
100644 b1bd38 2	data/numbers.txt
100644 48082f 3	data/numbers.txt
100644 db102a 0	data/text.txt
```

`git ls-files`: index와 작업 중인 tree에 속해있는 파일들에 대한 정보를 보여준다.
위에 보면, `data/numbers.txt` 파일이 총 3개로 늘어난 것을 알 수 있는데, 각각 베이스 커밋 / master 커밋 / prod 커밋을 나타낸다.
