# D6.6
simpleapp/models.py
from django.db import models
from django.core.validators import MinValueValidator

class Post(models.Model):
    post_name = models.CharField(
        max_length=50,
        unique=True, 
    )
    time_created_post = models.DateTimeField(auto_now_add = True)

    description = models.TextField()
  
    category = models.ForeignKey(
        to='Category',
        on_delete=models.CASCADE,
        related_name='post', )

    def __str__(self):
        return f'{self.post_name.title()}:{self.time_created_post ()}:{self.description[:20]}'

class Category(models.Model):
    # названия категорий тоже не должны повторяться
    name = models.CharField(max_length=100, unique=True) 
    def __str__(self):
        return self.name.title()


simpleapp/admin.py

from django.contrib import admin
from .models import Category, Post


admin.site.register(Category)
admin.site.register(Post)

simpleapp/views.py
from django.views.generic import ListView, DetailView
from .models import Post
from datetime import datetime

class PostList(ListView):
    model = Post
    ordering = 'post_name'
    template_name = 'news.html'
    context_object_name = 'postes'

    def get_context_data(self, **kwargs):
         context['time_now'] = datetime.utcnow()
        context['next_sale'] = None
        return context


class PostDetail(DetailView):
    model = Post
    template_name = 'new.html'
    context_object_name = 'poste'


simpleapp/urls.py
from django.urls import path
from .views import PostList 

urlpatterns = [
   path('', PostList.as_view()), 
   path('<int:pk>', PostDetail.as_view())
]

project/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
   path('admin/', admin.site.urls),
   path('pages/', include('django.contrib.flatpages.urls')),
   path('news/', include('simpleapp.urls')),
]

templates/post.html
{% extends 'flatpages/default.html' %} 
 
{% block title %}
Post
{% endblock title %}
 
{% block content %}
<h1>Все новости</h1>
   <h3>{{ time_now|date:'D m Y' }}</h3>
   <hr>
   {% if post %}
       <table>
           <tr>
               <td>Заголовок</td>
               <td>Текст статьи</td>
           </tr>

           {% for postes in post %}
           <tr>
               <td>{{ post_name.name }}</td>
               <td>{{ post.description|truncatechars:20 }}</td>
               <td>{{ post.category.name }}</td>
               <td>{{ post.post_name|censor }}</td>
           </tr>
           {% endfor %}

       </table>
   {% else %}
       <h2>Новости нет!</h2>
   {% endif %}
{% endblock content %}


simpleapp/templatetags/custom_filters.py
from django import template

register = template.Library()

@register.filter()
def censor(value):
   """
   value: 
   """
   return f'{value} *'

