---
layout: post
title: 'Backtracking Programming Problems'
date: 2019-12-12 22:00:00 +09:00
categories: 'algorithms'
published: true
---

Reference: [Back to Back SWE - The Backtracking Blueprint](https://youtu.be/Zq4upTEaQyM)

## 개요

Sudoku를 통해 알아보는 Backtracking

핵심: Choice / Constraints / Goal

- Choice: You have a decision
- Constraints: Your decisions are restricted
- Goal: Your goal

어떻게 이것을 풀 것인가?

1. Choice: 문제가 주어졌을 때 어떻게 풀 것인가?

- Sudoku board의 경우 board의 cell을 채워야 한다.
- 여기서 cell을 주목하자.
- cell은 row와 column에 있다. <- 이 부분이 우리의 리드다

```java
solve(row, col ...) {

}
```

자 이제 어떻게 문제를 어떻게 진행할까?

2. 전체 Sudoku 보드를 전부 접근하지 말고, 가장 첫번째 row부터 집중하는 것으로 narrow down 해보자.

3. 이제 cell에 뭘 채울 수 있는지를 생각해보자. 1 ~ 9 사이의 숫자를 채울 수 있다.

- 이제 빈 cell에 대해서 반복문을 넣어보자.

```java
solve(row, col ...) {

  for (int value = 1; value <= 9; value++) {
    board[row][col] = value;
  }

}
```

4. 이제 모든 항목을 넣을 수는 없으니까, Constraint를 결정하자.

- 여기서 validation에 해당하는 항목이 들어간다.
  - row를 break하는가?
  - column을 break하는가?
  - sub-group을 break하는가?

```java
solve(row, col ...) {

  for (int value = 1; value <= 9; value++) {
    board[row][col] = value;

    if (validPlacement(row, col, ...)) {
      if (solve(row, col + 1, board)) {
        return true;
      }
    }

  }

}
```

5. 검증 과정을 거치고 다음번 solve로 넘어간다.

- 그런데 왜 `col`일까? 왜냐면, 하나를 해결하면 다음으로 넘어가고, 그 다음으로 넘어가기 때문이다.
- `col`을 다 했는데, 아직 `row`가 끝이 아니라면 어떻게 될까?
  - 자연스럽게 다음 번 `row`로 넘어간 후, 첫 번째 cell부터 작업을 다시 하면 된다.

6. 여기서 base case가 생긴다.

```java
solve(row, col ...) { // Sub problem
  if (col == board[row].length) {
    // ...base case question
  }

  for (int value = 1; value <= 9; value++) { // Decision space
    board[row][col] = value;

    if (validPlacement(row, col, ...)) { // Converge to a base case
      if (solve(row, col + 1, board)) {
        return true;
      }
    }
  }

  board[row][col] = EMPTY_ENTRY; // undo our decision when it fails

}
```
