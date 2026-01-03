# Comment 기능 완전 가이드

## 📋 목차
1. [개요](#개요)
2. [Comment 모델 추가](#1-comment-모델-추가-modelspy)
3. [CommentSerializer 생성](#2-commentserializer-생성-serializerspy)
4. [CommentViewSet 생성](#3-commentviewset-생성-viewspy)
5. [URL 라우터 등록](#4-url-라우터-등록-urlspy)
6. [마이그레이션 생성 및 적용](#5-마이그레이션-생성-및-적용)
7. [Comment API 사용법](#comment-api-사용법)
8. [모델 관계 이해하기](#모델-관계-이해하기)

---

## 개요

Post에 댓글을 달 수 있는 Comment 기능을 추가했습니다. Post와 Comment는 **일대다(One-to-Many)** 관계입니다.
- 하나의 Post에 여러 개의 Comment가 달릴 수 있습니다
- 하나의 Comment는 하나의 Post에만 속합니다

### 관계 구조
```
Post (1) ──────< (Many) Comment
  ↑                    ↑
하나의 Post        여러 개의 Comment
```

---

## 1. Comment 모델 추가 (models.py)

**위치**: `mysite/posts/models.py`

```python
class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='comments')
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.post.title} - {self.content[:20]}"
```

### 코드 분석

**14줄**: `post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='comments')`

#### ForeignKey란?
- 다른 모델(Post)과의 관계를 정의하는 필드 타입
- 데이터베이스에서 외래키(Foreign Key)로 구현됨
- "이 댓글은 어떤 게시글에 속하는가?"를 나타냄

#### 각 파라미터 설명

**`Post`**: 연결할 모델
- 어떤 Post에 댓글이 달리는지 지정
- Comment는 반드시 하나의 Post에 속해야 함

**`on_delete=models.CASCADE`**: 삭제 동작 설정
- Post가 삭제되면 관련된 모든 Comment도 자동 삭제
- 예: 게시글이 삭제되면 그 게시글의 모든 댓글도 삭제됨

**다른 on_delete 옵션들:**
```python
# CASCADE: 부모 삭제 시 자식도 삭제 (기본값)
post = models.ForeignKey(Post, on_delete=models.CASCADE)

# PROTECT: 부모 삭제 시 에러 발생 (자식이 있으면 삭제 불가)
post = models.ForeignKey(Post, on_delete=models.PROTECT)

# SET_NULL: 부모 삭제 시 자식의 외래키를 NULL로 설정
post = models.ForeignKey(Post, on_delete=models.SET_NULL, null=True)

# SET_DEFAULT: 부모 삭제 시 기본값으로 설정
post = models.ForeignKey(Post, on_delete=models.SET_DEFAULT, default=1)

# DO_NOTHING: 아무것도 하지 않음 (위험할 수 있음)
post = models.ForeignKey(Post, on_delete=models.DO_NOTHING)
```

**`related_name='comments'`**: 역방향 접근 이름
- Post에서 댓글에 접근할 때 사용할 이름
- `post.comments.all()`로 해당 Post의 모든 댓글 조회 가능
- 이 옵션이 없으면 `post.comment_set.all()`로 접근해야 함

**왜 ForeignKey를 사용하나요?**
1. **데이터 무결성**: 존재하지 않는 Post에 댓글을 달 수 없음
2. **관계 명확화**: 데이터베이스에서 관계를 명확하게 표현
3. **편리한 조회**: Django ORM으로 쉽게 관련 데이터 조회 가능
4. **자동 검증**: Django가 자동으로 유효성 검사

**15줄**: `content = models.TextField()`
- 댓글 내용을 저장하는 필드
- `TextField`: 긴 텍스트 저장 가능 (길이 제한 없음)
- `CharField`와의 차이: CharField는 최대 길이 제한이 있음

**16줄**: `created_at = models.DateTimeField(auto_now_add=True)`
- 댓글 생성 시간을 자동으로 저장
- `auto_now_add=True`: 객체가 처음 생성될 때만 현재 시간 저장
- 이후 수정해도 시간이 변경되지 않음

**18-19줄**: `def __str__(self): return f"{self.post.title} - {self.content[:20]}"`
- 객체를 문자열로 표현할 때 사용
- `f"...{변수}..."`: f-string (문자열 포맷팅)
- `self.content[:20]`: 댓글 내용의 처음 20자만 표시
- 예: "첫 번째 게시글 - 좋은 글입니다!"

**실제 사용 예시:**
```python
# Django 쉘에서
>>> comment = Comment.objects.get(id=1)
>>> print(comment)
첫 번째 게시글 - 좋은 글입니다!
```

---

## 2. CommentSerializer 생성 (serializers.py)

**위치**: `mysite/posts/serializers.py`

```python
class CommentSerializer(serializers.ModelSerializer):
    class Meta:
        model = Comment
        fields = ["id", "post", "content", "created_at"]
        read_only_fields = ["created_at"]
```

### 코드 분석

**11줄**: `class CommentSerializer(serializers.ModelSerializer):`
- Comment 모델을 직렬화하는 Serializer 클래스
- `ModelSerializer`: 모델의 필드를 자동으로 직렬화

**13줄**: `model = Comment`
- 어떤 모델을 직렬화할지 지정
- DRF가 이 모델의 필드를 자동으로 인식

**14줄**: `fields = ["id", "post", "content", "created_at"]`
- API 응답에 포함할 필드들
- `id`: 자동 생성되는 고유 번호
- `post`: 댓글이 속한 Post의 ID (ForeignKey는 ID로 변환됨)
- `content`: 댓글 내용
- `created_at`: 생성 시간

**ForeignKey 필드의 동작:**
```python
# Comment 객체
comment = Comment(post=post_object, content="댓글")

# Serializer로 변환하면
{
    "id": 1,
    "post": 1,  # Post 객체가 아니라 Post의 ID로 변환됨
    "content": "댓글",
    "created_at": "2024-01-01T12:00:00Z"
}
```

**15줄**: `read_only_fields = ["created_at"]`
- 읽기 전용 필드 지정
- 클라이언트가 이 필드를 수정할 수 없음
- 댓글 생성 시 자동으로 현재 시간이 설정됨

**왜 read_only_fields를 사용하나요?**
1. **자동 설정**: `created_at`은 댓글이 생성될 때 자동으로 설정되므로 수정할 필요가 없음
2. **보안**: 클라이언트가 임의로 시간을 조작하는 것을 방지
3. **데이터 무결성**: 생성 시간은 변경되어서는 안 되는 정보

**다른 방법:**
```python
# 방법 1: read_only_fields 사용 (권장)
read_only_fields = ["created_at"]

# 방법 2: 필드에 직접 지정
created_at = serializers.DateTimeField(read_only=True)

# 방법 3: Meta에서 exclude 사용
exclude = ["updated_at"]  # 특정 필드 제외
```

---

## 3. CommentViewSet 생성 (views.py)

**위치**: `mysite/posts/views.py`

```python
class CommentViewSet(viewsets.ModelViewSet):
    queryset = Comment.objects.all().order_by("-created_at")
    serializer_class = CommentSerializer
```

### 코드 분석

**11줄**: `class CommentViewSet(viewsets.ModelViewSet):`
- Comment의 CRUD 기능을 제공하는 ViewSet
- `ModelViewSet`: 자동으로 list, create, retrieve, update, destroy 메서드 제공

**자동 생성되는 메서드들:**
- `list()`: GET /api/comments/ - 전체 댓글 목록 조회
- `create()`: POST /api/comments/ - 새 댓글 생성
- `retrieve()`: GET /api/comments/{id}/ - 특정 댓글 조회
- `update()`: PUT /api/comments/{id}/ - 댓글 전체 수정
- `partial_update()`: PATCH /api/comments/{id}/ - 댓글 부분 수정
- `destroy()`: DELETE /api/comments/{id}/ - 댓글 삭제

**12줄**: `queryset = Comment.objects.all().order_by("-created_at")`
- 이 ViewSet이 다룰 데이터를 지정
- `Comment.objects.all()`: 모든 Comment 객체를 가져옴
- `.order_by("-created_at")`: 생성일 기준 내림차순 정렬 (최신순)
  - `-`는 내림차순을 의미 (없으면 오름차순)

**13줄**: `serializer_class = CommentSerializer`
- 사용할 Serializer 지정
- 모든 CRUD 작업에서 이 Serializer를 사용

**PostViewSet과 동일한 구조:**
- 같은 패턴을 따르므로 이해하기 쉬움
- Django의 일관성 있는 코드 스타일
- 유지보수가 쉬움

---

## 4. URL 라우터 등록 (urls.py)

**위치**: `mysite/posts/urls.py`

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import PostViewSet, CommentViewSet

router = DefaultRouter()
router.register(r"posts", PostViewSet, basename="post")
router.register(r"comments", CommentViewSet, basename="comment")  # 추가된 부분

urlpatterns = [
    path("", include(router.urls)),
]
```

### 코드 분석

**6줄**: `router.register(r"comments", CommentViewSet, basename="comment")`
- `/api/comments/` 경로에 CommentViewSet 연결
- Post와 동일한 방식으로 CRUD 엔드포인트 자동 생성

**파라미터 설명:**
- `r"comments"`: URL 경로 이름 (예: `/api/comments/`)
  - `r`은 raw string (백슬래시 이스케이프 처리 안 함)
- `CommentViewSet`: 연결할 ViewSet
- `basename="comment"`: URL 이름의 기본 이름
  - URL reverse 기능에서 사용 (예: `reverse('comment-list')`)

**자동 생성되는 URL들:**
- `GET /api/comments/` → `CommentViewSet.list()` - 전체 댓글 목록 조회
- `POST /api/comments/` → `CommentViewSet.create()` - 새 댓글 생성
- `GET /api/comments/{id}/` → `CommentViewSet.retrieve()` - 특정 댓글 조회
- `PUT /api/comments/{id}/` → `CommentViewSet.update()` - 댓글 전체 수정
- `PATCH /api/comments/{id}/` → `CommentViewSet.partial_update()` - 댓글 부분 수정
- `DELETE /api/comments/{id}/` → `CommentViewSet.destroy()` - 댓글 삭제

**Router를 사용하지 않으면?**
```python
# 수동으로 작성해야 함 (매우 번거로움)
urlpatterns = [
    path('comments/', CommentViewSet.as_view({'get': 'list', 'post': 'create'})),
    path('comments/<int:pk>/', CommentViewSet.as_view({
        'get': 'retrieve',
        'put': 'update',
        'patch': 'partial_update',
        'delete': 'destroy'
    })),
]
```

---

## 5. 마이그레이션 생성 및 적용

모델을 추가하거나 수정한 후에는 반드시 마이그레이션을 해야 합니다.

### 마이그레이션이란?
- 모델의 변경사항을 데이터베이스에 반영하는 과정
- Python 코드(모델)와 데이터베이스 스키마를 동기화
- 버전 관리가 가능한 데이터베이스 스키마 변경 이력

### 명령어

**1. 마이그레이션 파일 생성:**
```bash
cd mysite
python manage.py makemigrations
```

**결과:**
```
Migrations for 'posts':
  posts/migrations/0002_comment.py
    + Create model Comment
```

**설명:**
- 모델 변경사항을 감지하여 마이그레이션 파일 생성
- `posts/migrations/0002_comment.py` 파일이 생성됨
- 이 파일은 Python 코드로 데이터베이스 변경사항을 기록

**2. 마이그레이션 적용:**
```bash
python manage.py migrate
```

**결과:**
```
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, posts, sessions
Running migrations:
  Applying posts.0002_comment... OK
```

**설명:**
- 생성된 마이그레이션을 데이터베이스에 적용
- 실제로 데이터베이스에 Comment 테이블이 생성됨

### 왜 두 단계로 나누나요?

**`makemigrations`**: 변경사항을 파일로 저장
- 버전 관리 가능 (Git에 커밋 가능)
- 개발 중에는 변경사항을 확인하고 수정할 수 있음
- 여러 개발자가 같은 마이그레이션을 공유 가능

**`migrate`**: 실제 데이터베이스에 적용
- 실제로 데이터베이스 스키마를 변경
- 프로덕션 환경에서는 신중하게 실행해야 함

### 마이그레이션 파일 확인

**생성된 파일**: `posts/migrations/0002_comment.py`
```python
from django.db import migrations, models
import django.db.models.deletion

class Migration(migrations.Migration):
    dependencies = [
        ('posts', '0001_initial'),
    ]

    operations = [
        migrations.CreateModel(
            name='Comment',
            fields=[
                ('id', models.BigAutoField(...)),
                ('content', models.TextField()),
                ('created_at', models.DateTimeField(auto_now_add=True)),
                ('post', models.ForeignKey(
                    on_delete=django.db.models.deletion.CASCADE,
                    related_name='comments',
                    to='posts.post'
                ))),
            ],
        ),
    ]
```

이 파일은 Django가 자동으로 생성한 것이므로 직접 수정하지 않는 것이 좋습니다.

---

## Comment API 사용법

### 기본 URL
모든 Comment API는 `http://localhost:8000/api/comments/`를 기준으로 동작합니다.

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
- `content`는 필수 필드입니다

**에러 예시:**
```json
// Post ID가 존재하지 않을 때
{
    "post": ["Invalid pk \"999\" - object does not exist."]
}

// content가 없을 때
{
    "content": ["This field is required."]
}
```

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
    },
    {
        "id": 3,
        "post": 2,
        "content": "유용한 정보네요",
        "created_at": "2024-01-01T13:00:00Z"
    }
]
```

**특징:**
- 모든 댓글이 최신순으로 정렬되어 반환됨
- 여러 Post의 댓글이 섞여서 반환됨

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

**에러:**
- 존재하지 않는 ID를 요청하면 404 Not Found

### 4. 댓글 전체 수정 (PUT)

**요청:**
```bash
PUT http://localhost:8000/api/comments/1/
Content-Type: application/json

{
    "post": 1,
    "content": "수정된 댓글 내용"
}
```

**응답:**
```json
{
    "id": 1,
    "post": 1,
    "content": "수정된 댓글 내용",
    "created_at": "2024-01-01T12:00:00Z"
}
```

**주의사항:**
- 모든 필드를 포함해야 함 (일부만 수정 불가)
- `post` 필드도 반드시 포함해야 함

### 5. 댓글 부분 수정 (PATCH)

**요청:**
```bash
PATCH http://localhost:8000/api/comments/1/
Content-Type: application/json

{
    "content": "제목만 수정"
}
```

**응답:**
```json
{
    "id": 1,
    "post": 1,
    "content": "제목만 수정",
    "created_at": "2024-01-01T12:00:00Z"
}
```

**PUT vs PATCH:**
- **PUT**: 모든 필드를 포함해야 함 (전체 수정)
- **PATCH**: 일부 필드만 포함해도 됨 (부분 수정)

### 6. 댓글 삭제 (DELETE)

**요청:**
```bash
DELETE http://localhost:8000/api/comments/1/
```

**응답:**
```
HTTP 204 No Content
```

**주의사항:**
- 댓글 삭제 시 데이터베이스에서 완전히 제거됨
- 복구 불가능 (CASCADE 설정에 따라)

---

## 모델 관계 이해하기

### 관계의 종류

#### 1. 일대다 (One-to-Many) - ForeignKey
```
Post (1) ──────< (Many) Comment
```
- 하나의 Post에 여러 Comment
- Comment 모델에 `ForeignKey` 사용
- **현재 프로젝트에서 사용 중**

**실제 예시:**
```python
# Post 1개에 Comment 3개
post = Post.objects.get(id=1)
comment1 = Comment(post=post, content="첫 번째 댓글")
comment2 = Comment(post=post, content="두 번째 댓글")
comment3 = Comment(post=post, content="세 번째 댓글")

# Post에서 댓글 접근
post.comments.all()  # [comment1, comment2, comment3]
post.comments.count()  # 3

# Comment에서 Post 접근
comment1.post  # Post 객체
comment1.post.title  # "첫 번째 게시글"
```

#### 2. 다대다 (Many-to-Many) - ManyToManyField
```
Post (Many) ──── (Many) Tag
```
- 하나의 Post에 여러 Tag
- 하나의 Tag가 여러 Post에 사용됨

**예시:**
```python
class Post(models.Model):
    title = models.CharField(max_length=100)
    tags = models.ManyToManyField(Tag)

class Tag(models.Model):
    name = models.CharField(max_length=50)

# 사용
post = Post.objects.get(id=1)
post.tags.add(tag1, tag2)  # 여러 태그 추가
post.tags.all()  # 모든 태그 조회
```

#### 3. 일대일 (One-to-One) - OneToOneField
```
User (1) ────── (1) Profile
```
- 하나의 User에 하나의 Profile

**예시:**
```python
class User(models.Model):
    username = models.CharField(max_length=100)

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField()

# 사용
user = User.objects.get(id=1)
user.profile  # Profile 객체 (하나만 존재)
```

### 현재 프로젝트의 관계

```
Post (1) ──────< (Many) Comment
```

**Post에서 Comment 접근:**
```python
post = Post.objects.get(id=1)

# 모든 댓글 조회
comments = post.comments.all()

# 댓글 개수
count = post.comments.count()

# 특정 조건의 댓글
recent_comments = post.comments.filter(created_at__gte=today)

# 댓글 생성
post.comments.create(content="새 댓글")
```

**Comment에서 Post 접근:**
```python
comment = Comment.objects.get(id=1)

# Post 객체 접근
post = comment.post

# Post의 필드 접근
title = comment.post.title
content = comment.post.content
```

### 관계 활용 예시

**특정 Post의 댓글만 조회:**
```python
# 방법 1: Post에서 접근
post = Post.objects.get(id=1)
comments = post.comments.all()

# 방법 2: Comment에서 필터링
comments = Comment.objects.filter(post_id=1)

# 방법 3: Post 객체로 필터링
post = Post.objects.get(id=1)
comments = Comment.objects.filter(post=post)
```

**댓글이 많은 Post 조회:**
```python
from django.db.models import Count

# 댓글 개수로 정렬
posts = Post.objects.annotate(
    comment_count=Count('comments')
).order_by('-comment_count')
```

---

## 마무리

Comment 기능을 추가하여 Post에 댓글을 달 수 있는 완전한 CRUD API가 완성되었습니다!

### 핵심 정리

1. **ForeignKey**: 모델 간 일대다 관계 정의
2. **related_name**: 역방향 접근을 위한 이름
3. **on_delete**: 부모 삭제 시 자식 처리 방법
4. **마이그레이션**: 모델 변경사항을 DB에 반영
5. **read_only_fields**: 자동 설정 필드 보호

### 다음 단계 학습 추천

- 특정 Post의 댓글만 조회하는 필터링 기능
- 댓글에 대댓글 기능 추가 (재귀적 관계)
- 댓글 좋아요 기능
- 댓글 작성자 정보 추가
- 댓글 페이징 처리

Happy Coding! 🚀

