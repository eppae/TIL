# 블로그 로그인,로그아웃 구현

## 부트스트랩 적용
- django settings.py 에 static 경로 추가
```python
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(os.path.dirname(BASE_DIR), 'static')
STATICFILES_DIRS = (
    os.path.join(BASE_DIR,'static'),
)
```
- static 폴더 내에 부트스트랩의 css,js,vendor 파일 집어넣기
- html 상단에 `{% load static %}` 표기

```python
 link rel = '' href ="static '/내부경로' " 
 ```
# 프로젝트 시작
- `django-admin startproject <프로젝트-이름>`

- 앱 생성 (user)

해당하는 프로젝트에서 사용자 앱 생성

python manage.py startapp <app 이름>
```bash
python manage.py startapp user
```

- setttings.py 에 사용할 app 등록
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'users',
]
```
- settings.py templates 의 기본 경로 설정 
```python
TEMPLATES = [
    {
        'DIRS': [BASE_DIR/'templates'],        
    },
]
```
## 링크 연결
- 프로젝트의 urls.py 에 path,include import
```python
from django.urls import path,include
urlpatterns = [
    path('admin/', admin.site.urls),
    path('',include('users.urls')),
]
```

- users/templates 에 main.html 생성 후 users.url.py 에 연결
```python
from django.urls import path
from . import views
urlpatterns = [path('',views.main,name='main'),
]
```
- views.py 에 메인 홈페이지로 가는 함수 구현
```python
def main(request):
    return(render(request,'main.html'))
```

## 회원가입
- url.py 추가
```python
urlpatterns = [path('',views.main,name='main'),
path('register/',views.register,name='register'),
]
```
- 회원가입 폼 구현

django.contrib.auth.models 의 User 모델을 기본으로 사용한다 

이 모델은 공식 홈페이지에 나와있다.

form의 경우는 UserCreationForm을 사용한다
```python
from django import forms
from django.contrib.auth.models import User
from django.contrib.auth.forms import UserCreationForm
```
### form 구현

username, email , password1, password2 는 기본적으로 User에서 제공해주는 필드이다.

이메일 필드 재설정
```python
class RegisterForm(UserCreationForm):
    email= forms.EmailField(label='Email adress', max_length=254, required=True)
```
class Meta를 사용하여 사용할 모델을 지정한다

**회원가입필드를 이름,이메일,비밀번호 순으로 받음**

```python
class RegisterForm(UserCreationForm):
    email= forms.EmailField(label='Email adress', max_length=254, required=True)
    class Meta:
        model= User
        fields = ['username','email','password1','password2']
```
> 이 경우 초반에 에러가 생겼는데 그 이유는 User 모델은 기본적으로 password를 하나만 받을 수가 없다 저 필드들은 기본적으로 주어진 필수 필드들이고, 자동적으로 생성되는 필드들이라,   
비밀번호를 하나만 입력하면, 패스워드 필드를 두개 생성해서 각각 비교 후 일치시 통과시키기 때문, 하나만 받고 password1,2 에 각각 대입시키던가, 아니면 회원 가입 시에 두개 필드를 다 받아야 한다.

### views.py 구현

> 회원가입 html 양식을 작성하고, templates 의 account 폴더에 넣는다 
html 의 명은 auth-register-basic.html로 했다.

- 우선 request 가 get 방식일 때
```python
    def sign_up(request):
    if request.method =='GET':
        #form = RegisterForm()
        return render(request,'account/auth-register-basic.html')#,{'form':form})
```
> 회원가입 홈페이지로 이동시킨다

- request 가 POST 방식일 때
```python
if request.method =='POST':
        form = RegisterForm(request.POST)
```
>form 은 RegisterForm 으로 제작한 Form에 request.POST로 받은 값을 대입하겠다.
이때 form 은 객체로써 이 데이터를 받게 된다.

```python
        if form.is_valid():
            user = form.save(commit=False)
            user.username = user.username.lower()
            user.user_id = user.pk
            user.save()
