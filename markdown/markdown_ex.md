# 8월10일 강의내용 정리


## **Django 프로젝트 순서**
> 1. 프로젝트 작성 : `django-admin startproject yourprojectname` 학습을 사용하여 새 프로젝트를 시작(관리자권한)
> 2. 앱 생성 : `python manage.py startapp yourappname` 학습을 이용하여 웹사이트의 구성 요소(예: 블로그, 갤러리 등)를 대표하는 앱을 생성
> 3. 모델 정의 : 앱의 `models.py` 파일에서 데이터베이스 테이블을 정의
> 4. 보기(view)와 템플릿 작성 : 검토를 통해 로직을 처리하고 템플릿을 사용하여 사용자에게 보여주기 HTML을 작성
> 5. URL 설정 : `urls.py` 파일에서 URL 패턴을 정의하여 보기(view)와 연결
> 6. 데이터베이스 카탈로그 : `python manage.py makemigrations`과 `python manage.py migrate` 사용하여 모델 변경 사항을 데이터베이스에 적용
> 7. 정적 및 미디어 파일 관리 : `CSS`, `JavaScript`, 이미지 등의 정적 파일을 관리하고 설정
> 8. 개발 서버 실행 : `python manage.py runserver` 기반으로 개발 서버를 실행하고 웹사이트 확인
> 9. 배포 : 웹사이트가 준비되어 있는 호스팅 서비스에 포함하여 인터넷에 배포를 공개


<br>
<br>

> - 프로젝트 생성
```cmd
    > django-admin startproject yourprojectname
```
![이미지](./image02.PNG)
  사진과 같이 프로젝트 폴더 생성


<br>
<br>

> - app생성

```cmd
    > python manage.py startapp yourappname
```
![이미지](./image03.PNG)
blog 라는 디렉토리 생성된 것을 확인

<br>
<br>

> - root폴더에서 settings.py -> INSTALLED_APPS에 'blog'추가

root폴더의 urls.py를 blog에 복사하고 
```python
# blog/urls.py
	from django.contrib import admin
	from django.urls import path
	from . import views

	urlpatterns = [
	    path('', views.home, name ="posts"),
	]
```
root폴더의 urls가서
```python
# mysite/urls.py
	from django.contrib import admin
	from django.urls import path, include

	urlpatterns = [
	    path('admin/', admin.site.urls),
	    path('blog/', include('blog.urls'))
	]
```

<br>
<br>

> - blog/view.py 함수생성 ( request, response)

```python
# blog/view.py
from django.shortcuts import render, redirect, get_object_or_404
from django.http import HttpResponse
from django.contrib import messages     #진행상태를 메세지로 폼에서 출력하고 싶을 때 사용
from .forms import PostForm
from .models import Post
from django.contrib.auth.decorators import login_required
# Create your views here.


def home(request):
    posts = Post.objects.all()  #selectall_QuerySet
    context = {'posts' : posts}
    return render(request, 'blog/home.html', context)

```

<br>
<br>

> - blog/urls.py Url패턴을 매핑선언

```python
#blog/urls/py
from django.contrib import admin
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name ="posts"),
]
```

<br>
<br>

*위와같이 매핑하고 나면, 페이지를 추가할 때마다 `blog/view.py`와 `blog.urls.py`만 수정해주면 된다.*

