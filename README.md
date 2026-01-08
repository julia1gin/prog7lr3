# Django Polls Application

## Гиниятуллина Юлия Сергеевна, ИВТ 2.2

--- 

## Часть 1: Запросы и ответы

### Что сделано:
- Создан проект Django `mysite`
- Создано приложение `polls`
- Настроены базовые URL маршруты
- Создано первое представление (view)

### Комментарии:

#### Структура проекта:
```
Django/
    .venv/              # Виртуальное окружение
    mysite/             # Главный пакет проекта
        __init__.py
        settings.py     # Настройки (TIME_ZONE, LANGUAGE_CODE, INSTALLED_APPS)
        urls.py         # Главные URL маршруты
        asgi.py
        wsgi.py
    polls/              # Приложение для опросов
        __init__.py
        admin.py        # Регистрация моделей в админке
        apps.py
        models.py       # Модели данных (Question, Choice)
        views.py        # Представления (логика обработки запросов)
        urls.py         # URL маршруты приложения (создал вручную!)
        tests.py        # Тесты
        migrations/     # Миграции БД
    manage.py           # Утилита управления проектом
```
![](/images/img.png)

---

## Часть 2: Модели и база данных

### Что сделано:
- Созданы модели Question и Choice
- Выполнены миграции базы данных
- Настроена админ-панель Django
- Протестирована работа с Django ORM в shell
- Добавлен метод was_published_recently()
- Создан суперпользователь для доступа к админке

### Комментарии:

#### Миграции - система контроля версий для БД:
# 1. Создать файлы миграций (обнаружить изменения)
python manage.py makemigrations polls

# 2. Посмотреть SQL (опционально, для обучения)
python manage.py sqlmigrate polls 0001

# 3. Применить миграции (создать таблицы)
python manage.py migrate


#### Django ORM - работа через Shell:
```
python manage.py shell
```
```
from polls.models import Question, Choice
from django.utils import timezone

# CREATE - создание
q = Question(question_text="What's new?", pub_date=timezone.now())
q.save()

# READ - чтение всех
Question.objects.all()

# READ - фильтрация
Question.objects.filter(id=1)
Question.objects.filter(question_text__startswith="What")
Question.objects.filter(pub_date__year=2024)

# READ - получение одного объекта
q = Question.objects.get(pk=1)

# UPDATE - обновление
q.question_text = "What's up?"
q.save()

# DELETE - удаление
q.delete()

# Работа со связанными объектами
q.choice_set.all()  # Все варианты для вопроса
q.choice_set.create(choice_text='Not much', votes=0)
q.choice_set.count()

# Обратная связь
c = Choice.objects.get(pk=1)
c.question  # Получить связанный Question
```

#### Создание суперпользователя:
```
python manage.py createsuperuser
Username: admin
Email: admin@example.com
Password: ********
```

#### Админ-панель Django и вопрос:
![](/images/img_1.png)
![](/images/img_2.png)
![](/images/img_3.png)

---

## Часть 3: Представления и шаблоны

### Что сделано:
- Созданы представления: index, detail, results
- Настроены HTML шаблоны
- Использованы generic views (ListView, DetailView)
- Добавлено пространство имен для URLs

### Комментарии:

#### Структура шаблонов:
```
polls/
    templates/
        polls/              
            index.html
            detail.html
            results.html
```

#### Представления (views) - функциональные:
# polls/views.py
```
from django.shortcuts import render, get_object_or_404
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    context = {"latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})

def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/results.html", {"question": question})

```

#### URL с параметрами:
# polls/urls.py
```
from django.urls import path
from . import views

app_name = "polls"  # Пространство имен

urlpatterns = [
    path("", views.index, name="index"),
    path("<int:question_id>/", views.detail, name="detail"),
    path("<int:question_id>/results/", views.results, name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]

`<int:question_id>` - захватывает число из URL и передает в view как параметр
```

#### Generic Views - получается меньше кода:

