# Django_tutorial
Writing your first Django app, part 1¶ Let’s learn by example.  Throughout this tutorial, we’ll walk you through the creation of a basic poll application.  It’ll consist of two parts:  A public site that lets people view polls and vote in them.  An admin site that lets you add, change, and delete polls.

## Create a directory djangotutorial with a project called mysite inside. 
The directory name doesn’t matter to Django; you can rename it to anything you like.
```bash
django-admin startproject mysite djangotutorial
```
What startproject created:
```
djangotutorial/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
```

## Running the development server
```bash
cd djangotutorial
python manage.py runserver
```

## Creating the Polls app
Remember to be in the same directory as manage.py next the command put you in the same folder directory.
```bash
cd djangotutorial
python manage.py startapp polls
```

Then in the ```views.py``` file we paste
```python
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
This is the most basic view possible in Django. To access it in a browser, we need to map it to a URL - and for this we need to define a URL configuration, or “URLconf” for short. These URL configurations are defined inside each Django app, and they are Python files named ```urls.py```.

To define a URLconf for the ```polls``` app, create a file ```polls/urls.py``` with the following content:

```python
from django.urls import path

from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```

Your app directory should now look like:

```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    urls.py
    views.py
```

The next step is to configure the root URLconf in the ```mysite``` project to include the URLconf defined in ```polls.urls```. To do this, add an import for ```django.urls.include``` in ```mysite/urls.py``` and insert an ```include()``` in the ```urlpatterns``` list, so you have:

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("polls/", include("polls.urls")),
    path("admin/", admin.site.urls),
]
```

Go to http://localhost:8000/polls/ in your browser, and you should see the text “Hello, world. You’re at the polls index.”, which you defined in the index view.

## Database setup

Now, open up ```mysite/settings.py```. It’s a normal Python module with module-level variables representing Django settings.

By default, the ```DATABASES``` configuration uses SQLite. If you’re new to databases, or you’re just interested in trying Django, this is the easiest choice. SQLite is included in Python, so you won’t need to install anything else to support your database. When starting your first real project, however, you may want to use a more scalable database like PostgreSQL, to avoid database-switching headaches down the road.

If you wish to use another database, see details to customize and get your database running.

While you’re editing ```mysite/settings.py```, set ```TIME_ZONE``` to your time zone.

Also, note the ```INSTALLED_APPS``` setting at the top of the file. That holds the names of all Django applications that are activated in this Django instance. Apps can be used in multiple projects, and you can package and distribute them for use by others in their projects.

By default, ```INSTALLED_APPS``` contains the following apps, all of which come with Django:

- ```django.contrib.admin``` – The admin site. You’ll use it shortly.

- ```django.contrib.auth``` – An authentication system.

- ```django.contrib.contenttypes``` – A framework for content types.

- ```django.contrib.sessions``` – A session framework.

- ```django.contrib.messages``` – A messaging framework.

- ```django.contrib.staticfiles``` – A framework for managing static files.

These applications are included by default as a convenience for the common case.

Some of these applications make use of at least one database table, though, so we need to create the tables in the database before we can use them. To do that, run the following command:

```bash
cd djangotutorial
python manage.py migrate
```

The ```migrate``` command looks at the ```INSTALLED_APPS``` setting and creates any necessary database tables according to the database settings in your ```mysite/settings.py``` file and the database migrations shipped with the app. You’ll see a message for each migration it applies. If you’re interested, run the command-line client for your database and type ```\dt``` (PostgreSQL), ```SHOW TABLES;``` (MariaDB, MySQL), ```.tables``` (SQLite), or ```SELECT TABLE_NAME FROM USER_TABLES;``` (Oracle) to display the tables Django created.

## Creating models

Now we’ll define your models – essentially, your database layout, with additional metadata.

In our poll app, we’ll create two models: ```Question``` and ```Choice```. A Question has a question and a publication date. A ```Choice``` has two fields: the text of the choice and a vote tally. Each ```Choice``` is associated with a ```Question```.

