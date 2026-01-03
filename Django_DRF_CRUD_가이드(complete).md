# Django Rest Framework CRUD API 완전 분석 가이드

## 📋 목차
1. [개요](#개요)
2. [프로젝트 구조](#프로젝트-구조)
3. [파일별 상세 분석](#파일별-상세-분석)
4. [코드 작성 흐름 및 플로우](#코드-작성-흐름-및-플로우)
5. [API 엔드포인트 사용법](#api-엔드포인트-사용법)
6. [핵심 개념 정리](#핵심-개념-정리)

---

## 개요

이 프로젝트는 **Django Rest Framework(DRF)**를 사용하여 Post(게시글)의 CRUD(Create, Read, Update, Delete) 기능을 구현한 REST API입니다.

### CRUD란?
- **C**reate: 생성 (새 게시글 작성)
- **R**ead: 읽기 (게시글 조회)
- **U**pdate: 수정 (게시글 수정)
- **D**elete: 삭제 (게시글 삭제)

### Django Rest Framework란?
Django에서 RESTful API를 쉽게 만들 수 있도록 도와주는 강력한 라이브러리입니다. 복잡한 API 로직을 간단하게 구현할 수 있게 해줍니다.

---

## 프로젝트 구조

```
mysite/
├── mysite/              # 프로젝트 설정 폴더
│   ├── settings.py      # 프로젝트 전체 설정
│   └── urls.py          # 프로젝트 메인 URL 설정
└── posts/               # posts 앱 폴더
    ├── models.py        # 데이터베이스 모델 정의
    ├── serializers.py   # 데이터 직렬화/역직렬화
    ├── views.py         # API 뷰 로직
    └── urls.py          # posts 앱의 URL 설정
```

---

## 파일별 상세 분석

### 1. models.py - 데이터베이스 모델 정의

**위치**: `mysite/posts/models.py`

```python
from django.db import models


class Post(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

#### 코드 분석

**1줄**: `from django.db import models`
- Django의 데이터베이스 모델 기능을 사용하기 위해 import합니다.
- `models`는 데이터베이스 테이블을 Python 클래스로 정의할 수 있게 해주는 모듈입니다.

**4줄**: `class Post(models.Model):`
- `Post`라는 이름의 모델 클래스를 정의합니다.
- `models.Model`을 상속받아야 Django가 이 클래스를 데이터베이스 테이블로 변환합니다.
- 이 클래스는 데이터베이스의 "posts" 테이블이 됩니다.

**5줄**: `title = models.CharField(max_length=100)`
- 게시글 제목을 저장하는 필드입니다.
- `CharField`: 문자열을 저장하는 필드 타입
- `max_length=100`: 최대 100자까지 입력 가능

**6줄**: `content = models.TextField()`
- 게시글 내용을 저장하는 필드입니다.
- `TextField`: 긴 텍스트를 저장할 수 있는 필드 타입 (길이 제한 없음)

**7줄**: `created_at = models.DateTimeField(auto_now_add=True)`
- 게시글 생성 시간을 저장하는 필드입니다.
- `DateTimeField`: 날짜와 시간을 저장하는 필드 타입
- `auto_now_add=True`: 객체가 처음 생성될 때 자동으로 현재 시간을 저장

**9-10줄**: `def __str__(self): return self.title`
- Python의 특수 메서드(매직 메서드 또는 던더 메서드)입니다.
- `__str__`는 객체를 문자열로 표현할 때 자동으로 호출되는 메서드입니다.
- 이 메서드가 없으면 객체를 출력할 때 `<Post object (1)>` 같은 의미 없는 문자열이 나옵니다.
- 이 메서드를 정의하면 객체를 출력할 때 제목이 보입니다.

#### __str__ 메서드가 사용되는 곳

**1. Django 관리자 페이지 (Admin)**
```python
# admin.py에서 Post 모델을 등록했다면
# 관리자 페이지에서 Post 목록을 볼 때:
# ❌ __str__ 없으면: Post object (1), Post object (2)
# ✅ __str__ 있으면: "첫 번째 게시글", "두 번째 게시글"
```

**2. Python 쉘에서 객체 출력**
```python
# Django 쉘에서
python manage.py shell

>>> from posts.models import Post
>>> post = Post.objects.get(id=1)
>>> print(post)
# ❌ __str__ 없으면: <Post: Post object (1)>
# ✅ __str__ 있으면: 첫 번째 게시글

>>> post  # 그냥 입력해도 자동으로 __str__ 호출
# ❌ __str__ 없으면: <Post: Post object (1)>
# ✅ __str__ 있으면: 첫 번째 게시글
```

**3. 로깅 및 디버깅**
```python
# 코드에서
post = Post.objects.get(id=1)
print(f"현재 게시글: {post}")  
# ✅ __str__ 있으면: "현재 게시글: 첫 번째 게시글"
# ❌ __str__ 없으면: "현재 게시글: <Post object (1)>"
```

**4. 템플릿에서 (HTML)**
```django
<!-- Django 템플릿에서 -->
{{ post }}  
<!-- ✅ __str__ 있으면: "첫 번째 게시글" -->
<!-- ❌ __str__ 없으면: "Post object (1)" -->
```

#### 왜 필요한가?
- **가독성**: 객체를 출력할 때 의미 있는 정보를 볼 수 있습니다.
- **디버깅**: 개발 중에 어떤 객체인지 쉽게 파악할 수 있습니다.
- **사용자 경험**: 관리자 페이지에서 직관적으로 확인할 수 있습니다.
- **Python 관례**: Python에서 객체를 표현하는 표준 방법입니다.

#### 실제 비교 예시

**__str__ 메서드가 없을 때:**
```python
>>> post = Post.objects.get(id=1)
>>> print(post)
<Post: Post object (1)>  # 의미 없는 정보
```

**__str__ 메서드가 있을 때:**
```python
>>> post = Post.objects.get(id=1)
>>> print(post)
첫 번째 게시글  # 의미 있는 정보!
```

#### 왜 필요한가? (모델 전체)
모델은 데이터베이스의 구조를 정의합니다. 이 파일을 작성한 후 마이그레이션을 실행하면 실제 데이터베이스에 테이블이 생성됩니다.

**모델의 역할:**
1. **데이터베이스 스키마 정의**: 어떤 필드가 있고, 어떤 타입인지 정의
2. **데이터 검증**: 필드 타입과 제약 조건으로 데이터 무결성 보장
3. **ORM 기능 제공**: Python 코드로 데이터베이스 조작 가능
4. **관계 정의**: 다른 모델과의 관계(외래키, 다대다 등) 정의 가능

---

### 2. serializers.py - 데이터 직렬화

**위치**: `mysite/posts/serializers.py`

```python
from rest_framework import serializers
from .models import Post


class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = ["id", "title", "content", "created_at"]
```

#### 코드 분석

**1줄**: `from rest_framework import serializers`
- Django Rest Framework의 serializers 모듈을 import합니다.

**2줄**: `from .models import Post`
- 같은 앱(posts)의 models.py에서 Post 모델을 import합니다.
- `.models`는 `posts.models`를 의미합니다 (상대 경로).
- `.`은 현재 디렉토리(현재 앱)를 의미합니다.
- 절대 경로로는 `from posts.models import Post`로도 가능하지만, 상대 경로가 더 유연합니다.

#### 왜 상대 경로를 사용하나요?
- 앱 이름이 바뀌어도 코드 수정이 적습니다.
- 같은 앱 내에서 import할 때 간결합니다.
- Django의 권장 방식입니다.

**5줄**: `class PostSerializer(serializers.ModelSerializer):`
- `PostSerializer` 클래스를 정의합니다.
- `ModelSerializer`를 상속받으면 모델의 필드를 자동으로 직렬화할 수 있습니다.

**6-8줄**: `class Meta:`
- Django/DRF에서 설정을 정의하는 내부 클래스입니다.
- `Meta`는 "메타데이터(metadata)"의 줄임말로, "데이터에 대한 데이터"를 의미합니다.
- 즉, 클래스 자체에 대한 설정 정보를 담는 곳입니다.
- `model = Post`: 어떤 모델을 직렬화할지 지정
- `fields = [...]`: API 응답에 포함할 필드들을 지정
  - `id`: 자동 생성되는 고유 번호 (Django가 자동으로 추가)
  - `title`, `content`, `created_at`: Post 모델의 필드들

#### 왜 Meta 클래스가 필요한가?
- **설정 분리**: 클래스의 동작과 설정을 명확히 구분
- **Django 관례**: Django/DRF에서 표준으로 사용하는 패턴
- **자동 처리**: Meta 정보를 통해 DRF가 자동으로 많은 기능을 제공

#### fields 옵션의 다른 방법들
```python
# 방법 1: 특정 필드만 포함
fields = ["id", "title", "content", "created_at"]

# 방법 2: 모든 필드 포함
fields = "__all__"

# 방법 3: 특정 필드 제외
exclude = ["created_at"]  # created_at만 제외하고 나머지 모두 포함
```

#### 직렬화(Serialization)란?
- **직렬화**: Python 객체 → JSON 형식으로 변환 (서버 → 클라이언트)
- **역직렬화**: JSON 형식 → Python 객체로 변환 (클라이언트 → 서버)

예시:
```python
# Python 객체
post = Post(title="안녕하세요", content="첫 게시글입니다")

# 직렬화 후 JSON
{
    "id": 1,
    "title": "안녕하세요",
    "content": "첫 게시글입니다",
    "created_at": "2024-01-01T12:00:00Z"
}
```

#### 왜 필요한가?
웹 API는 JSON 형식으로 데이터를 주고받습니다. Serializer는 Python 객체와 JSON 사이를 변환해주는 역할을 합니다.

**Serializer가 없다면:**
```python
# 수동으로 변환해야 함 (매우 번거로움)
post = Post.objects.get(id=1)
json_data = {
    "id": post.id,
    "title": post.title,
    "content": post.content,
    "created_at": post.created_at.isoformat()  # 날짜 형식 변환도 필요
}
```

**Serializer가 있으면:**
```python
# 자동으로 변환
post = Post.objects.get(id=1)
serializer = PostSerializer(post)
json_data = serializer.data  # 끝!
```

**추가 기능:**
- **데이터 검증**: 잘못된 데이터 입력 시 자동으로 에러 반환
- **타입 변환**: 날짜, 숫자 등을 자동으로 적절한 형식으로 변환
- **보안**: 민감한 필드 제외 가능

---

### 3. views.py - API 뷰 로직

**위치**: `mysite/posts/views.py`

```python
from rest_framework import viewsets
from .models import Post
from .serializers import PostSerializer


class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all().order_by("-created_at")
    serializer_class = PostSerializer
```

#### 코드 분석

**1줄**: `from rest_framework import viewsets`
- DRF의 viewsets 모듈을 import합니다.
- ViewSet은 여러 HTTP 메서드(GET, POST, PUT, DELETE 등)를 하나의 클래스에서 처리할 수 있게 해줍니다.

**2줄**: `from .models import Post`
- Post 모델을 import합니다.

**3줄**: `from .serializers import PostSerializer`
- PostSerializer를 import합니다.

**6줄**: `class PostViewSet(viewsets.ModelViewSet):`
- `PostViewSet` 클래스를 정의합니다.
- `ModelViewSet`을 상속받으면 CRUD 기능이 자동으로 제공됩니다!
  - `list()`: GET /posts/ - 전체 목록 조회
  - `create()`: POST /posts/ - 새 게시글 생성
  - `retrieve()`: GET /posts/{id}/ - 특정 게시글 조회
  - `update()`: PUT /posts/{id}/ - 전체 수정
  - `partial_update()`: PATCH /posts/{id}/ - 부분 수정
  - `destroy()`: DELETE /posts/{id}/ - 삭제

**7줄**: `queryset = Post.objects.all().order_by("-created_at")`
- 이 ViewSet이 다룰 데이터를 지정합니다.
- `Post.objects.all()`: 모든 Post 객체를 가져옵니다.
  - `Post.objects`: Post 모델의 데이터베이스 매니저
  - `.all()`: 모든 레코드를 가져오는 메서드
- `.order_by("-created_at")`: 생성일 기준 내림차순 정렬 (최신순)
  - `-`는 내림차순을 의미합니다 (없으면 오름차순)
  - `created_at`은 모델에서 정의한 필드명

**왜 queryset을 클래스 변수로 정의하나요?**
- ViewSet의 모든 메서드(list, retrieve, update 등)에서 공통으로 사용
- 한 곳에서 관리하면 유지보수가 쉬움
- 필요시 메서드에서 오버라이드(재정의) 가능

**8줄**: `serializer_class = PostSerializer`
- 어떤 Serializer를 사용할지 지정합니다.
- 이 Serializer가 데이터 변환을 담당합니다.
- 모든 CRUD 작업에서 이 Serializer를 사용합니다.

#### 왜 필요한가?
ViewSet은 실제 API 요청을 처리하는 로직입니다. 클라이언트가 요청을 보내면 ViewSet이 적절한 메서드를 실행하여 응답을 반환합니다.

---

### 4. urls.py (posts 앱) - URL 라우팅

**위치**: `mysite/posts/urls.py`

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import PostViewSet

router = DefaultRouter()
router.register(r"posts", PostViewSet, basename="post")

urlpatterns = [
    path("", include(router.urls)),
]
```

#### 코드 분석

**1줄**: `from django.urls import path, include`
- URL 패턴을 정의하기 위한 함수들을 import합니다.
- `path`: URL 경로를 정의
- `include`: 다른 URL 설정 파일을 포함

**2줄**: `from rest_framework.routers import DefaultRouter`
- DRF의 DefaultRouter를 import합니다.
- Router는 ViewSet을 자동으로 URL에 연결해주는 도구입니다.

**3줄**: `from .views import PostViewSet`
- PostViewSet을 import합니다.

**5줄**: `router = DefaultRouter()`
- DefaultRouter 인스턴스를 생성합니다.
- 이 Router가 ViewSet을 URL 패턴으로 자동 변환해줍니다.

**6줄**: `router.register(r"posts", PostViewSet, basename="post")`
- ViewSet을 Router에 등록합니다.
- `r"posts"`: URL 경로 이름 (예: `/posts/`)
  - `r`은 raw string을 의미합니다 (백슬래시 이스케이프 처리 안 함)
  - 여기서는 사실 `r`이 없어도 되지만, Django 관례상 사용
- `PostViewSet`: 연결할 ViewSet
- `basename="post"`: URL 이름의 기본 이름
  - URL reverse 기능에서 사용 (예: `reverse('post-list')`)

**이 한 줄로 다음 URL들이 자동 생성됩니다:**
- `GET /posts/` → `PostViewSet.list()` - 전체 목록 조회
- `POST /posts/` → `PostViewSet.create()` - 새 게시글 생성
- `GET /posts/{id}/` → `PostViewSet.retrieve()` - 특정 게시글 조회
- `PUT /posts/{id}/` → `PostViewSet.update()` - 전체 수정
- `PATCH /posts/{id}/` → `PostViewSet.partial_update()` - 부분 수정
- `DELETE /posts/{id}/` → `PostViewSet.destroy()` - 삭제

**Router를 사용하지 않으면?**
각 URL을 수동으로 하나씩 작성해야 하고, 코드가 길어지고 실수할 가능성이 높습니다.

**8-10줄**: `urlpatterns = [...]`
- Django가 인식하는 URL 패턴 리스트입니다.
- `path("", include(router.urls))`: Router가 생성한 모든 URL을 포함
- 빈 문자열 `""`은 이 앱의 루트 경로를 의미합니다.

#### 왜 필요한가?
URL 설정은 어떤 URL 요청이 어떤 ViewSet 메서드로 연결될지 결정합니다. Router를 사용하면 수동으로 하나씩 작성할 필요 없이 자동으로 생성됩니다.

---

### 5. urls.py (프로젝트 메인) - 프로젝트 전체 URL 설정

**위치**: `mysite/mysite/urls.py`

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('posts.urls')),
]
```

#### 코드 분석

**1줄**: `from django.contrib import admin`
- Django 관리자 페이지를 위한 모듈입니다.

**2줄**: `from django.urls import path, include`
- URL 설정을 위한 함수들을 import합니다.

**4-7줄**: `urlpatterns = [...]`
- 프로젝트 전체의 URL 패턴을 정의합니다.

**5줄**: `path('admin/', admin.site.urls)`
- `/admin/` 경로로 접근하면 Django 관리자 페이지가 열립니다.

**6줄**: `path('api/', include('posts.urls'))`
- `/api/` 경로로 접근하면 `posts.urls`의 URL 패턴들이 포함됩니다.
- 따라서 실제 API 경로는 `/api/posts/`가 됩니다.

#### URL 경로 흐름
```
클라이언트 요청: GET /api/posts/
    ↓
mysite/urls.py: 'api/' → posts.urls로 전달
    ↓
posts/urls.py: '' → router.urls로 전달
    ↓
Router: 'posts/' → PostViewSet.list() 실행
    ↓
응답 반환
```

---

### 6. settings.py - 프로젝트 설정

**위치**: `mysite/mysite/settings.py`

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',  # ← 추가된 부분

    'posts',  # ← 추가된 부분
]
```

#### 코드 분석

**INSTALLED_APPS**: Django 프로젝트에서 사용할 앱들의 목록입니다.

**'rest_framework'**: 
- Django Rest Framework를 사용하기 위해 반드시 추가해야 합니다.
- 이 앱이 없으면 serializers, viewsets 등을 사용할 수 없습니다.

**'posts'**: 
- 우리가 만든 posts 앱을 등록합니다.
- 등록하지 않으면 Django가 이 앱을 인식하지 못합니다.

#### 왜 필요한가?
Django는 `INSTALLED_APPS`에 등록된 앱만 활성화합니다. 새로 만든 앱이나 라이브러리는 여기에 추가해야 사용할 수 있습니다.

---

## 코드 작성 흐름 및 플로우

### 1. 전체 개발 흐름

```
1. 모델 정의 (models.py)
   ↓
2. 마이그레이션 생성 및 적용
   ↓
3. Serializer 작성 (serializers.py)
   ↓
4. ViewSet 작성 (views.py)
   ↓
5. URL 설정 (urls.py)
   ↓
6. settings.py에 앱 등록
   ↓
7. 테스트 및 실행
```

### 2. 요청 처리 플로우 (Request Flow)

```
[클라이언트] 
    ↓ HTTP 요청 (예: GET /api/posts/1/)
    
[mysite/urls.py]
    ↓ 'api/' 경로 매칭
    ↓ include('posts.urls')로 전달
    
[posts/urls.py]
    ↓ Router가 'posts/{id}/' 경로 인식
    ↓ PostViewSet.retrieve() 메서드 호출
    
[views.py - PostViewSet]
    ↓ queryset에서 id=1인 Post 객체 조회
    ↓ PostSerializer로 직렬화
    
[serializers.py - PostSerializer]
    ↓ Post 객체 → JSON 변환
    
[views.py]
    ↓ JSON 응답 반환
    
[클라이언트]
    ← JSON 데이터 수신
```

### 3. 상세 플로우 예시: 게시글 생성 (POST)

```
1. 클라이언트가 POST /api/posts/ 요청
   Body: {"title": "제목", "content": "내용"}

2. mysite/urls.py
   - 'api/' 경로 매칭 → posts.urls로 전달

3. posts/urls.py
   - Router가 'posts/' 경로와 POST 메서드 인식
   - PostViewSet.create() 메서드 호출

4. views.py - PostViewSet.create()
   - PostSerializer로 JSON 데이터 검증 및 역직렬화
   - {"title": "제목", "content": "내용"} → Post 객체

5. serializers.py - PostSerializer
   - JSON → Python 객체 변환
   - 데이터 유효성 검사 (validation)

6. views.py
   - Post 객체를 데이터베이스에 저장
   - Post.objects.create(...)

7. serializers.py
   - 저장된 Post 객체 → JSON 변환
   - {"id": 1, "title": "제목", "content": "내용", "created_at": "..."}

8. 클라이언트에 JSON 응답 반환
```

### 4. 파일 간 의존성 관계

```
models.py (Post 모델)
    ↑
    │ 사용
    │
serializers.py (PostSerializer)
    ↑
    │ 사용
    │
views.py (PostViewSet)
    ↑
    │ 연결
    │
urls.py (Router 등록)
    ↑
    │ 포함
    │
mysite/urls.py (프로젝트 메인 URL)
```

---

## API 엔드포인트 사용법

### 기본 URL
모든 API는 `http://localhost:8000/api/posts/`를 기준으로 동작합니다.

### 1. 전체 게시글 조회 (GET)

**요청:**
```bash
GET http://localhost:8000/api/posts/
```

**응답:**
```json
[
    {
        "id": 1,
        "title": "첫 번째 게시글",
        "content": "안녕하세요",
        "created_at": "2024-01-01T12:00:00Z"
    },
    {
        "id": 2,
        "title": "두 번째 게시글",
        "content": "반갑습니다",
        "created_at": "2024-01-02T10:00:00Z"
    }
]
```

### 2. 특정 게시글 조회 (GET)

**요청:**
```bash
GET http://localhost:8000/api/posts/1/
```

**응답:**
```json
{
    "id": 1,
    "title": "첫 번째 게시글",
    "content": "안녕하세요",
    "created_at": "2024-01-01T12:00:00Z"
}
```

### 3. 게시글 생성 (POST)

**요청:**
```bash
POST http://localhost:8000/api/posts/
Content-Type: application/json

{
    "title": "새 게시글",
    "content": "게시글 내용입니다"
}
```

**응답:**
```json
{
    "id": 3,
    "title": "새 게시글",
    "content": "게시글 내용입니다",
    "created_at": "2024-01-03T14:30:00Z"
}
```

### 4. 게시글 전체 수정 (PUT)

**요청:**
```bash
PUT http://localhost:8000/api/posts/1/
Content-Type: application/json

{
    "title": "수정된 제목",
    "content": "수정된 내용"
}
```

**응답:**
```json
{
    "id": 1,
    "title": "수정된 제목",
    "content": "수정된 내용",
    "created_at": "2024-01-01T12:00:00Z"
}
```

### 5. 게시글 부분 수정 (PATCH)

**요청:**
```bash
PATCH http://localhost:8000/api/posts/1/
Content-Type: application/json

{
    "title": "제목만 수정"
}
```

**응답:**
```json
{
    "id": 1,
    "title": "제목만 수정",
    "content": "기존 내용",
    "created_at": "2024-01-01T12:00:00Z"
}
```

### 6. 게시글 삭제 (DELETE)

**요청:**
```bash
DELETE http://localhost:8000/api/posts/1/
```

**응답:**
```
HTTP 204 No Content
```

---

## 핵심 개념 정리

### 1. MVC 패턴 vs Django의 MVT 패턴

**MVC (Model-View-Controller)**
- Model: 데이터 구조
- View: 사용자 인터페이스
- Controller: 비즈니스 로직

**Django MVT (Model-View-Template)**
- Model: 데이터 구조 (models.py)
- View: 비즈니스 로직 (views.py)
- Template: 사용자 인터페이스 (HTML)

**DRF에서는:**
- Model: 데이터 구조 (models.py)
- Serializer: 데이터 변환 (serializers.py)
- ViewSet: API 로직 (views.py)

### 2. ModelViewSet의 장점

일반적인 Django View를 사용하면:
```python
def post_list(request):
    # GET 요청 처리
    pass

def post_create(request):
    # POST 요청 처리
    pass

def post_detail(request, id):
    # GET 요청 처리
    pass

def post_update(request, id):
    # PUT 요청 처리
    pass

def post_delete(request, id):
    # DELETE 요청 처리
    pass
```

ModelViewSet을 사용하면:
```python
class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
```

**단 3줄로 모든 CRUD 기능이 자동 생성됩니다!**

### 3. Router의 역할

Router는 ViewSet을 자동으로 URL 패턴으로 변환해줍니다.

**수동으로 작성한다면:**
```python
urlpatterns = [
    path('posts/', PostViewSet.as_view({'get': 'list', 'post': 'create'})),
    path('posts/<int:pk>/', PostViewSet.as_view({
        'get': 'retrieve',
        'put': 'update',
        'patch': 'partial_update',
        'delete': 'destroy'
    })),
]
```

**Router 사용:**
```python
router.register(r"posts", PostViewSet)
```

훨씬 간단하고 명확합니다!

### 4. Serializer의 역할

**직렬화 (Serialization)**: Python 객체 → JSON
```python
post = Post(title="제목", content="내용")
serializer = PostSerializer(post)
json_data = serializer.data
# {"id": 1, "title": "제목", "content": "내용", ...}
```

**역직렬화 (Deserialization)**: JSON → Python 객체
```python
json_data = {"title": "제목", "content": "내용"}
serializer = PostSerializer(data=json_data)
if serializer.is_valid():
    post = serializer.save()  # 데이터베이스에 저장
```

### 5. QuerySet이란?

QuerySet은 데이터베이스에서 데이터를 가져오는 Django의 도구입니다.

```python
# 모든 Post 가져오기
Post.objects.all()

# 특정 조건의 Post 가져오기
Post.objects.filter(title="제목")

# 정렬하기
Post.objects.all().order_by("-created_at")  # 최신순

# 하나만 가져오기
Post.objects.get(id=1)
```

---

## 마무리

이 가이드를 통해 Django Rest Framework를 사용한 CRUD API의 전체 구조를 이해하셨을 것입니다. 

### 핵심 요약

1. **Model**: 데이터베이스 구조 정의
2. **Serializer**: 데이터 변환 (JSON ↔ Python 객체)
3. **ViewSet**: API 로직 처리
4. **Router**: URL 자동 생성
5. **URL 설정**: 요청 경로 연결

각 파일의 역할과 상호작용을 이해하면, 더 복잡한 API도 쉽게 확장할 수 있습니다!

### 다음 단계 학습 추천

- 인증/권한 설정 (Authentication & Permissions)
- 필터링 및 검색 기능 (Filtering)
- 페이지네이션 (Pagination)
- 커스텀 액션 추가 (Custom Actions)
- 테스트 작성 (Testing)
