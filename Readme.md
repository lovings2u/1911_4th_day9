# Day9

### python , django, vscode Setting

- `pip install pylint`
- `pip install pylint-django`
- vscode pythonpath 설정하기





### Application 분리하기

- url name space 설정하기

  - 그동안 각자의 역할을 가지고 있는 application에서 views 파일과 urls 파일을 분리하는 작업을 진행했었는데, 이외에도 namespace로 각자의 역할을 분리하고 모든 url을 `urls.py` 파일에서 관리할 수 있게끔 만들어야함

  - *urls.py*

  - ```python
    from django.contrib import admin
    from django.urls import path, include
    
    urlpatterns = [
        path('admin/', admin.site.urls),
        path('articles/', include('articles.urls'))
    ]
    
    ```

    

  - *articles/urls.py*

  - ```python
    from django.contrib import admin
    from django.urls import path
    
    from . import views
    
    app_name = 'articles'
    
    urlpatterns = [
        # CRUD 
        # C -> new, create
        # R -> index, show
        # U -> edit, update
        # D -> Delete
    
        path('', views.index, name="index"),
        path('<int:id>/', views.show, name="show"),
        path('new/', views.new, name="new"),
        path('create/', views.create, name="create"),
        path('<int:id>/edit/', views.edit, name="edit"),
        path('<int:id>/update/', views.update, name="update"),
        path('<int:id>/delete/', views.delete, name="delete"),
    ]
    ```

  - *articles/views.py*

  - ```python
    from django.shortcuts import render, redirect
    
    from .models import Article
    
    # Create your views here.
    
    def index(request):
        # Article Model 에 있는 모든 Article을 불러옴
        articles = Article.objects.all()
        context = {
            'articles': articles
        }
        return render(request, 'index.html', context)
    
    def show(request, id):
        # Article Model 에서 특정 id를 가진 하나의 Article을 불러옴
        article = Article.objects.get(id=id)
        context = {
            'article': article
        }
        return render(request, 'show.html', context)
    
    def new(request):
        # 불러올 Article 없음
        return render(request, 'new.html')
    
    def create(request):
        # Article을 새로 생성
        title = request.GET['title']
        contents = request.GET['contents']
        creator = request.GET['creator']
        # 방법1
        # article = Article.objects.create(title=title, contents=contents, creator=creator)
        # 방법2
        article = Article()
        article.title = title
        article.contents = contents
        article.creator = creator
        article.save()
    
        return redirect('articles:show', article.id)
    
    def edit(request, id):
        # Article Model 에 있는 특정 Article을 가져와야 함
        article = Article.objects.get(id=id)
        context = {
            'article': article
        }
        return render(request, 'edit.html', context)
    
    def update(request, id):
        # Article Model 에 있는 특정 Article을 가져와야 함
        article = Article.objects.get(id=id)
        # 기존 article 정보를 바꿔서 저장하는 부분
        title = request.GET['title']
        contents = request.GET['contents']
        creator = request.GET['creator']
    
        article.title = title
        article.contents = contents
        article.creator = creator
        article.save()
    
        return redirect('articles:show', article.id)
    
    def delete(request, id):
        # Article Model 에 있는 특정 Article을 가져와야 함
        article = Article.objects.get(id=id)
        article.delete()
        return redirect('articles:index')
    ```

  - *articles/templates/index.html*

  - ```html
    {% extends 'base.html' %}
    
    {% block content %}
    <div class="container">
        <table class="table text-center">
            <thead>
                <tr>
                    <th scope="col">#</th>
                    <th scope="col">Title</th>
                    <th scope="col">Creator</th>
                    <th scope="col">Created At</th>
                </tr>
            </thead>
            <tbody>
                {% for article in articles %}
                <tr>
                    <th scope="row">{{article.id}}</th>
                    <td><a href="{% url 'articles:show' article.id %}">{{article.title}}</a></td>
                    <td>{{article.creator}}</td>
                    <td>{{article.datetime_to_string}}</td>
                </tr>
                {% endfor %}
            </tbody>
        </table>
    
        <a href="{% url 'articles:new' %}" class="btn btn-info">새글쓰기</a>
    </div>
    {% endblock %}
    ```

  - 기존에 하드코딩 되어 있던 url 들을 `{% url 'namespace명:name명' %}` 의 형식으로 지정할 수 있고 추후에 `urls.py`에서 end-point에 해당하는 부분만 수정하면 전체적으로 바뀌기 때문에 모든 url을 `urls.py`에서 작성할 수 있음

  - *articles/templates/show.html*

  - ```html
    {% extends 'base.html' %}
    {% block content %}
    <div class="container">
        <h1>{{article.title}}</h1>
        <hr/>
        <p>{{article.contents}}</p>
        <p class="text-right">{{article.created_by}}</p>
        <div>
            <a href="{% url 'articles:edit' article.id %}" class="btn btn-primary">수정하기</a>
            <a href="{% url 'articles:delete' article.id %}" class="btn btn-danger">삭제하기</a>
        </div>
    </div>
    {% endblock %}
    ```

  - *articles/templates/new.html*

  - ```html
    {% extends 'base.html' %}
    {% block content %}
    <div class="container">
        <form action="{% url 'articles:create' %}">
            <div class="form-group">
                <label for="title">Title</label>
                <input name="title" type="text" class="form-control" id="title" placeholder="제목을 입력하세요">
            </div>
            <div class="form-group">
                <label for="contents">Contents</label>
                <textarea name="contents" rows="5" class="form-control" id="contents"></textarea>
            </div>
            <div class="form-group">
                <label for="creator">Creator</label>
                <input name="creator" type="text" class="form-control" id="creator" placeholder="작성자를 입력하세요">
            </div>
            <button type="submit" class="btn btn-primary">작성하기</button>
        </form>
    </div>
    {% endblock %}
    ```

  - *articles/templates/edit.html*

  - ```html
    {% extends 'base.html' %}
    {% block content %}
    <div class="container">
        <form action="{% url 'articles:update' article.id %}">
            <div class="form-group">
                <label for="title">Title</label>
                <input value="{{article.title}}" name="title" type="text" class="form-control" id="title" placeholder="제목을 입력하세요">
            </div>
            <div class="form-group">
                <label for="contents">Contents</label>
                <textarea name="contents" rows="5" class="form-control" id="contents">{{article.contents}}</textarea>
            </div>
            <div class="form-group">
                <label for="creator">Creator</label>
                <input value="{{article.creator}}" readonly="true" name="creator" type="text" class="form-control" id="creator" placeholder="작성자를 입력하세요">
            </div>
            <button type="submit" class="btn btn-primary">수정하기</button>
        </form>
    </div>
    {% endblock %}
    ```





