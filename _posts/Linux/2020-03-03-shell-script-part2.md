---
layout: post
title:  "리눅스 쉘 스크립트 (2)"
date:   2020-03-04 22:28:25 +0900
categories: Linux
tags: Linux Shell-Script
---

## 쉘 스크립트의 if 명령어
일반적으로 프로그래밍 언어에서의 if문은 다음에 오는 식이 True인지 False인지에 따라 실행이 분기된다.<br>
그러나 리눅스의 if문은 **다음에 오는 명령어의 실행 성공 여부**에 따라 실행이 분기된다.
```bash
# 명령어 'ls'의 실행 결과에 따라 분기. 1> /dev/null로 표준 출력을 무시한다.
$ if ls 1> /dev/null; then echo "success"; else echo "fail"; fi
success
```
```bash
# 'rm /' 명령어가 실행되지 못한다(=실패한다). 2> /dev/null로 표준 에러를 무시한다.
$ if rm / 2> /dev/null; then echo "success"; else echo "fail"; fi
fail
```

## 종료 상태
`bash`는 명령어 실행이 성공인지 실패인지에 따라 다른 종료 상태(exit status) 값을 갖는다. 명령어 성공 시 0을 갖고, 
실패 시 0이 아닌 값을 갖게 된다. 종료 상태 값은 ? 변수의 값을 출력하여 확인할 수 있다.
```bash
$ ls 1> /dev/null
$ echo $?
0
```
```bash
$ rm / 2> /dev/null
$ echo $?
1
```

## test 명령과 싱글 브라켓([ ])
`test` 명령은 표현식의 결과가 true인지 false에 따라 명령어 성공인지(종료 상태 값 0) 실패인지(0이 아닌 종료 상태 값) 
테스트한다. `if` 명령어와 함께 사용되어 표현식의 true, false에 따른 분기 실행을 가능하게 해준다. `test` 명령어는 
다음의 두 가지 방법으로 사용할 수 있다. 다만 주의할 점은 싱글 브라켓([ ]) 표현을 사용할 경우 반드시 시작 브라켓([) 다음에는
공백이 존재해야 한다. 이는 '[' 또한 `test` 명령어이고 ']'을 만날때 까지 공백 문자를 기준으로 인자를 구분하기 때문이다.
bash 2.x 버전에서 더블 브라켓([[ ]])을 사용할 수 있으며, 표현식 내부에서 다른 표현식을 판단하기 위해 사용된다.
```bash
test expression
[ expression ]
```


## test 명령 연산자
### 문자열 테스트

| test 명령 연산자 | 참일 때(True) |
|-------|--------|
| [ string1 == string2 ] | 문자열1과 문자열2의 값이 같은지 테스트한다 |
| [ string1 = string2 ] | 단일 = 기호는 bash 2.x 버전 이상에서 사용할 수 있다 |
| [ string1 != string2 ] | 문자열1과 문자열2의 값이 다른지 테스트한다 |
| [ string ] | 문자열이 null이 아닌지 테스트한다 |
| [ -z string ] | 문자열의 길이가 0인지 테스트한다 |
| [ -n string ] | 문자열의 길이가 0이 아닌지 테스트한다 |

### 논리적 테스트

| test 명령 연산자 | 참일 때(True) |
|-------|--------|
| [ string1 -a string2 ] | 문자열1과 문자열2가 모두 true인지 테스트한다 |
| [ string1 -o string2 ] | 문자열1과 문자열2 둘 중 하나라도 true인지 테스트한다 |
| [ ! string1 ] | 문자열1과 매칭되지 않는지 테스트한다 |

### 논리적 테스트(복합적)

| test 명령 연산자 | 참일 때(True) |
|-------|--------|
| [[ pattern1 && pattern2 ]] | 패턴1과 패턴2 모두 true인지 테스트한다 |
| [[ pattern1 \|\| pattern2 ]] | 패턴1과 패턴2 둘 중 하나라도 true인지 테스트한다 |
| [[ ! pattern ]] | 패턴과 매칭되지 않는지 테스트한다 |

### 정수형 테스트

| test 명령 연산자 | 참일 때(True) |
|-------|--------|
| [ int1 -eq int2 ] | int1과 int2가 같은지 테스트한다 |
| [ int1 -ne int2 ] | int1과 int2가 다른지 테스트한다 |
| [ int1 -gt int2 ] | int1이 int2보다 큰지 테스트한다 |
| [ int1 -ge int2 ] | int1이 int2보다 크거나 같은지 테스트한다 |
| [ int1 -lt int2 ] | int1이 int2보다 작은지 테스트한다 |
| [ int1 -le int2 ] | int1이 int2보다 작거나 같은지 테스트한다 |

### 파일 테스트를 위한 바이너리 연산자

| test 명령 연산자 | 참일 때(True) |
|-------|--------|
| [ file1 -nt file2 ] | file1이 file2보다 최근 파일(수정 일자)인지 테스트한다 |
| [ file1 -ot file2 ] | file1이 file2보다 이전 파일인지 테스트한다 |
| [ file1 -ef file2 ] | file1과 file2가 같은 디바이스 또는 아이노드 번호를 갖는지 테스트한다 |

## Reference
> 김태용의 리눅스 쉘 스크립트 프로그래밍 입문, 김태용, 제이펍, 2009 