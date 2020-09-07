# Live Project

After months of eductation in in mutilple programming languages through the Tech Academy's Python Bootcamp, I was able to work on a team of developers using the Django framework to create a hobby tracker application. Using Python, I created functionality on my pages to have the ability to add, edit, and delete song listings in a database. All (template) pages were styled using Boostrap 4. Not only was there more coding experience gained but experience with the team dynamics and project management of Azure DevOps was also gained through this sprint. Below are code snippets of some of the work done on the Live Project.


## Back End
### models.py
Create the model to add to the SQL database rendered by DJango:

    from django.db import models



    class Song(models.Model):
        title = models.CharField(max_length=500, default="", blank=True, null=False)
        artist = models.CharField(max_length=50, default="", blank=True)
        genre = models.CharField(max_length=50, default="", blank=True)
        album = models.CharField(max_length=50, default="", blank=True)
        year = models.IntegerField(blank=True)
        objects = models.Manager()

    def __str__(self):
        return self.title

### views.py
Functions for the user to be able to interact with the database:

      from django.shortcuts import render, redirect, get_object_or_404
      from django.contrib import messages
      from .forms import SongForm
      from .models import Song



      #home
      def MusicApp_home(request):
          return render(request, "MusicApp/MusicApp_home.html")

      #song addition
      def MusicApp_addForm(request):
          form = SongForm(request.POST or None)
          if form.is_valid():
              form.save()
              return redirect('view_index')
          else:
              print(form.errors)
              form = SongForm()
          context = {'form': form}
          return render(request, "MusicApp/MusicCreate.html", context)

      #index
      def MusicApp_index(request):
          MyMusic = Song.objects.all()
          return render(request, "MusicApp/music_index.html", {"MyMusic": MyMusic})

      #song details
      def MusicApp_details(request, pk):
          pk = int(pk)
          song_details = Song.objects.get(id=pk)
          return render(request, "MusicApp/music_details.html", {'song_details': song_details})

      #edit song
      def MusicApp_edit(request, pk):
          pk = int(pk)
          song = get_object_or_404(Song, pk=pk)
          form = SongForm(data=request.POST or None, instance=song)
          if request.method == 'POST':
              if form.is_valid():
                  form_edit = form.save(commit=False)
                  form_edit.save()
                  return redirect('view_index')
              else:
                  print(form.errors)
          else:
              return render(request, 'MusicApp/music_edit.html', {'form': form})

      #delete song
      def MusicApp_delete(request, pk):
          pk = int(pk)
          song = get_object_or_404(Song, pk=pk)
          if request.method == 'POST':
              song.delete()
              return redirect('view_index')
          context = {'song': song}
          return render(request, "MusicApp/music_delete.html", context)
          
### urls.py          
Have each view reference it's specific template:
        
        from django.urls import path
        from django.contrib import admin
        from . import views

        urlpatterns = [
            path('', views.MusicApp_home, name='home_Music'), #homepage url
            path('add-song/', views.MusicApp_addForm, name='add_Music'), #song addition form
            path('index/', views.MusicApp_index, name='view_index'), #view database
            path('index/song-details/<int:pk>/', views.MusicApp_details, name='song_details'),#song details
            path('index/song-details/<int:pk>/edit-song/', views.MusicApp_edit, name='edit_song'),#edit song
            path('index/song-details/<int:pk>/delete-song/', views.MusicApp_delete, name='delete_song') #delete song
        ]
        
        
## Front End
### base.html
All Django templates call for a base.html for the rest of the templates to inherit elements from:

      {% load static from staticfiles %}
      <!DOCTYPE html>
      <html lang="en">
        <head>
            <meta charset="UTF-8">
          <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
          <title>{% block title %}{% endblock %}</title>
          <link rel="stylesheet" type="text/css" href="{% static 'MusicApp/css/home.css' %}">
          <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous">
        </head>
          <body class="bg-dark">

          <div>
            {% block nav %}
            {% endblock %}
          </div>
          <div>
            <h1 class="pb-4 display-3">{% block header %}{% endblock %}</h1>
          </div>
          <div>
            {% block content %}{% endblock %}
          </div>


        <script src="https://code.jquery.com/jquery-3.5.1.slim.min.js" integrity="sha384-DfXdz2htPH0lsSSs5nCTpuj/zy4C+OGpamoFVy38MVBnE+IbbVYUew+OrCXaRkfj" crossorigin="anonymous"></script>
        <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.1/dist/umd/popper.min.js" integrity="sha384-9/reFTGAW83EW2RDu2S0VKaIzap3H66lZH81PoYlFhbGU+6BZp6G7niu735Sk7lN" crossorigin="anonymous"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js" integrity="sha384-B4gt1jrGC7Jh4AgTPSdUtOBvfO8shuf57BaghqFfPlYxofvL8/KUEfYiJOMMV+rV" crossorigin="anonymous"></script>
          </body>
      </html>
      