These concepts are represented by Python classes. Edit the ```polls/models.py``` file so it looks like this:

```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

Here, each model is represented by a class that subclasses ```django.db.models.Model```. Each model has a number of class variables, each of which represents a database field in the model.

Each field is represented by an instance of a ```Field``` class – e.g., ```CharField``` for character fields and ```DateTimeField``` for datetimes. This tells Django what type of data each field holds.

The name of each ```Field``` instance (e.g. ```question_text``` or ```pub_date```) is the field’s name, in machine-friendly format. You’ll use this value in your Python code, and your database will use it as the column name.

You can use an optional first positional argument to a ```Field``` to designate a human-readable name. That’s used in a couple of introspective parts of Django, and it doubles as documentation. If this field isn’t provided, Django will use the machine-readable name. In this example, we’ve only defined a human-readable name for ```Question.pub_date```. For all other fields in this model, the field’s machine-readable name will suffice as its human-readable name.

Some ```Field``` classes have required arguments. ```CharField```, for example, requires that you give it a ```max_length```. That’s used not only in the database schema, but in validation, as we’ll soon see.

A ```Field``` can also have various optional arguments; in this case, we’ve set the ```default``` value of ```votes``` to 0.

Finally, note a relationship is defined, using ```ForeignKey```. That tells Django each ```Choice``` is related to a single ```Question```. Django supports all the common database relationships: many-to-one, many-to-many, and one-to-one.

## Activating models

That small bit of model code gives Django a lot of information. With it, Django is able to:

- Create a database schema (```CREATE TABLE``` statements) for this app.

- Create a Python database-access API for accessing ```Question``` and ```Choice``` objects.

But first we need to tell our project that the ```polls``` app is installed.

To include the app in our project, we need to add a reference to its configuration class in the ```INSTALLED_APPS``` setting. The ```PollsConfig``` class is in the ```polls/apps.py``` file, so its dotted path is ```'polls.apps.PollsConfig'```. Edit the ```mysite/settings.py``` file and add that dotted path to the ```INSTALLED_APPS``` setting. It’ll look like this:

```python
INSTALLED_APPS = [
    "polls.apps.PollsConfig",
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
]
```

Now Django knows to include the ```polls``` app. Let’s run another command:

```bash
cd djangotutorial
python manage.py makemigrations polls
```

You should see something similar to the following:

```terminaloutput
Migrations for 'polls':
  polls/migrations/0001_initial.py
    + Create model Question
    + Create model Choice
```

If you’re interested, you can also run 
```bash
cd djangotutorial
python manage.py check;
```
this checks for any problems in your project without making migrations or touching the database.

Now, run migrate again to create those model tables in your database:

```bash
cd djangotutorial
python manage.py migrate
```
```terminaloutput
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Applying polls.0001_initial... OK
```

The ```migrate``` command takes all the migrations that haven’t been applied (Django tracks which ones are applied using a special table in your database called ```django_migrations```) and runs them against your database - essentially, synchronizing the changes you made to your models with the schema in the database.

Migrations are very powerful and let you change your models over time, as you develop your project, without the need to delete your database or tables and make new ones - it specializes in upgrading your database live, without losing data. We’ll cover them in more depth in a later part of the tutorial, but for now, remember the three-step guide to making model changes:

- Change your models (in ```models.py```).

- Run ```python manage.py makemigrations``` in the ```manage.py``` location to create migrations for those changes.

- Run 
```python manage.py migrate``` in the ```manage.py``` location to apply those changes to the database.

The reason that there are separate commands to make and apply migrations is because you’ll commit migrations to your version control system and ship them with your app; they not only make your development easier, they’re also usable by other developers and in production.

See Migrations for the full details, including how to inspect the SQL a migration will run with ```sqlmigrate``` (see Executing sqlmigrate).

Read the django-admin documentation for full information on what the ```manage.py``` utility can do.

## Playing with the API

ow, let’s hop into the interactive Python shell and play around with the free API Django gives you. To invoke the Python shell, use this command:

```bash
cd djangotutorial
python manage.py shell
```

We’re using this instead of simply typing “python”, because ```manage.py``` sets the ```DJANGO_SETTINGS_MODULE``` environment variable, which gives Django the Python import path to your ```mysite/settings.py``` file. By default, the ```shell``` command automatically imports the models from your ```INSTALLED_APPS```.

Once you’re in the shell, explore the database API:

```python
# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=datetime.UTC)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```

