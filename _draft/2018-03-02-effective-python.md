---
layout: post
title: "파이썬 코딩의 기술 (Effective Python) 정리"
subtitle: "파이썬 코딩의 기술 (Effective Python) 정리"
categories: development
tags: python
comments: true
---
파이썬 코딩의 기술 책을 읽고 정리한 문서입니다

# 목차

# CHAPTER 1. 파이썬다운 생각
- 파이썬 프로그래머들은 복잡함보다는 단순함을 선호하고 가독성을 극대화하기 위해 명료한 것을 좋아한다

```
>>> import this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

## BETTER WAY 1
사용중인 파이썬 버전 알기

```$ python --version```  
```$ python3 --version```

```
import sys
print(sys.version_info)
print(sys.version)

>>>
sys.version_info(major=3, minor=4, micro=2, releaselevel='final', serial=0)  
3.4.2 (defaul, Mar 2 2018, ~)
```

- 파이썬에는 CPython, Jython, IronPython, PyPy 같은 다양한 런타임이 있음
- 시스템에서 파이썬을 실행하는 명령이 몇 버전인지 확인하기
- 새 파이썬 프로젝트를 시작할 때는 파이썬 3 추천

## BETTER WAY 2
[PEP 8 스타일 가이드](https://www.python.org/dev/peps/pep-0008/)를 따르자

### 화이트스페이스(공백)
- 탭이 아닌 스페이스로 들여쓴다
- 문법적으로 의미있는 들여쓰기는 스페이스 4개
- 한 줄의 문자 길이가 79자 이하여야 한다
- 표현식이 길어서 다음 줄로 일어지면 스페이스 4개 사용
- 파일에서 함수와 클래스는 빈 줄 2개로 구분해야 한다
- 클래스에서 메서드는 빈 줄 1개로 구분해야 한다
- 리스트 인덱스, 함수 호출, 키워드 인수 할당에는 스페이스를 사용하지 않는다
- 변수 할당 앞뒤에 스페이스를 하나만 사용한다

### 명명(naming)
- 함수, 변수, 속성은 **lowercase_underscore** 형식을 따른다
- 보호(protected) 인스턴스 속성은 **\_leading_underscore** 형식을 따른다
- 비공개(private) 인스턴스 속성은 **\__double_leading_underscore** 형식을 따른다
- 클래스와 예외는 CapitalizedWord 형식을 따른다
- 모듈 수준 상수는 ALL_CAPS 형식을 따른다
- 클래스와 인스턴스 메서드는 첫 파라미터의 이름을 self로 지정
- 클래스 메서드에서는 첫 파라미터의 이름을 cls로 지정

### 표현식과 문장
- 긍정 표현식의 부정(if not a is b) 대신에 인라인 부정(if a is not b)를 사용한다
- 길이를 확인(if len(somelist)==0)하여 빈 값([] 또는 '')을 확인하지 않는다. if not somelist를 사용하고 빈 값은 암시적으로 False가 된다고 가정한다
- 비어있지 않은 값([1] 또는 'hi')에도 위와 같은 방식이 적용된다. 값이 비어있지 않으면 if somelist 문이 암시적으로 True가 된다
- 한줄로 된 if문, for와 while 루프, except 복합문을 쓰지 않는다. 이런 문장은 여러 줄로 나눠서 명료하게 작성한다
- 항상 파일의 맨 위에 import문을 놓는다
- 모듈을 임포트할 때는 항상 모듈의 절대 이름을 사용하며 현재 모듈의 경로를 기준으로 상대 경로로 된 이름을 사용하지 않는다. 예를들어 bar 패키지의 foo 모듈을 임포트하려면 from bar import foo라고 해야 한다
- 상대적인 임포트를 해야한다면 명시적인 구문을 써서 from . import foo라고 한다
- 임포트는 '표준 라이브러리 모듈, 서드파티 모듈, 자신이 만든 모듈' 섹션 순으로 구분해야 한다. 각각의 하위 섹션에서는 알파벳 순서로 임포트한다

## BETTER WAY 3
bytes, str, unicode의 차이점을 알자

- 파이썬3는 문자를 bytes와 str 두가지 타입으로 나타냅니다. bytes 인스턴스는 raw 8 비트 값을 저장하며 str 인스턴스는 유니코드 문자를 저장합니다  
- 파이썬2는 문자를 str와 unicode로 나타냅니다. str 인스턴스는 raw 8 비트값을 저장하며 unicode 인스턴스는 유니코드 문자를 저장합니다

- 유니코드 문자를 바이너리 데이터(raw 8 비트)로 표현하는 방법은 많습니다. 가장 일반적인 인코딩은 UTF-8입니다. 중요한 것은 파이썬3의 str 인스턴스와 파이썬 2의 unicode 인스턴스는 연관된 바이너리 인코딩이 없다는 점입니다. 유니코드 문자를 바이너리 데이터로 변환하려면 ```encode``` 메서드를 사용하고, 바이너리 데이터를 유니코드 문자로 변환하려면 ```decode``` 메서드를 사용해야 합니다
- 외부에 제공할 인터페이스에선 유니코드를 인코드하고 디코드해야 합니다. 프로그램의 핵심 부분엔 유니코드 문자 타입(파이썬3은 str, 파이썬2는 unicode)를 사용하고 문자 인코딩에 대해선 어떤 가정을 하지 말아야 합니다. 이렇게 하면 출력 텍스트 인코딩을 유지하면서 다른 텍스트 인코딩을 쉽게 수용할 수 있습니다

그러나 아래와 같은 상황에 부딪힐 수 있습니다

- UTF-8(혹은 다른 인코딩)으로 인코드된 문자인 raw 8비트 값으로 처리하려는 상황
- 인코딩이 없는 유니코드 문자를 처리하는 상황

```
# Python 3
# 먼저 str/bytes를 입력으로 받고 str을 반환하는 메서드
def to_str(bytes_or_str):
    if isinstance(bytes_or_str, bytes):
        value = bytes_or_str.decode('utf-8')
    else:
        value = bytes_or_str
    return value
```

```
# Python 3
# str/bytes를 받고 bytes를 반환하는 메서드
def to_bytes(bytes_or_str):
    if isinstance(bytes_or_str, str):
        value = bytes_or_str.encode('utf-8')
    else:
        value = bytes_or_str
    return value
```