# polls/views.py
```
from django.views import generic

class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):
        return Question.objects.order_by("-pub_date")[:5]

class DetailView(generic.DetailView):
    model = Question
    template_name = "polls/detail.html"

class ResultsView(generic.DetailView):
    model = Question
    template_name = "polls/results.html"

#### URLs для generic views:
# polls/urls.py
urlpatterns = [
    path("", views.IndexView.as_view(), name="index"),
    path("<int:pk>/", views.DetailView.as_view(), name="detail"),
    path("<int:pk>/results/", views.ResultsView.as_view(), name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```

---

## Часть 4: Формы и обработка данных

### Что сделано:
- Создана форма для голосования
- Реализована обработка POST-запросов
- Добавлена защита от CSRF
- Использован F() для атомарного обновления
- Реализован паттерн POST/Redirect/GET

### Комментарии:

#### Шаблон результатов голосования:
<!-- polls/templates/polls/results.html -->
```
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>

`|pluralize` - автоматически добавляет "s" для множественного числа
```
---

## Часть 5: Автоматизированное тестирование

### Что сделано:
- Написаны тесты для модели Question
- Написаны тесты для представлений
- Исправлена ошибка в was_published_recently()
- Добавлен фильтр будущих вопросов в IndexView
- Добавлен фильтр будущих вопросов в DetailView

### Комментарии:

#### Первый тест для модели:
# polls/tests.py
```
import datetime
from django.test import TestCase
from django.utils import timezone
from .models import Question

class QuestionModelTests(TestCase):
    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() должен возвращать False для вопросов 
        с pub_date в будущем
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)
```

#### Дополнительные тесты:
```
def test_was_published_recently_with_old_question(self):
    """
    Для вопросов старше 1 дня должно быть False
    """
    time = timezone.now() - datetime.timedelta(days=1, seconds=1)
    old_question = Question(pub_date=time)
    self.assertIs(old_question.was_published_recently(), False)

def test_was_published_recently_with_recent_question(self):
    """
    Для вопросов в пределах последних 24 часов должно быть True
    """
    time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
    recent_question = Question(pub_date=time)
    self.assertIs(recent_question.was_published_recently(), True)
```

#### Результаты тестирования:
![](/images/img_7.png)

---


## Часть 6: Статические файлы (CSS)

### Что сделано:
- Создана структура для статических файлов
- Добавлены стили CSS
- Добавлено фоновое изображение
- Настроена загрузка статики через {% static %}

### Комментарии:

#### Структура статических файлов:
```
polls/
    static/
        polls/              
            style.css
            images/
                background1.png

```

#### Создание CSS файла:

#### Подключение статики в шаблоне:

<!-- polls/templates/polls/index.html -->
```
{% load static %}

<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="{% static 'polls/style.css' %}">
    <title>Polls</title>
</head>
<body>
    {% if latest_question_list %}
        <ul>
        {% for question in latest_question_list %}
            <li>
                <a href="{% url 'polls:detail' question.id %}">
                    {{ question.question_text }}
                </a>
            </li>
        {% endfor %}
        </ul>
    {% else %}
        <p>No polls are available.</p>
    {% endif %}
</body>
</html>
```

Путь в CSS к изображению:

url("images/background1.png")  /* Относительный путь от style.css */

#### Кастомизация

![](/images/img_4.png)

---

## Часть 7: Настройка админ-панели

### Что сделано:
- Настроена кастомная форма редактирования Question
- Добавлено inline редактирование Choice
- Настроен список вопросов (list_display)
- Добавлены фильтры и поиск
- Добавлен декоратор @admin.display
- Кастомизирован заголовок админки
- Создан шаблон templates/admin/base_site.html

### Комментарии:

#### Базовая настройка admin:

Что изменилось:
- По умолчанию: question_text, pub_date
- После настройки: pub_date, question_text

![](/images/img_5.png)
![](/images/img_6.png)