Wait a minute. ```<Question: Question object (1)>``` isn’t a helpful representation of this object. Let’s fix that by editing the ```Question``` model (in the ```polls/models.py``` file) and adding a ```__str__()``` method to both ```Question``` and ```Choice```:

In polls/models.py:
```python
from django.db import models


class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text


class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

It’s important to add ```__str__()``` methods to your models, not only for your own convenience when dealing with the interactive prompt, but also because objects’ representations are used throughout Django’s automatically-generated admin.

Let’s also add a custom method to this model in polls/models.py:

```python
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```

Note the addition of ```import datetime``` and ```from django.utils import timezone```, to reference Python’s standard ```datetime``` module and Django’s time-zone-related utilities in ```django.utils.timezone```, respectively. If you aren’t familiar with time zone handling in Python, you can learn more in the time zone support docs.

Save these changes and start a new Python interactive shell. (If a three-chevron prompt (>>>) indicates you are still in the shell, you need to exit first using ```exit()```). Run:
```bash
cd djangotutorial
python manage.py shell
``` 
again to reload the models.

```python
# Make sure our __str__() addition worked.
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django provides a rich database lookup API that's entirely driven by
# keyword arguments.
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith="What")
<QuerySet [<Question: What's up?>]>

# Get the question that was published this year.
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# Request an ID that doesn't exist, this will raise an exception.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# Lookup by a primary key is the most common case, so Django provides a
# shortcut for primary-key exact lookups.
# The following is identical to Question.objects.get(id=1).
>>> Question.objects.get(pk=1)
<Question: What's up?>

# Make sure our custom method worked.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# Give the Question a couple of Choices. The create call constructs a new
# Choice object, does the INSERT statement, adds the choice to the set
# of available choices and returns the new Choice object. Django creates
# a set (defined as "choice_set") to hold the "other side" of a ForeignKey
# relation (e.g. a question's choice) which can be accessed via the API.
>>> q = Question.objects.get(pk=1)

# Display any choices from the related object set -- none so far.
>>> q.choice_set.all()
<QuerySet []>

# Create three choices.
>>> q.choice_set.create(choice_text="Not much", votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text="The sky", votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text="Just hacking again", votes=0)

# Choice objects have API access to their related Question objects.
>>> c.question
<Question: What's up?>

# And vice versa: Question objects get access to Choice objects.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# The API automatically follows relationships as far as you need.
# Use double underscores to separate relationships.
# This works as many levels deep as you want; there's no limit.
# Find all Choices for any question whose pub_date is in this year
# (reusing the 'current_year' variable we created above).
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith="Just hacking")
>>> c.delete()
```

For more information on model relations, see Accessing related objects. For more on how to use double underscores to perform field lookups via the API, see Field lookups. For full details on the database API, see our Database API reference.

# Introducing the Django Admin

## Creating an admin user

First we’ll need to create a user who can login to the admin site. Run the following command:

```bash
cd djangotutorial
python manage.py createsuperuser
```

Enter your desired username and press enter.

```terminaloutput
Username: admin
```

You will then be prompted for your desired email address:

```terminaloutput
Email address: admin@example.com
```

The final step is to enter your password. You will be asked to enter your password twice, the second time as a confirmation of the first.

```terminaloutput
Password: **********
Password (again): *********
Superuser created successfully.
```

## tart the development server

The Django admin site is activated by default. Let’s start the development server and explore it.

If the server is not running start it like so:

```bash
cd djangotutorial
python manage.py runserver
```

Now, open a web browser and go to “/admin/” on your local domain – e.g., http://127.0.0.1:8000/admin/. You should see the admin’s login screen:

## Make the poll app modifiable in the admin

But where’s our poll app? It’s not displayed on the admin index page.

Only one more thing to do: we need to tell the admin that ```Question``` objects have an admin interface. To do this, open the ```polls/admin.py``` file, and edit it to look like this:

```python
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