```
> form 이 유효성 검사를 통과할 경우, form 을 저장하고 
이때 commit을 False로 넣는 이유는, username 은 unique key값으로 중복되면 안된다. 그것을 위해 현재 대문자도 중복이 안되는 아이디값만을 저장하기 위해,
값이 들어왔을때, 유효성 검사를 마치고 저장하기 전에 저장을 보류하고, 소문자로 변환해서 값을 일치시킨 후 저장한다


```python
            messages.success(request,"회원가입 성공")
            username = form.cleaned_data['username']
            password = form.cleaned_data['password1']
            user = authenticate(request, username=username, password=password)
            if user:
                login(request,user)
                return redirect('main')
        else:
            print('error')
            return render(request,'account/auth-register-basic.html',{'form':form})
```
> 유저 데이터를 User 에 저장한 후, 성공 메세지를 보내고, username,password를 cleaned_data로 바꾸고, user 의 authenticate 로 username,password를 할당한 뒤
login() 을 사용해서 username,password로 로그인 한다

값이 유효하지 않을 경우 다시 로그인 홈페이지로 render한다.

- 로그인의 경우
loginForm 을 생성 후 (ID,Password) 두개만 필요
방법은 위와 같다. 

# 패스워드 리셋

패스워드 리셋의 경우 좀더 복잡한 절차가 필요하다. 

필요한 폼은 우선 비밀번호를 분실 했을 경우, 
- 개인정보, 이메일을 입력해서 이메일 발송을 누를 창
- 이메일에서 보여줄 비밀번호 리셋 링크 창
- 비밀번호 변경시 입력해야되는 창
- 비밀번호 변경 완료 시 로그인으로 돌아가게 완료됨을 표시하는 창

네개가 필요하다 

이 경우 근데 장고에서 기본적으로 제공해주는 비밀번호 리셋 템플릿을 이미 제공하고 있다

- urls.py 에 다음과 같은 패스를 추가한다
```python
    path('password_reset/', auth_views.PasswordResetView.as_view(),name='password_reset'),

    path('password_reset_done/', auth_views.PasswordResetDoneView.as_view(), name='password_reset_done'),

    path('password_reset_confirm/<uidb64>/<token>/', auth_views.PasswordResetConfirmView.as_view(), name='password_reset_confirm'),

    path('password_reset_complete/', auth_views.PasswordResetCompleteView.as_view(), name='password_reset_complete'),
```
- 비밀번호 되찾기를 할때 ID,email을 입력하는 form을 생성한다

forms.py 
```python
class PasswordReset(PasswordResetForm):
    class Meta:
        model = User
        fields = ['username','email']
```

- 해당 필드를 입력했을 때, auth-password-reset.html 의 링크를 전달 할 수 있도록 settings에 email 필드를 저장한다.
```python
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_USE_TLS = True
EMAIL_HOST = 'smtp.naver.com'  # Naver의 SMTP 서버 호스트
EMAIL_PORT = 587
EMAIL_HOST_USER = 'your_email'  # Naver 계정 이메일 주소
EMAIL_HOST_PASSWORD = 'your_password'  # Naver 계정 이메일 비밀번호
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER  # 기본 발신자 이메일
```
> stmp: Simple Mail Transfer Protocol 의 약자로 인터넷을 통해 전자 메일을 전송하는데 사용되는 프로토콜
메일송신,메일 라우팅, 메일 전송 보고 

> TLS: stmp 를 암호화 하는것

이메일, 아이디를 필드로 받아 해당하는 email에 비밀번호를 재설정 할 수 있는 링크를 보내는 코드를 작성
```python
def forgot(request):
    if request.method =='GET':
        return render(request,'account/auth-forgot-password-basic.html')
    if request.method =='POST':
        form = PasswordResetForm(request.POST)
        if not form.is_valid():
            print(form.errors)
        elif form.is_valid():
            form.save(
                request=request,
                from_email='cinester@naver.com',
                email_template_name='account/auth-password-reset.html'
            )
        else:
            form = PasswordResetForm()
        return render(request,'account/auth-forgot-password-basic.html')
```
> request 를 해당하는 email에서 보내겠다. 보내는 템플릿은 해당하는 auth-password-reset.html 이다.

주의할 점은 해당하는 이메일에서 stmp 사용설정을 해놔야 보낼 수 있다.

***내일 작성할 요소***
- 프로필 설정 구현
- 게시판 검색 기능 
- 영화 배너의 youtube 스크롤바에 따른 크기 조정은 어떻게 구현하였는가