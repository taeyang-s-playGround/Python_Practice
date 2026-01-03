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

---

## Comment 기능 추가하기

### 개요

Post에 댓글을 달 수 있는 Comment 기능을 추가했습니다. Post와 Comment는 **일대다(One-to-Many)** 관계입니다.
- 하나의 Post에 여러 개의 Comment가 달릴 수 있습니다
- 하나의 Comment는 하나의 Post에만 속합니다

### 1. Comment 모델 추가 (models.py)

**위치**: `mysite/posts/models.py`

```python
class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='comments')
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.post.title} - {self.content[:20]}"
```

#### 코드 분석

**14줄**: `post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='comments')`
- `ForeignKey`: 다른 모델(Post)과의 관계를 정의하는 필드 타입
- `Post`: 연결할 모델 (어떤 Post에 댓글이 달리는지)
- `on_delete=models.CASCADE`: Post가 삭제되면 관련된 모든 Comment도 자동 삭제
  - 예: 게시글이 삭제되면 그 게시글의 모든 댓글도 삭제됨
- `related_name='comments'`: Post에서 댓글에 접근할 때 사용할 이름
  - `post.comments.all()`로 해당 Post의 모든 댓글 조회 가능

**왜 ForeignKey를 사용하나요?**
- 데이터베이스에서 관계를 명확하게 표현
- 데이터 무결성 보장 (존재하지 않는 Post에 댓글을 달 수 없음)
- Django ORM으로 쉽게 관련 데이터 조회 가능

**15줄**: `content = models.TextField()`
- 댓글 내용을 저장하는 필드
- `TextField`: 긴 텍스트 저장 가능

**16줄**: `created_at = models.DateTimeField(auto_now_add=True)`
- 댓글 생성 시간을 자동으로 저장

**18-19줄**: `def __str__(self): return f"{self.post.title} - {self.content[:20]}"`
- 객체를 문자열로 표현할 때 사용
- `f"...{변수}..."`: f-string (문자열 포맷팅)
- `self.content[:20]`: 댓글 내용의 처음 20자만 표시
- 예: "첫 번째 게시글 - 좋은 글입니다!"

#### ForeignKey 관계 이해하기

**일대다 관계:**
```
Post (1) ──────< (Many) Comment
  ↑                    ↑
하나의 Post        여러 개의 Comment
```

**실제 예시:**
```python
# Post 1개에 Comment 3개
post = Post.objects.get(id=1)
comment1 = Comment(post=post, content="첫 번째 댓글")
comment2 = Comment(post=post, content="두 번째 댓글")
comment3 = Comment(post=post, content="세 번째 댓글")

# Post에서 댓글 접근
post.comments.all()  # [comment1, comment2, comment3]

# Comment에서 Post 접근
comment1.post  # Post 객체
comment1.post.title  # "첫 번째 게시글"
```

**on_delete 옵션들:**
- `CASCADE`: 부모(Post) 삭제 시 자식(Comment)도 삭제
- `PROTECT`: 부모 삭제 시 에러 발생 (자식이 있으면 삭제 불가)
- `SET_NULL`: 부모 삭제 시 자식의 외래키를 NULL로 설정 (필드에 null=True 필요)
- `SET_DEFAULT`: 부모 삭제 시 기본값으로 설정

---

### 2. CommentSerializer 생성 (serializers.py)

**위치**: `mysite/posts/serializers.py`

```python
class CommentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Comment
        fields = ["id", "post", "content", "created_at"]
        read_only_fields = ["created_at"]
```

#### 코드 분석

**11줄**: `class CommentSerializer(serializers.ModelSerializer):`
- Comment 모델을 직렬화하는 Serializer 클래스

**13줄**: `model = Comment`
- 어떤 모델을 직렬화할지 지정

**14줄**: `fields = ["id", "post", "content", "created_at"]`
- API 응답에 포함할 필드들
- `id`: 자동 생성되는 고유 번호
- `post`: 댓글이 속한 Post의 ID (ForeignKey는 ID로 변환됨)
- `content`: 댓글 내용
- `created_at`: 생성 시간

**15줄**: `read_only_fields = ["created_at"]`
- 읽기 전용 필드 지정
- 클라이언트가 이 필드를 수정할 수 없음 (자동으로 생성 시간이 설정됨)

