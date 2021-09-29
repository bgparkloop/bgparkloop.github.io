---
title: "[Study] Django Basic"
categories:
  - Django
tags:
  - Django
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---

# 0. Django 특징들‌

Django는 MVC 패턴 기반 MVT로 동작

- MVC : Model - View - Controller
    - Model : 데이터베이스에 엑세스하는 컴포넌트
    - View : 데이터를 사용자에게 보여주는 컴포넌트
    - Controller : 데이터를 가져오고 변형하는 컴포넌트
- MVT in django : Model - View - Template
    - Model은 동일
    - View는 MVC의 Controller와 동일
    - Template은 MVC의 View와 동일

‌

## 0.1. 기본 특징들

[Copy of Untitled](https://www.notion.so/8f102af78c024c299bd5394a52630f00)

‌

## 0.2. 장고 기본 세팅

1. Python 사용 가능 환경에서 pip install Django 로 설치.

2. 프로젝트 생성을 위해 다음과 같은 명령어 실행 (project_name 부분에 프로젝트 이름을 넣으면 생성)

```
django-admin startproject project_name
```

3. 웹 애플리케이션에서 동작할 애플리케이션 생성을 위해 다음과 같은 명령어 실행. 실행 전 프로젝트 폴더로 이동.(위의 project_name 폴더)

```
python manage.py startapp app_name
```

4. settings.py를 적절하게 조정 후, 아래와 같은 명령어로 데이터베이스 기본 설정 진행. (기본 테이블 생성을 진행함)

```
python manage.py migrate
```

5. 웹 서버를 실행하여 잘 동작하는지 확인

```
python manage.py runserver [IP][PORT]
IP : 지정하지 않으면 기본 주소 127.0.0.1
PORT : 지정하지 않으면 기본 포트 8888
PORT만 할 경우 runserver 8888
IP만 할 경우 runserver 127.0.0.1
둘다 할 경우, IP:PORT
```

6. 테이블이 정상적으로 생성됬는지 Admin 페이지에서 확인하기

- 관리자(슈퍼유저) 생성하기

```
python manage.py createsuperuser
```

- username, email, password 등록 후 아래의 사이트에서 확인 가능
- http://IP:PORT/admin으로 접속
- 기존 미리 정의된 user와 groups 테이블이 보기

‌

## 0.3. Model 개발

models.py에서 DB를 정의해준다.

```
from django.db import models
# Create your models here.
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
    def __str__(self):
        return self.question_text
class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
    def __str__(self):
        return self.choice_text
```

‌

2. admin.py에 정의한 데이터베이스 테이블을 반영

```
from django.contrib import admin
from polls.models import Question, Choice
# Register your models here.
admin.site.register(Question)
admin.site.register(Choice)
```

‌

3. 데이터베이스 변경사항 반영

```
python manage.py makemigrations
python manage.py migrate
```

‌

## 0.4. 애플리케이션 개발(View & Template)‌

URLconf 코딩 (아래와 같이 필요한 경로에 따라 추가)

```
from django.contrib import admin
from django.urls import path
from polls import views
urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/', views.index, name='index'),
    path('polls/<int:question_id>/', views.detail, name='detail'),
    path('polls/<int:question_id>/results/', views.results, name='results'),
    path('polls/<int:question_id>/vote/', views.vote, name='vote'),
]
```

‌

2.html template 작성

```
## index.html
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
    <li><a href="/polls/{{question.id}}/">{{question.question_text}}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
## detail.html
<h1>{{question.question_text}}</h1>
{% if error_message %}<p><strong>{{error_message}}</strong></p>{% endif %}
<form action="{% url 'polls:vote' question.id %}" method="post">
    {% csrf_token %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{choice.id}}"/>
        <label for="choice{{ forloop.counter }}">{{choice.choice_text}}</label><br/>
    {% endfor %}
<input type="submit" value="Vote" />
</form>
## results.html
<h1>{{ question.question_text }}</h1>
<ul>
    {% for choice in question.choice_set.all %}
        <li>{{ choice.choice_text }} - {{ choice.votes }} vote{{choice.votes|pluralize }}</li>
    {% endfor %}    
</ul>
<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

‌

3. views.py 업데이트

```
from django.shortcuts import render, get_object_or_404
from django.http import HttpResponseRedirect
from django.urls import reverse
from polls.models import Choice, Question
# Create your views here.
def index(request):
    latest_question_list = Question.objects.all().order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # POST 데이터 정상처리하면 리다이렉션
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```

‌

4. IP:PORT/admin에서 데이터 추가하기

- 코드 상에 추가된 질문과 답변 목록은 데이터베이스에저장이 안되있으므로 임의로 입력하여야함.
- 입력하면 IP:PORT/polls로 접속하면 문답 기능을 확인 가능


# 1. Django DB (Admin 관련)

## 1.1. 데이터 수정

```
# UPDATE 절
q.question_text = 'What is your favorite hobby?'
q.save()
# 여러 개 한 번에 수정
Question.objects.filter(pub_date__year=2007)
    .update(question_text='Everything is the same')
```

## 1.2. 데이터 삭제

```
# DELETE 절
Question.objects.filter(pub_date__year=2005).delete()
# 모든 객체 삭제
Question.objects.all().delete()
```

# 2. Django: Web Basic

웹 프로그래밍 : HTTP(S) 프로토콜로 통신하는 클라이언트와 서버 개발

**웹 클라이언트 종류**

- 웹 브라우저를 사용하여 요청
- 리눅스 curl 명령어와 같이
- Telnet 이용
- 직접 만든 클라이언트

HTTP Protocol : Hypertext + multimedia와 같이 모든 종류의 데이터 전송 가능 프로토콜

- HTTP 메시지 구조 : Start Line(Request/Status) | Header | Blank Line | Body
- URI(Uniform Resource Identifier) : URL(Uniform Resource Locator) + URN(Uniform Resource Name)을 포함하는 넓은 의미

[Copy of Untitled](https://www.notion.so/420c650ff7dc4604bb55eedce6fdb926)

GET과 POST의 경우 데이터를 전송하는 방식이 차이가 있음. GET은 주소창에도 보여 공개적이며, 주소길이가 한정적이어서 데이터 전송 길이에도 한계가 있음. 그래서 POST를 통해 파라미터를 전달함.


**상태코드 분류**

[Copy of Untitled](https://www.notion.so/458c3ba5518e4082922f7f4fc3c079d8)

‌

**URL 설계**

[http://www.example.com:80/services?category=2&kind-patents#n10www.example.com](http://www.example.com/services?category=2&kind-patents#n10)

- http:// : URL 스킴. URL에 사용된 프로토콜을 의미
- www~.com : 호스트명. 웹 서버의 호스트명. 도메인명 또는 IP주소
- :80 : 포트번호. 웹 서버 내의 서비스 포트번호. 생략하면 디폴트로 http는 80, https는 443을 사용.
- service : 경로. 파일이나 애플리케이션 경로.
- ?~patents : 쿼리스트링. &로 구분된 이름=값의 쌍으로 표현.
- #n10 : 프라그먼트. 문서 내의 앵커 등 조각을 지정.

‌

**RPC(Remote Procedure Call)** : URL 설계와 API 설계를 동일하게 고려하여 URL 경로를 API 함수로, 쿼리 파라미터를 함수의 인자로 사용하는 방법.

‌

**REST(Representational State Transfer)** : URL을 통해 웹 서버의 특정 리소스를 표현한다음 개념. GET, POST, PUT, DELETE 등의 명령어로 이용함.

‌

Search Engine Friendly or User Friendly URL : 기존 URL에서 &, ?, = 과 같이 프로그래밍에 사용되는 부분을 제거하여 보기 편하게 변경한 방식.

‌

**웹 서버** : 웹 클라이언트의 요청을 받아서 요청을 처리하고, 그 결과를 웹 클라이언트에 응답. 주로 정적 페이지 HTML, 이미지, CSS 등을 웹 클라이언트에게 제공할 때 사용. (Apache httpd, Nginx, etc)

‌

**웹 애플리케이션 서버** : 웹 서버로부터 동적 페이지 요청을 받아 요청을 처리하고 그 결과를 웹 서버로 반환. (Apache Tomcat, JBoss, etc)

‌

**CGI(Common Gateway Interface)** : 웹 서버와 별도 프로그램 사이의 정보를 주고받는 규칙을 정의.

- CGI 방식의 단점 : 필요한 프로그램을 실행할 때, 독립적인 별도의 프로세스가 생성되어 메모리 요구량 및 시스템 부하가 커짐.
- CGI의 대안기술 : 별도의 애플리케이션을 스크립트로 작성하여 웹 서버에 내장하거나 CGI 애플리케이션을 데몬으로 처리하는 방식을 사용.
- 정적, 동적 페이지는 각각 소모되는 자원량이 다르므로 웹 서버와 웹 애플리케이션 서버는 독립적으로 만들어 사용하는 것이 효율적임.