### home.html      
Rendered home page:
      {% extends "MusicApp/MusicApp_base.html" %}
      {% load static %}
      {% block title %}Music Collections{% endblock %}

      {% block header %}Your Music Collections App{% endblock %}
      <link rel="stylesheet" type="text/css" href="{% static 'MusicApp/css/home.css' %}">

      {% block nav %}
      <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
          <div class="collapse navbar-collapse" id="navbarNavAltMarkup">
              <div class="navbar-nav m-auto p-3 mb-5">
                  <a class="nav-item nav-link active" href="{% url 'home_Music' %}">Home <span class="sr-only">(current)</span></a>
                  <a class="nav-item nav-link" href="{% url 'add_Music' %}">Add a Song</a>
                  <a class="nav-item nav-link" href="{% url 'view_index' %}">Song Index</a>
              </div>
          </div>
      </nav>
      {% endblock %}

      {% block content %}
      <div id="carouselExampleControls" class="carousel slide container" data-ride="carousel">
        <div class="carousel-inner">
          <div class="carousel-item active">
            <img class="d-block w-100 rounded" src="{% static 'MusicApp/images/concert.jpg' %}" alt="concert">
            <div class="carousel-caption d-none d-md-block">
              <h3>ALL of your favorite...</h3>
            </div>
          </div>
          <div class="carousel-item">
            <img class="d-block w-100 rounded" src="{% static 'MusicApp/images/addPage.jpg' %}" alt="guitar">
            <div class="carousel-caption d-none d-md-block">
              <h3>Songs and Artists...</h3>
            </div>
          </div>
          <div class="carousel-item">
            <img class="d-block w-100 rounded" src="{% static 'MusicApp/images/recordPlayer1.jpg' %}" alt="record player">
            <div class="carousel-caption d-none d-md-block">
              <h3>in one place.</h3>
            </div>
          </div>
        </div>
        <a class="carousel-control-prev" href="#carouselExampleControls" role="button" data-slide="prev">
          <span class="carousel-control-prev-icon" aria-hidden="true"></span>
          <span class="sr-only">Previous</span>
        </a>
        <a class="carousel-control-next" href="#carouselExampleControls" role="button" data-slide="next">
          <span class="carousel-control-next-icon" aria-hidden="true"></span>
          <span class="sr-only">Next</span>
        </a>
      </div>
      {% endblock %}
      
### create.html      
Page to add a song to the database:
      {% extends "MusicApp/MusicApp_base.html" %}

      {% load static from staticfiles %}

      {% block title %}Music Collections{% endblock %}


      {% block nav %}
      <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
          <div class="collapse navbar-collapse" id="navbarNavAltMarkup">
              <div class="navbar-nav m-auto p-3 mb-5">
                  <a class="nav-item nav-link active" href="{% url 'home_Music' %}">Home <span class="sr-only">(current)</span></a>
                  <a class="nav-item nav-link" href="{% url 'add_Music' %}">Add a Song</a>
                  <a class="nav-item nav-link" href="{% url 'view_index' %}">Song Index</a>
              </div>
          </div>
      </nav>
      {% endblock %}
      {% block header %}Add a Song:{% endblock %}
      {% block content %}
      <div class="w-25 m-auto p-3 mb-5 form-group">
          <form method="POST" action="{% url 'add_Music' %}">
              <div>
                  {% csrf_token %}
                  {{ form.as_p }}
              </div>
              <div>
                  <input class="mt-5 btn btn-secondary btn-lg btn-block" type= "submit" value="Save">
              </div>

          </form>
      </div>

      {% endblock %}
      
### details.html
After clicking on a song listed in the index, details of the song are rendered:
      {% extends "MusicApp/MusicApp_base.html" %}

      {% block title %}Music Collection{% endblock %}

      {% block header %}Song Details{% endblock %}

      {% block nav %}
      <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
          <div class="collapse navbar-collapse" id="navbarNavAltMarkup">
              <div class="navbar-nav m-auto p-3 mb-5">
                  <a class="nav-item nav-link active" href="{% url 'home_Music' %}">Home <span class="sr-only">(current)</span></a>
                  <a class="nav-item nav-link" href="{% url 'add_Music' %}">Add a Song</a>
                  <a class="nav-item nav-link" href="{% url 'view_index' %}">Song Index</a>
              </div>
          </div>
      </nav>
      {% endblock %}

      {% block content %}

          <div class="m-auto p-5 mb-5 bg-gray">
              <div class="m-auto p-3">
                  <div class="card p-2 w-50 m-auto">
                  <div class="card-header">Song #{{song_details.id}}</div>
                  <div class="card-body">
                      <h5 class="card-title text-center">{{song_details.title}}</h5>
                      <h5 class="card-text text-center">{{song_details.artist}}</h5>
                      <h5 class="card-text text-center">{{song_details.genre}}</h5>
                      <h5 class="card-text text-center">{{song_details.album}}</h5>
                      <h5 class="card-text text-center">{{song_details.year}}</h5>
                  </div>
                      <a class="btn btn-secondary btn-lg btn-block mb-auto p-3" href="edit-song/" role="button">Edit</a>
                      <a class="btn btn-danger btn-lg btn-block mb-auto p-3" href="delete-song/" role="button">Delete</a>

                  </div>
              </div>

              <div>
                  <a class="btn btn-warning btn-lg m-auto" href="javascript:history.back(-1)" role="button">Back</a>
              </div>

          </div>
      {% endblock %}
      .