**왜 read_only_fields를 사용하나요?**
- `created_at`은 댓글이 생성될 때 자동으로 설정되므로 수정할 필요가 없음
- 보안: 클라이언트가 임의로 시간을 조작하는 것을 방지

---

### 3. CommentViewSet 생성 (views.py)

**위치**: `mysite/posts/views.py`

```python
class CommentViewSet(viewsets.ModelViewSet):
    queryset = Comment.objects.all().order_by("-created_at")
    serializer_class = CommentSerializer
```

#### 코드 분석

**11줄**: `class CommentViewSet(viewsets.ModelViewSet):`
- Comment의 CRUD 기능을 제공하는 ViewSet

**12줄**: `queryset = Comment.objects.all().order_by("-created_at")`
- 모든 Comment를 최신순으로 정렬
- `-created_at`: 내림차순 (최신 댓글이 먼저)

**13줄**: `serializer_class = CommentSerializer`
- 사용할 Serializer 지정

**PostViewSet과 동일한 구조:**
- 같은 패턴을 따르므로 이해하기 쉬움
- Django의 일관성 있는 코드 스타일

---

### 4. URL 라우터 등록 (urls.py)

**위치**: `mysite/posts/urls.py`

```python
router.register(r"comments", CommentViewSet, basename="comment")
```

#### 코드 분석

**전체 코드:**
```python
router = DefaultRouter()
router.register(r"posts", PostViewSet, basename="post")
router.register(r"comments", CommentViewSet, basename="comment")  # 추가된 부분
```

**6줄**: `router.register(r"comments", CommentViewSet, basename="comment")`
- `/api/comments/` 경로에 CommentViewSet 연결
- Post와 동일한 방식으로 CRUD 엔드포인트 자동 생성

**자동 생성되는 URL들:**
- `GET /api/comments/` - 전체 댓글 목록 조회
- `POST /api/comments/` - 새 댓글 생성
- `GET /api/comments/{id}/` - 특정 댓글 조회
- `PUT /api/comments/{id}/` - 댓글 전체 수정
- `PATCH /api/comments/{id}/` - 댓글 부분 수정
- `DELETE /api/comments/{id}/` - 댓글 삭제

---

### 5. 마이그레이션 생성 및 적용

모델을 추가하거나 수정한 후에는 반드시 마이그레이션을 해야 합니다.

#### 마이그레이션이란?
- 모델의 변경사항을 데이터베이스에 반영하는 과정
- Python 코드(모델)와 데이터베이스 스키마를 동기화

#### 명령어

**1. 마이그레이션 파일 생성:**
```bash
python manage.py makemigrations
```
- 모델 변경사항을 감지하여 마이그레이션 파일 생성
- `posts/migrations/0002_comment.py` 파일이 생성됨

**2. 마이그레이션 적용:**
```bash
python manage.py migrate
```
- 생성된 마이그레이션을 데이터베이스에 적용
- 실제로 데이터베이스에 Comment 테이블이 생성됨

**왜 두 단계로 나누나요?**
- `makemigrations`: 변경사항을 파일로 저장 (버전 관리 가능)
- `migrate`: 실제 데이터베이스에 적용
- 개발 중에는 변경사항을 확인하고 수정할 수 있음

---

## Comment API 사용법

### 1. 댓글 생성 (POST)

**요청:**
```bash
POST http://localhost:8000/api/comments/
Content-Type: application/json

{
    "post": 1,
    "content": "좋은 글입니다!"
}
```

**응답:**
```json
{
    "id": 1,
    "post": 1,
    "content": "좋은 글입니다!",
    "created_at": "2024-01-01T12:00:00Z"
}
```

**주의사항:**
- `post` 필드는 Post의 ID를 입력해야 합니다
- 존재하지 않는 Post ID를 입력하면 에러 발생

### 2. 전체 댓글 조회 (GET)

**요청:**
```bash
GET http://localhost:8000/api/comments/
```

**응답:**
```json
[
    {
        "id": 1,
        "post": 1,
        "content": "좋은 글입니다!",
        "created_at": "2024-01-01T12:00:00Z"
    },
    {
        "id": 2,
        "post": 1,
        "content": "저도 동의합니다",
        "created_at": "2024-01-01T12:30:00Z"
    }
]
```

### 3. 특정 댓글 조회 (GET)