# Writing your first Django app, part 3

This tutorial begins where Tutorial 2 left off. We’re continuing the web-poll application and will focus on creating the public interface – “views.”

## Overview

A view is a “type” of web page in your Django application that generally serves a specific function and has a specific template. For example, in a blog application, you might have the following views:

- Blog homepage – displays the latest few entries.

- Entry “detail” page – permalink page for a single entry.

- Year-based archive page – displays all months with entries in the given year.

- Month-based archive page – displays all days with entries in the given month.

- Day-based archive page – displays all entries in the given day.

- Comment action – handles posting comments to a given entry.

In our poll application, we’ll have the following four views:

- Question “index” page – displays the latest few questions.

- Question “detail” page – displays a question text, with no results but with a form to vote.

- Question “results” page – displays results for a particular question.

- Vote action – handles voting for a particular choice in a particular question.

In Django, web pages and other content are delivered by views. Each view is represented by a Python function (or method, in the case of class-based views). Django will choose a view by examining the URL that’s requested (to be precise, the part of the URL after the domain name).

Now in your time on the web you may have come across such beauties as ```ME2/Sites/dirmod.htm?sid=&type=gen&mod=Core+Pages&gid=A6CD4967199A42D9B65B1B```. You will be pleased to know that Django allows us much more elegant URL patterns than that.

A URL pattern is the general form of a URL - for example: ```/newsarchive/<year>/<month>/```.

To get from a URL to a view, Django uses what are known as ‘URLconfs’. A URLconf maps URL patterns to views.

This tutorial provides basic instruction in the use of URLconfs, and you can refer to URL dispatcher for more information.

## Writing more views

Now let’s add a few more views to ```polls/views.py```. These views are slightly different, because they take an argument:

```python
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)


def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)


def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

Wire these new views into the ```polls.urls``` module by adding the following ```path()``` calls:

```python
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path("", views.index, name="index"),
    # ex: /polls/5/
    path("<int:question_id>/", views.detail, name="detail"),
    # ex: /polls/5/results/
    path("<int:question_id>/results/", views.results, name="results"),
    # ex: /polls/5/vote/
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```

Take a look in your browser, at “/polls/34/”. It’ll run the ```detail()``` function and display whatever ID you provide in the URL. Try “/polls/34/results/” and “/polls/34/vote/” too – these will display the placeholder results and voting pages.

When somebody requests a page from your website – say, “/polls/34/”, Django will load the ```mysite.urls``` Python module because it’s pointed to by the ```ROOT_URLCONF``` setting. It finds the variable named ```urlpatterns``` and traverses the patterns in order. After finding the match at ```'polls/'```, it strips off the matching text (```"polls/"```) and sends the remaining text – ```"34/"``` – to the ‘polls.urls’ URLconf for further processing. There it matches ```'<int:question_id>/'```, resulting in a call to the detail() view like so:

```python
detail(request=<HttpRequest object>, question_id=34)
```

The ```question_id=34``` part comes from ```<int:question_id>```. Using angle brackets “captures” part of the URL and sends it as a keyword argument to the view function. The ```question_id``` part of the string defines the name that will be used to identify the matched pattern, and the ```int``` part is a converter that determines what patterns should match this part of the URL path. The colon (```:```) separates the converter and pattern name.

## Write views that actually do something

Each view is responsible for doing one of two things: returning an ```HttpResponse``` object containing the content for the requested page, or raising an exception such as ```Http404```. The rest is up to you.

Your view can read records from a database, or not. It can use a template system such as Django’s – or a third-party Python template system – or not. It can generate a PDF file, output XML, create a ZIP file on the fly, anything you want, using whatever Python libraries you want.

All Django wants is that ```HttpResponse```. Or an exception.

Because it’s convenient, let’s use Django’s own database API, which we covered in Tutorial 2. Here’s one stab at a new ```index()``` view, which displays the latest 5 poll questions in the system, separated by commas, according to publication date:

```python
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    output = ", ".join([q.question_text for q in latest_question_list])
    return HttpResponse(output)


