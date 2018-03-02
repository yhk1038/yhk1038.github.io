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
- 시스템에서 파이썬을 실행하는 명령이 몇 버전인지 ㅘㄱ인하기
- 새 파이썬 프로젝트를 시작할 때는 파이썬 3 추천