**요청:**
```bash
GET http://localhost:8000/api/comments/1/
```

**응답:**
```json
{
    "id": 1,
    "post": 1,
    "content": "좋은 글입니다!",
    "created_at": "2024-01-01T12:00:00Z"
}
```

### 4. 댓글 수정 (PUT)

**요청:**
```bash
PUT http://localhost:8000/api/comments/1/
Content-Type: application/json

{
    "post": 1,
    "content": "수정된 댓글 내용"
}
```

### 5. 댓글 삭제 (DELETE)

**요청:**
```bash
DELETE http://localhost:8000/api/comments/1/
```

**응답:**
```
HTTP 204 No Content
```

---

## 데이터베이스 연결하기

### 1. Django의 데이터베이스 설정

Django는 `settings.py`에서 데이터베이스 연결을 설정합니다.

**위치**: `mysite/mysite/settings.py`

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

#### 코드 분석

**`'default'`**: 기본 데이터베이스 연결 이름
- 여러 데이터베이스를 사용할 수 있지만, 기본은 `default`

**`'ENGINE'`**: 사용할 데이터베이스 엔진
- `'django.db.backends.sqlite3'`: SQLite 사용
- 다른 데이터베이스도 가능 (PostgreSQL, MySQL 등)

**`'NAME'`**: 데이터베이스 파일 경로
- `BASE_DIR / 'db.sqlite3'`: 프로젝트 루트에 `db.sqlite3` 파일 생성

---

### 2. SQLite (기본 설정)

#### SQLite란?
- 파일 기반 데이터베이스
- 별도 설치 없이 사용 가능
- 개발 및 소규모 프로젝트에 적합

#### 장점
- 설정이 간단함
- 별도의 서버 설치 불필요
- 파일 하나로 모든 데이터 관리

#### 단점
- 동시 접근에 제한적
- 대규모 프로젝트에는 부적합

#### 사용 방법
**설정은 이미 완료되어 있음:**
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

**데이터베이스 파일 위치:**
```
mysite/
└── db.sqlite3  # 여기에 자동 생성됨
```

**마이그레이션 실행 시 자동 생성:**
```bash
python manage.py migrate
```
- `db.sqlite3` 파일이 없으면 자동으로 생성됨

---

### 3. PostgreSQL 연결하기

#### PostgreSQL이란?
- 강력한 오픈소스 관계형 데이터베이스
- 프로덕션 환경에서 많이 사용

#### 설치

**1. PostgreSQL 설치:**
```bash
# macOS
brew install postgresql
brew services start postgresql

# Ubuntu
sudo apt-get install postgresql postgresql-contrib

# Windows
# https://www.postgresql.org/download/windows/ 에서 설치
```

**2. 데이터베이스 생성:**
```bash
# PostgreSQL 접속
psql -U postgres

# 데이터베이스 생성
CREATE DATABASE myproject_db;

# 사용자 생성 (선택사항)
CREATE USER myuser WITH PASSWORD 'mypassword';
GRANT ALL PRIVILEGES ON DATABASE myproject_db TO myuser;
```

**3. Django 패키지 설치:**
```bash
pip install psycopg2-binary
```

**4. settings.py 수정:**
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'myproject_db',
        'USER': 'myuser',
        'PASSWORD': 'mypassword',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

**설정 항목 설명:**
- `ENGINE`: PostgreSQL 엔진 사용
- `NAME`: 데이터베이스 이름
- `USER`: 데이터베이스 사용자 이름
- `PASSWORD`: 비밀번호
- `HOST`: 데이터베이스 서버 주소 (로컬은 'localhost')
- `PORT`: 포트 번호 (기본값 5432)

**5. 마이그레이션 적용:**
```bash
python manage.py migrate
```

---

### 4. MySQL 연결하기

#### MySQL이란?
- 널리 사용되는 오픈소스 관계형 데이터베이스

#### 설치

**1. MySQL 설치:**
```bash
# macOS
brew install mysql
brew services start mysql

# Ubuntu
sudo apt-get install mysql-server

# Windows
# https://dev.mysql.com/downloads/installer/ 에서 설치
```

**2. 데이터베이스 생성:**
```bash
# MySQL 접속
mysql -u root -p

# 데이터베이스 생성
CREATE DATABASE myproject_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

**3. Django 패키지 설치:**
```bash
pip install mysqlclient
```

**4. settings.py 수정:**
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'myproject_db',
        'USER': 'root',
        'PASSWORD': 'yourpassword',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}
```

