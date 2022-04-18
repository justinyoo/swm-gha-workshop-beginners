# 헬로월드 워크플로우 만들기 #

## 매트릭스 사용하기 #1 ##

```yaml
name: 'SWM GitHub Actions Basic'

on: push

jobs:
  first-job:
    name: 'First Job'

    strategy:
      matrix:
        os: [ 'windows-latest', 'macos-latest', 'ubuntu-latest' ]

    runs-on: ${{ matrix.os }}

    steps:
    - name: Say Hello World on ${{ matrix.os }}
      shell: bash
      run: |
        echo "Hello World on ${{ matrix.os }}"
```


## 매트릭스 사용하기 #2 ##

```yaml
name: 'SWM GitHub Actions Basic'

on: push

jobs:
  first-job:
    name: 'First Job'

    strategy:
      matrix:
        os: [ 'windows-latest', 'macos-latest', 'ubuntu-latest' ]
        message: [ 'Hello World', 'Lorem Ipsum' ]

    runs-on: ${{ matrix.os }}

    steps:
    - name: Greetings from ${{ matrix.os }}
      shell: bash
      run: |
        echo "${{ matrix.message }} from ${{ matrix.os }}"
```


## 조건문 사용하기 &ndash; 잡 수준 ##

```yaml
name: 'SWM GitHub Actions Basic'

on: [ 'push', 'pull_request' ]

jobs:
  first-job:
    name: 'First Job'
    if: github.event_name == 'pull_request'

    strategy:
      matrix:
        os: [ 'windows-latest', 'macos-latest', 'ubuntu-latest' ]

    runs-on: ${{ matrix.os }}

    steps:
    - name: Say Hello World from ${{ matrix.os }}
      shell: bash
      run: |
        echo "Hello World on ${{ matrix.os }}"
```


## 조건문 사용하기 &ndash; 스텝 수준 ##

```yaml
name: 'SWM GitHub Actions Basic'

on: push

jobs:
  first-job:
    name: 'First Job'

    strategy:
      matrix:
        os: [ 'windows-latest', 'macos-latest', 'ubuntu-latest' ]

    runs-on: ${{ matrix.os }}

    steps:
    - name: Say Hello World from ${{ matrix.os }} 1
      shell: bash
      run: |
        echo "Hello World on ${{ matrix.os }} 1"

    - name: Say Hello World from ${{ matrix.os }} 2
      if: matrix.os == 'ubuntu-latest'
      shell: bash
      run: |
        echo "Hello World on ${{ matrix.os }} 2"
```


## 깃헙 액션 이벤트 ##

```yaml
name: 'SWM GitHub Actions Basic'

on:
  push:
    branches:
    - main
    tags:
    - 'v*'
  pull_request:
    branches:
    - main

jobs:
  first-job:
    name: 'First Job'

    runs-on: ubuntu-latest

    steps:
    - name: Say Hello World from ${{ github.ref }}
      shell: bash
      run: |
        echo "Hello World from ${{ github.ref }}"
```


## 깃헙 액션 잡 체이닝 ##

```yaml
name: 'SWM GitHub Actions Basic'

on:
  push:
    branches:
    - main

jobs:
  first-job:
    name: 'First Job'

    runs-on: ubuntu-latest

    steps:
    - name: Say Hello World from first job
      shell: bash
      run: |
        echo "Hello World from first job"

  second-job:
    name: 'Second Job'
    needs: first-job

    runs-on: ubuntu-latest

    steps:
    - name: Say Hello World from second job
      shell: bash
      run: |
        echo "Hello World from second job"
```