# Leave the rest of the views (detail, results, vote) unchanged
```

There’s a problem here, though: the page’s design is hardcoded in the view. If you want to change the way the page looks, you’ll have to edit this Python code. So let’s use Django’s template system to separate the design from Python by creating a template that the view can use.

First, create a directory called ```templates``` in your ```polls``` directory. Django will look for templates in there.

Your project’s ```TEMPLATES``` setting describes how Django will load and render templates. The default settings file configures a ```DjangoTemplates``` backend whose ```APP_DIRS``` option is set to ```True```. By convention ```DjangoTemplates``` looks for a “templates” subdirectory in each of the ```INSTALLED_APPS```.

Within the ```templates``` directory you have just created, create another directory called ```polls```, and within that create a file called ```index.html```. In other words, your template should be at ```polls/templates/polls/index.html```. Because of how the ```app_directories``` template loader works as described above, you can refer to this template within Django as ```polls/index.html```.

Put the following code in that template in ```polls/templates/polls/index.html```:

```html
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

Now let’s update our ```index``` view in ```polls/views.py``` to use the template:

```python
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    template = loader.get_template("polls/index.html")
    context = {"latest_question_list": latest_question_list}
    return HttpResponse(template.render(context, request))
```

That code loads the template called ```polls/index.html``` and passes it a context. The context is a dictionary mapping template variable names to Python objects.

Load the page by pointing your browser at “/polls/”, and you should see a bulleted-list containing the “What’s up” question from Tutorial 2. The link points to the question’s d

## A shortcut: render()

It’s a very common idiom to load a template, fill a context and return an ```HttpResponse``` object with the result of the rendered template. Django provides a shortcut. Here’s the full ```index()``` view, rewritten:

```python
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    context = {"latest_question_list": latest_question_list}
    return render(request, "polls/index.html", context)
```

Note that once we’ve done this in all these views, we no longer need to import ```loader``` and ```HttpResponse``` (you’ll want to keep ```HttpResponse``` if you still have the stub methods for ```detail```, ```results```, and ```vote```).

The ```render()``` function takes the request object as its first argument, a template name as its second argument and a dictionary as its optional third argument. It returns an ```HttpResponse``` object of the given template rendered with the given context.

## Raising a 404 error

Now, let’s tackle the question detail view – the page that displays the question text for a given poll. Here’s the view in ```polls/view.py```:

```python
from django.http import Http404
from django.shortcuts import render

from .models import Question


# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, "polls/detail.html", {"question": question})
```

The new concept here: The view raises the ```Http404``` exception if a question with the requested ID doesn’t exist.

We’ll discuss what you could put in that ```polls/detail.html``` template a bit later, but if you’d like to quickly get the above example working, a file containing just in ```polls/templates/polls/detail.html```:

```html
{{ question }}
```

will get you started for now.

## A shortcut: get_object_or_404()

It’s a very common idiom to use ```get()``` and raise ```Http404``` if the object doesn’t exist. Django provides a shortcut. Here’s the ```detail()``` view, rewritten in ```polls/views.py```:

```python
from django.shortcuts import get_object_or_404, render

from .models import Question


# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})
```

The ```get_object_or_404()``` function takes a Django model as its first argument and an arbitrary number of keyword arguments, which it passes to the ```get()``` function of the model’s manager. It raises ```Http404``` if the object doesn’t exist.

There’s also a ```get_list_or_404()``` function, which works just as ```get_object_or_404()``` – except using ```filter()``` instead of ```get()```. It raises ```Http404``` if the list is empty.

## Use the template system

Back to the ```detail()``` view for our poll application. Given the context variable ```question```, here’s what the ```polls/detail.html``` template might look like in ```polls/templates/polls/detail.html```:

```html
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