**5. 마이그레이션 적용:**
```bash
python manage.py migrate
```

---

### 5. 데이터베이스 연결 확인

#### 방법 1: Django 쉘 사용

```bash
python manage.py shell
```

```python
# 데이터베이스 연결 확인
from django.db import connection
print(connection.ensure_connection())  # None이면 성공

# 모델 사용 테스트
from posts.models import Post, Comment
Post.objects.all()  # 쿼리가 실행되면 연결 성공
```

#### 방법 2: 마이그레이션 실행

```bash
python manage.py migrate
```

- 에러가 없으면 연결 성공
- 에러 메시지가 나오면 설정을 확인

#### 방법 3: 서버 실행

```bash
python manage.py runserver
```

- 서버가 정상적으로 실행되면 연결 성공
- 데이터베이스 에러가 나오면 설정 확인

---

### 6. 환경 변수로 설정 관리 (보안)

#### 왜 환경 변수를 사용하나요?
- 비밀번호 등 민감한 정보를 코드에 직접 작성하지 않음
- 설정을 환경별로 다르게 관리 가능

#### 사용 방법

**1. python-decouple 설치:**
```bash
pip install python-decouple
```

**2. .env 파일 생성:**
```
# .env 파일 (프로젝트 루트에 생성)
DB_NAME=myproject_db
DB_USER=myuser
DB_PASSWORD=mypassword
DB_HOST=localhost
DB_PORT=5432
```

**3. settings.py 수정:**
```python
from decouple import config

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': config('DB_NAME'),
        'USER': config('DB_USER'),
        'PASSWORD': config('DB_PASSWORD'),
        'HOST': config('DB_HOST'),
        'PORT': config('DB_PORT'),
    }
}
```

**4. .gitignore에 추가:**
```
.env
```

- `.env` 파일은 Git에 커밋하지 않음 (보안)

---

### 7. 데이터베이스 관리 도구

#### SQLite Browser
- SQLite 파일을 시각적으로 확인
- 다운로드: https://sqlitebrowser.org/

#### pgAdmin (PostgreSQL)
- PostgreSQL 관리 도구
- 다운로드: https://www.pgadmin.org/

#### phpMyAdmin (MySQL)
- MySQL 관리 도구
- 웹 기반 인터페이스

#### Django Admin
- Django 내장 관리 도구
- `http://localhost:8000/admin/`에서 접근

---

## 모델 관계 정리

### 관계의 종류

**1. 일대다 (One-to-Many) - ForeignKey**
```
Post (1) ──────< (Many) Comment
```
- 하나의 Post에 여러 Comment
- Comment 모델에 `ForeignKey` 사용

**2. 다대다 (Many-to-Many) - ManyToManyField**
```
Post (Many) ──── (Many) Tag
```
- 하나의 Post에 여러 Tag
- 하나의 Tag가 여러 Post에 사용됨

**3. 일대일 (One-to-One) - OneToOneField**
```
User (1) ────── (1) Profile
```
- 하나의 User에 하나의 Profile

### 현재 프로젝트의 관계

```
Post (1) ──────< (Many) Comment
```

**Post에서 Comment 접근:**
```python
post = Post.objects.get(id=1)
comments = post.comments.all()  # related_name='comments' 사용
```

**Comment에서 Post 접근:**
```python
comment = Comment.objects.get(id=1)
post = comment.post  # ForeignKey로 직접 접근
```

---

## 마무리

Comment 기능을 추가하여 Post에 댓글을 달 수 있는 완전한 CRUD API가 완성되었습니다!

### 핵심 정리

1. **ForeignKey**: 모델 간 일대다 관계 정의
2. **related_name**: 역방향 접근을 위한 이름
3. **on_delete**: 부모 삭제 시 자식 처리 방법
4. **마이그레이션**: 모델 변경사항을 DB에 반영
5. **데이터베이스 설정**: settings.py에서 연결 관리

### 다음 단계

- 특정 Post의 댓글만 조회하는 필터링 기능
- 댓글에 대댓글 기능 추가
- 댓글 좋아요 기능
- 댓글 작성자 정보 추가

Happy Coding! 🚀