### Model 활용하기

- 추가 필드 설정하기

  - `python manage.py makemigrations`

  - `python manage.py migrate`

  - 추가적으로 마이그레이션을 진행할 경우, 특정 필드에서 NULL 값을 허용할 지에 대해 묻는 내용이 있음

  - NULL=True 혹은 black=True로 진행해야 하는데 자세한 내용은 [다음]( https://wayhome25.github.io/django/2017/09/23/django-blank-null/ )을 참고

  - *articles/models.py*

  - ```python
    class Article(models.Model):
        title = models.CharField(max_length=16)
        contents = models.TextField()
        creator = models.CharField(max_length=8)
        created_at = models.DateTimeField(auto_now_add=True, blank=True)
        updated_at = models.DateTimeField(auto_now=True, blank=True)
      	# created_at, updated_at 추가
    ```

  - 이후에 명령어 실행

- 모델 메소드 추가

  - *articles/models.py*

  - ```python
    class Article(models.Model):
        ...
        
        def __str__(self):
            return f'[{self.title}] - created by {self.creator} at {self.created_at}'
    
        def created_by(self):
            return "created by " + self.creator
        
        def datetime_to_string(self):
            return self.created_at.strftime("%Y-%m-%d")
    ```

  - 기본 제공되는 메소드 이외에 데이터를 우리가 원하는 형식/모양대로 커스텀해서 사용할 수 있도록 모델 메소드를 제공함

  - 다음과 같은 형식으로 사용할 수 있음

  - ```html
     <td>{{article.datetime_to_string}}</td>
    ```


### Django Admin

- 장고에서는 기본적으로 admin 템플릿을 제공하는데, 이 기능이 굉장히 강력함

- 다른 프레임워크에서 라이브러리를 여러개 붙여서 사용해야 하는 내용을 장고에서 기본적으로 제공함

- 사용법도 간단

  - *articles/admin.py*

  - ```python
    from django.contrib import admin
    from .models import Article
    
    # Register your models here.
    # 여기에서 우리가 만든 모델을 등록
    
    admin.site.register(Article)
    ```

  - `python manage.py createsuperuser`

  - id와 email, password 를 설정하면 `localhost:8000/admin/`으로 접속할 수 있음





- 수~목요일에 진행예정

### RESTful api

| 역할   | Request-Method | End-point              | Views(Function) |
| ------ | -------------- | ---------------------- | --------------- |
| Create | GET            | /articles/new/         | new             |
| Create | POST           | /articles/             | create          |
| Read   | GET            | /articles/<id>/        | show            |
| Read   | GET            | /articles/             | index           |
| Update | GET            | /articles/<id>/edit/   | edit            |
| Update | POST           | /articles/<id>/        | update          |
| Delete | POST(DELETE)   | /articles/<id>/delete/ | delete          |

