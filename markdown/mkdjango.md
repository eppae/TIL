# Django 로그인 구현

1. 프로젝트 작성
```Django
django-admin startproject blog 
```
새 프로젝트 생성 후 `blog`앱을 생성한다
```python
manage.py startapp blog
```
2. 모델 정의

데이터 베이스의 테이블을 **정의**

```python
class Post(models.Model):
    title = models.CharField(max_length=120)
    content = models.TextField()
    published_at = models.DateTimeField(default=timezone.now)
    author =models.ForeignKey(User, on_delete=models.CASCADE)

    def __str__(self):
        return self.title

    class Meta:
        #db_table = 'posts'
        ordering = ['-published_at']
```
>Post 클래스는 models.Model 모듈로,
Post의 Meta 클래스를 이용해 publishe_at 의 내림차순, 정렬 가능

[사용된 모델의 필드는 여기서 확인 가능](https://docs.djangoproject.com/en/4.2/ref/models/fields/)

3. view 와 템플릿 작성

```html
{%load static %}
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="stylesheet" href="{% static 'css/style.css' %}" />
    <script src="{% static 'js/app.js' %}" defer></script>
    <title>My Site</title>
  </head>
  <body>
  	<header>
  		{%if request.user.is_authenticated %}
  			<a href="{% url 'posts' %}">My Posts</a>
  			<a href="{% url 'post-create' %}">New Post</a>
  			<span>Hi {{ request.user.username | title }}</span>
  			<a href="{% url 'logout' %}">Logout</a>
  		{%else%}
  			<a href="{% url 'login' %}">Login</a>
  			<a href="{% url 'register' %}">Register</a>
  		{%endif%}
  	</header>
  	<main>
	  	{% if messages %}
			<div class="messages">
			{% for message in messages %}
				<div class="alert {% if message.tags %}alert-{{ message.tags }}"{% endif %}>
					{{ message }}
				</div>
			{% endfor %}
			</div>
		{% endif %}

	    {%block content%}
	    {%endblock content%}
  	</main>

  </body>
</html>
```
>{% load static %} - 해당 경로를 절대 경로로 지정- blog의 템플릿에서 extends로 상속받아 쓰기 위함 

