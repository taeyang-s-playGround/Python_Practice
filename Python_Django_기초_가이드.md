# Python & Django 기초 완전 정리 가이드

## 📋 목차
1. [Python 기초 문법](#python-기초-문법)
2. [Python 객체지향 프로그래밍](#python-객체지향-프로그래밍)
3. [Django 특화 개념](#django-특화-개념)
4. [실전 예제로 이해하기](#실전-예제로-이해하기)
5. [자주 묻는 질문](#자주-묻는-질문)

---

## Python 기초 문법

### 1. 변수와 데이터 타입

#### 변수란?
변수는 데이터를 저장하는 상자입니다. 이름을 붙여서 나중에 사용할 수 있습니다.

```python
# 변수 선언 및 할당
title = "첫 번째 게시글"  # 문자열(string)
number = 100              # 정수(integer)
price = 99.99             # 실수(float)
is_published = True       # 불린(boolean, True/False)
```

**왜 필요한가?**
- 같은 값을 여러 번 사용할 때 편리합니다
- 값이 바뀌어도 한 곳만 수정하면 됩니다
- 코드가 읽기 쉬워집니다

**예시:**
```python
# ❌ 나쁜 예: 같은 값을 반복
print("안녕하세요, 첫 번째 게시글입니다")
print("첫 번째 게시글을 읽고 있습니다")
print("첫 번째 게시글을 수정합니다")

# ✅ 좋은 예: 변수 사용
title = "첫 번째 게시글"
print(f"안녕하세요, {title}입니다")
print(f"{title}을 읽고 있습니다")
print(f"{title}을 수정합니다")
```

---

### 2. 리스트(List)와 딕셔너리(Dictionary)

#### 리스트 (List)
여러 개의 값을 순서대로 저장하는 컨테이너입니다.

```python
# 리스트 생성
fruits = ["사과", "바나나", "오렌지"]
numbers = [1, 2, 3, 4, 5]

# 리스트 접근 (인덱스는 0부터 시작!)
print(fruits[0])    # "사과" (첫 번째)
print(fruits[1])    # "바나나" (두 번째)
print(fruits[-1])   # "오렌지" (마지막, -1은 뒤에서 첫 번째)

# 리스트에 추가
fruits.append("포도")  # ["사과", "바나나", "오렌지", "포도"]

# 리스트 길이
print(len(fruits))  # 4
```

**Django에서의 사용:**
```python
# settings.py의 INSTALLED_APPS
INSTALLED_APPS = [
    'django.contrib.admin',
    'posts',  # 리스트에 앱 추가
]

# serializers.py의 fields
fields = ["id", "title", "content"]  # 리스트로 필드 지정
```

#### 딕셔너리 (Dictionary)
키(key)와 값(value)의 쌍으로 데이터를 저장합니다.

```python
# 딕셔너리 생성
person = {
    "name": "홍길동",
    "age": 30,
    "city": "서울"
}

# 값 접근
print(person["name"])      # "홍길동"
print(person.get("age"))   # 30 (get 메서드 사용, 키가 없으면 None 반환)

# 값 추가/수정
person["email"] = "hong@example.com"
person["age"] = 31

# 딕셔너리 순회
for key, value in person.items():
    print(f"{key}: {value}")
```

**Django에서의 사용:**
```python
# JSON 데이터 (실제로는 딕셔너리와 유사)
{
    "id": 1,
    "title": "제목",
    "content": "내용"
}

# settings.py의 DATABASES 설정
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

---

### 3. 함수 (Function)

#### 함수란?
특정 작업을 수행하는 코드 블록을 재사용 가능하게 만든 것입니다.

```python
# 함수 정의
def greet(name):
    return f"안녕하세요, {name}님!"

# 함수 호출
message = greet("홍길동")
print(message)  # "안녕하세요, 홍길동님!"
```

**함수의 구성 요소:**
```python
def 함수이름(매개변수1, 매개변수2):
    """함수 설명 (docstring)"""
    # 함수가 수행할 작업
    결과 = 매개변수1 + 매개변수2
    return 결과  # 결과 반환

# 사용
result = 함수이름(10, 20)
print(result)  # 30
```

**Django에서의 사용:**
```python
# models.py
def __str__(self):
    return self.title  # 객체를 문자열로 변환하는 함수

# views.py (일반 함수 기반 뷰)
def post_list(request):
    posts = Post.objects.all()
    return render(request, 'posts/list.html', {'posts': posts})
```

---

### 4. 클래스 (Class)와 객체 (Object)

#### 클래스란?
같은 종류의 객체들을 만들기 위한 설계도(템플릿)입니다.

```python
# 클래스 정의
class Dog:
    # 생성자: 객체가 생성될 때 자동으로 실행
    def __init__(self, name, age):
        self.name = name  # 인스턴스 변수
        self.age = age
    
    # 메서드: 클래스 내부의 함수
    def bark(self):
        return f"{self.name}가 멍멍 짖습니다!"
    
    def get_info(self):
        return f"{self.name}는 {self.age}살입니다"

# 객체 생성 (인스턴스화)
my_dog = Dog("뽀삐", 3)
your_dog = Dog("멍멍이", 5)

# 메서드 호출
print(my_dog.bark())      # "뽀삐가 멍멍 짖습니다!"
print(your_dog.get_info())  # "멍멍이는 5살입니다"
```

**핵심 개념:**
- **클래스**: 설계도 (Dog)
- **객체/인스턴스**: 설계도로 만든 실제 물건 (my_dog, your_dog)
- **속성(Attribute)**: 객체가 가진 데이터 (name, age)
- **메서드(Method)**: 객체가 할 수 있는 행동 (bark, get_info)
- **self**: 객체 자신을 가리키는 키워드

**Django에서의 사용:**
```python
# models.py - Post 클래스
class Post(models.Model):  # models.Model을 상속
    title = models.CharField(max_length=100)  # 속성
    
    def __str__(self):  # 메서드
        return self.title

# 객체 생성
post = Post(title="제목", content="내용")
post.save()  # 데이터베이스에 저장
```

---

### 5. 상속 (Inheritance)

#### 상속이란?
기존 클래스의 기능을 물려받아 새로운 클래스를 만드는 것입니다.

```python
# 부모 클래스 (기본 클래스)
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        return "동물이 소리를 냅니다"

# 자식 클래스 (상속받는 클래스)
class Dog(Animal):  # Animal을 상속
    def speak(self):  # 메서드 오버라이드 (재정의)
        return f"{self.name}가 멍멍 짖습니다!"

class Cat(Animal):  # Animal을 상속
    def speak(self):  # 메서드 오버라이드
        return f"{self.name}가 야옹 웁니다!"

# 사용
dog = Dog("뽀삐")
cat = Cat("나비")
print(dog.speak())  # "뽀삐가 멍멍 짖습니다!"
print(cat.speak())  # "나비가 야옹 웁니다!"
```

**왜 상속을 사용하나요?**
- 코드 재사용: 공통 기능을 한 번만 작성
- 확장성: 부모 클래스의 기능을 그대로 사용하면서 새로운 기능 추가
- 유지보수: 부모 클래스 수정 시 모든 자식 클래스에 자동 반영

**Django에서의 사용:**
```python
# models.Model을 상속받아 데이터베이스 기능 사용
class Post(models.Model):
    # models.Model의 모든 기능을 자동으로 상속받음
    # save(), delete(), objects 등이 자동으로 사용 가능
    title = models.CharField(max_length=100)

# viewsets.ModelViewSet을 상속받아 CRUD 기능 사용
class PostViewSet(viewsets.ModelViewSet):
    # ModelViewSet의 list(), create(), update() 등을 자동으로 사용
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

---

### 6. 모듈과 Import

#### 모듈이란?
다른 파일에 작성된 코드를 가져와서 사용하는 것입니다.

```python
# math.py (가정)
def add(a, b):
    return a + b

def multiply(a, b):
    return a * b

# main.py
# 방법 1: 전체 모듈 import
import math
result = math.add(1, 2)

# 방법 2: 특정 함수만 import
from math import add, multiply
result = add(1, 2)

# 방법 3: 별칭 사용
from math import add as 더하기
result = 더하기(1, 2)

# 방법 4: 모든 함수 import (비권장)
from math import *
result = add(1, 2)
```

**Django에서의 사용:**
```python
# models.py
from django.db import models  # Django의 models 모듈에서 models 클래스 import

# serializers.py
from rest_framework import serializers  # DRF의 serializers 모듈 import
from .models import Post  # 같은 앱의 models.py에서 Post import

# views.py
from rest_framework import viewsets
from .models import Post  # 상대 경로 import
from .serializers import PostSerializer
```

**상대 경로 vs 절대 경로:**
```python
# 상대 경로 (같은 앱 내에서)
from .models import Post  # .은 현재 디렉토리(앱)
from .serializers import PostSerializer

# 절대 경로 (다른 앱에서)
from posts.models import Post
from posts.serializers import PostSerializer
```

---

## Python 객체지향 프로그래밍

### 1. 특수 메서드 (Magic Methods / Dunder Methods)

#### __init__ 메서드
객체가 생성될 때 자동으로 호출되는 생성자입니다.

```python
class Post:
    def __init__(self, title, content):
        # 객체 생성 시 자동 실행
        self.title = title
        self.content = content
        print("Post 객체가 생성되었습니다!")

# 객체 생성 시 __init__ 자동 호출
post = Post("제목", "내용")
# 출력: "Post 객체가 생성되었습니다!"
```

**Django에서:**
```python
# Django 모델은 __init__을 직접 정의하지 않아도
# models.Model이 자동으로 처리해줍니다
post = Post(title="제목", content="내용")
```

#### __str__ 메서드
객체를 문자열로 변환할 때 호출됩니다.

```python
class Post:
    def __init__(self, title):
        self.title = title
    
    def __str__(self):
        return self.title  # 문자열로 변환할 때 이 값 반환

post = Post("첫 번째 게시글")
print(post)  # "첫 번째 게시글" 출력
# print()는 내부적으로 __str__()을 호출합니다
```

**실제 사용 예시:**
```python
# ❌ __str__ 없을 때
class Post:
    def __init__(self, title):
        self.title = title

post = Post("제목")
print(post)  # <__main__.Post object at 0x...> (의미 없는 정보)

# ✅ __str__ 있을 때
class Post:
    def __init__(self, title):
        self.title = title
    
    def __str__(self):
        return self.title

post = Post("제목")
print(post)  # "제목" (의미 있는 정보!)
```

**Django에서의 활용:**
```python
# models.py
class Post(models.Model):
    title = models.CharField(max_length=100)
    
    def __str__(self):
        return self.title

# Django 쉘에서
>>> post = Post.objects.get(id=1)
>>> print(post)
첫 번째 게시글  # __str__이 반환한 값

# Django 관리자 페이지에서도
# Post 목록에 "첫 번째 게시글"로 표시됨
```

---

### 2. 클래스 변수 vs 인스턴스 변수

#### 인스턴스 변수
각 객체마다 다른 값을 가집니다.

```python
class Post:
    def __init__(self, title):
        self.title = title  # 인스턴스 변수 (각 객체마다 다름)

post1 = Post("제목1")
post2 = Post("제목2")
print(post1.title)  # "제목1"
print(post2.title)  # "제목2" (다른 값)
```

#### 클래스 변수
모든 객체가 공유하는 변수입니다.

```python
class Post:
    count = 0  # 클래스 변수 (모든 객체가 공유)
    
    def __init__(self, title):
        self.title = title  # 인스턴스 변수
        Post.count += 1  # 클래스 변수 증가

post1 = Post("제목1")
print(Post.count)  # 1

post2 = Post("제목2")
print(Post.count)  # 2 (공유됨)
print(post1.count)  # 2 (같은 값)
print(post2.count)  # 2 (같은 값)
```

**Django에서의 사용:**
```python
# views.py
class PostViewSet(viewsets.ModelViewSet):
    # 클래스 변수: 모든 메서드에서 공유
    queryset = Post.objects.all()  # 클래스 변수
    serializer_class = PostSerializer  # 클래스 변수
    
    # 인스턴스 변수는 메서드 내에서 self.변수명으로 사용
    def list(self, request):
        # self.queryset은 클래스 변수 queryset을 참조
        posts = self.queryset
        return Response(...)
```

---

### 3. 메서드의 종류

#### 인스턴스 메서드
객체(인스턴스)를 통해 호출하는 메서드입니다.

```python
class Post:
    def __init__(self, title):
        self.title = title
    
    def get_title(self):  # 인스턴스 메서드
        return self.title  # self를 통해 인스턴스 변수 접근

post = Post("제목")
post.get_title()  # 인스턴스를 통해 호출
```

#### 클래스 메서드
클래스를 통해 호출하는 메서드입니다.

```python
class Post:
    count = 0
    
    @classmethod  # 데코레이터
    def get_count(cls):  # cls는 클래스 자체
        return cls.count

Post.get_count()  # 클래스를 통해 호출
```

#### 정적 메서드
클래스나 인스턴스와 무관하게 동작하는 메서드입니다.

```python
class Post:
    @staticmethod  # 데코레이터
    def format_date(date):
        return date.strftime("%Y-%m-%d")

Post.format_date(some_date)  # 클래스를 통해 호출
post.format_date(some_date)  # 인스턴스를 통해 호출 (둘 다 가능)
```

---

## Django 특화 개념

### 1. Meta 클래스

#### Meta 클래스란?
클래스에 대한 메타데이터(설정 정보)를 담는 내부 클래스입니다.

```python
class Post(models.Model):
    title = models.CharField(max_length=100)
    
    class Meta:  # 내부 클래스
        ordering = ['-created_at']  # 기본 정렬 순서
        verbose_name = '게시글'  # 관리자 페이지에서 보이는 이름
        verbose_name_plural = '게시글들'  # 복수형 이름

# Meta는 클래스의 설정을 담는 곳입니다
# 실제 동작은 models.Model이 Meta 정보를 읽어서 처리합니다
```

**왜 Meta 클래스를 사용하나요?**
- **설정 분리**: 클래스의 동작과 설정을 명확히 구분
- **Django 관례**: Django에서 표준으로 사용하는 패턴
- **자동 처리**: Meta 정보를 Django가 자동으로 읽어서 처리

**실제 예시:**
```python
# serializers.py
class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post  # 어떤 모델을 사용할지
        fields = ["id", "title", "content"]  # 어떤 필드를 포함할지
        # 이 정보를 DRF가 읽어서 자동으로 Serializer 생성
```

---

### 2. QuerySet

#### QuerySet이란?
데이터베이스에서 데이터를 가져오고 조작하는 Django의 도구입니다.

```python
# QuerySet은 지연 평가(Lazy Evaluation)됩니다
# 실제로 데이터를 가져오는 것은 나중에 일어납니다

# 1. QuerySet 생성 (아직 DB 조회 안 함)
posts = Post.objects.all()

# 2. 실제 사용할 때 DB 조회
for post in posts:  # 이때 실제로 DB에서 데이터 가져옴
    print(post.title)

# 또는
post_list = list(posts)  # 리스트로 변환할 때 DB 조회
```

**QuerySet 메서드들:**
```python
# 필터링
Post.objects.filter(title="제목")  # 조건에 맞는 것만
Post.objects.exclude(title="제목")  # 조건에 맞지 않는 것만

# 정렬
Post.objects.all().order_by("-created_at")  # 최신순
Post.objects.all().order_by("title")  # 제목순

# 개수
Post.objects.count()  # 전체 개수
Post.objects.filter(title="제목").count()  # 조건에 맞는 개수

# 하나만 가져오기
Post.objects.get(id=1)  # 정확히 하나 (없거나 여러 개면 에러)
Post.objects.first()  # 첫 번째 (없으면 None)
Post.objects.last()  # 마지막 (없으면 None)
```

**체이닝 (Chaining):**
```python
# 여러 메서드를 연속으로 사용 가능
posts = Post.objects.all()  # 모든 Post
    .filter(title__contains="Django")  # 제목에 "Django" 포함
    .exclude(id=1)  # id가 1인 것 제외
    .order_by("-created_at")  # 최신순 정렬
    .filter(content__isnull=False)  # content가 비어있지 않은 것만
```

---

### 3. 상대 경로 Import

#### 상대 경로란?
현재 파일의 위치를 기준으로 다른 파일을 찾는 방법입니다.

```python
# 프로젝트 구조
posts/
├── models.py
├── serializers.py
└── views.py

# serializers.py에서
from .models import Post  # .은 현재 디렉토리(posts)
# = from posts.models import Post

# views.py에서
from .models import Post  # posts/models.py
from .serializers import PostSerializer  # posts/serializers.py

# 상위 디렉토리로 가려면
from ..other_app.models import OtherModel  # ..은 상위 디렉토리
```

**왜 상대 경로를 사용하나요?**
- 앱 이름이 바뀌어도 코드 수정이 적음
- 같은 앱 내에서 import할 때 간결함
- Django의 권장 방식

---

## 실전 예제로 이해하기

### 예제 1: 간단한 클래스 만들기

```python
# book.py
class Book:
    # 클래스 변수
    total_books = 0
    
    # 생성자
    def __init__(self, title, author, price):
        # 인스턴스 변수
        self.title = title
        self.author = author
        self.price = price
        Book.total_books += 1  # 클래스 변수 증가
    
    # 인스턴스 메서드
    def get_info(self):
        return f"{self.title} by {self.author}, ${self.price}"
    
    # 특수 메서드
    def __str__(self):
        return self.title
    
    # 클래스 메서드
    @classmethod
    def get_total(cls):
        return cls.total_books

# 사용
book1 = Book("파이썬 기초", "홍길동", 29.99)
book2 = Book("Django 완전정복", "김철수", 39.99)

print(book1.get_info())  # "파이썬 기초 by 홍길동, $29.99"
print(book2)  # "Django 완전정복" (__str__ 호출)
print(Book.get_total())  # 2
```

### 예제 2: Django 모델과 비교

```python
# Django 모델
class Post(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.title
    
    class Meta:
        ordering = ['-created_at']

# 일반 Python 클래스로 비슷하게 만들면
class Post:
    def __init__(self, title, content, created_at=None):
        self.title = title
        self.content = content
        self.created_at = created_at or datetime.now()
    
    def __str__(self):
        return self.title
    
    # Django는 자동으로 제공하지만, 일반 클래스는 수동으로
    def save(self):
        # 데이터베이스에 저장하는 로직
        pass

# 차이점:
# - Django 모델은 models.Model을 상속받아 자동으로 많은 기능 제공
# - save(), delete(), objects 등이 자동으로 사용 가능
# - 데이터베이스와 자동으로 연동
```

### 예제 3: Serializer 이해하기

```python
# Serializer를 일반 Python으로 비유하면
class PostSerializer:
    def __init__(self, post=None, data=None):
        self.post = post  # Python 객체
        self.data_dict = data  # 딕셔너리 (JSON과 유사)
    
    # 직렬화: Python 객체 → 딕셔너리
    def to_dict(self):
        if self.post:
            return {
                "id": self.post.id,
                "title": self.post.title,
                "content": self.post.content,
                "created_at": self.post.created_at.isoformat()
            }
        return None
    
    # 역직렬화: 딕셔너리 → Python 객체
    def to_object(self):
        if self.data_dict:
            return Post(
                title=self.data_dict["title"],
                content=self.data_dict["content"]
            )
        return None

# 사용
post = Post(title="제목", content="내용")
serializer = PostSerializer(post=post)
json_data = serializer.to_dict()
# {"id": 1, "title": "제목", "content": "내용", ...}

# 역직렬화
json_data = {"title": "새 제목", "content": "새 내용"}
serializer = PostSerializer(data=json_data)
new_post = serializer.to_object()
```

---

## 자주 묻는 질문

### Q1: self는 뭔가요?

**A:** `self`는 객체 자신을 가리키는 키워드입니다.

```python
class Post:
    def __init__(self, title):
        self.title = title  # self.title은 이 객체의 title 속성
    
    def get_title(self):
        return self.title  # 이 객체의 title을 반환

post = Post("제목")
# post.get_title()을 호출하면
# 내부적으로 get_title(post)처럼 작동
# self는 post를 가리킴
```

**비유:**
- "나"라는 단어처럼, 각 객체가 자신을 지칭하는 말
- `self.title` = "이 객체의 title"

### Q2: 왜 클래스를 사용하나요?

**A:** 관련된 데이터와 기능을 하나로 묶어서 관리하기 위해서입니다.

```python
# ❌ 클래스 없이 (구조화되지 않음)
title1 = "제목1"
content1 = "내용1"
created_at1 = "2024-01-01"

title2 = "제목2"
content2 = "내용2"
created_at2 = "2024-01-02"

# ✅ 클래스 사용 (구조화됨)
class Post:
    def __init__(self, title, content, created_at):
        self.title = title
        self.content = content
        self.created_at = created_at

post1 = Post("제목1", "내용1", "2024-01-01")
post2 = Post("제목2", "내용2", "2024-01-02")
```

### Q3: 상속은 언제 사용하나요?

**A:** 기존 기능을 재사용하면서 새로운 기능을 추가할 때 사용합니다.

```python
# 공통 기능이 있는 경우
class Animal:
    def eat(self):
        return "먹는다"

# 각 동물마다 다른 소리를 내지만, 모두 먹는 기능은 공통
class Dog(Animal):
    def speak(self):
        return "멍멍"

class Cat(Animal):
    def speak(self):
        return "야옹"

# Dog와 Cat 모두 eat() 메서드를 사용할 수 있음
dog = Dog()
dog.eat()  # "먹는다" (상속받은 메서드)
dog.speak()  # "멍멍" (자신만의 메서드)
```

### Q4: QuerySet은 언제 실제로 DB를 조회하나요?

**A:** QuerySet을 실제로 사용할 때 조회합니다 (지연 평가).

```python
# 1단계: QuerySet 생성 (아직 DB 조회 안 함)
posts = Post.objects.all()
print(posts)  # <QuerySet [Post: ...]> (아직 조회 안 함)

# 2단계: 실제 사용 (이때 DB 조회)
for post in posts:  # 이 순간 DB 조회!
    print(post.title)

# 또는
list(posts)  # 리스트로 변환할 때 조회
posts[0]  # 인덱스로 접근할 때 조회
len(posts)  # 개수를 셀 때 조회
```

### Q5: Meta 클래스는 왜 필요한가요?

**A:** 클래스의 설정 정보를 명확하게 분리하기 위해서입니다.

```python
# ❌ Meta 없이 (설정이 흩어짐)
class Post(models.Model):
    title = models.CharField(max_length=100)
    # 정렬 순서는 어디에?
    # 관리자 페이지 이름은 어디에?

# ✅ Meta 사용 (설정이 명확함)
class Post(models.Model):
    title = models.CharField(max_length=100)
    
    class Meta:
        ordering = ['-created_at']  # 여기에 정렬 순서
        verbose_name = '게시글'  # 여기에 관리자 페이지 이름
```

---

## 마무리

이 가이드를 통해 Python과 Django의 기본 개념을 이해하셨을 것입니다.

### 핵심 정리

1. **변수**: 데이터를 저장하는 상자
2. **함수**: 재사용 가능한 코드 블록
3. **클래스**: 객체를 만드는 설계도
4. **상속**: 기존 클래스의 기능을 물려받기
5. **모듈**: 다른 파일의 코드를 가져오기
6. **특수 메서드**: `__init__`, `__str__` 등 자동 호출 메서드
7. **QuerySet**: 데이터베이스 조작 도구
8. **Meta 클래스**: 클래스 설정 정보

### 다음 단계

- 실제 프로젝트에서 코드 작성해보기
- 에러 메시지를 읽고 이해하기
- Django 공식 문서 읽기
- 다른 사람의 코드 읽어보기

꾸준한 연습이 실력 향상의 지름길입니다! 🚀

