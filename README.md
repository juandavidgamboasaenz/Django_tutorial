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

The template system uses dot-lookup syntax to access variable attributes. In the example of ```{{ question.question_text }}```, first Django does a dictionary lookup on the object ```question```. Failing that, it tries an attribute lookup – which works, in this case. If attribute lookup had failed, it would’ve tried a list-index lookup.

Method-calling happens in the ```{% for %}``` loop: ```question.choice_set.all``` is interpreted as the Python code ```question.choice_set.all()```, which returns an iterable of ```Choice``` objects and is suitable for use in the ```{% for %}``` tag.

## Removing hardcoded URLs in templates

Remember, when we wrote the link to a question in the ```polls/index.html``` template, the link was partially hardcoded like this:

```html
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```

The problem with this hardcoded, tightly-coupled approach is that it becomes challenging to change URLs on projects with a lot of templates. However, since you defined the ```name``` argument in the ```path()``` functions in the ```polls.urls``` module, you can remove a reliance on specific URL paths defined in your url configurations by using the ```{% url %}``` template tag:

```html
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

The way this works is by looking up the URL definition as specified in the ```polls.urls``` module. You can see exactly where the URL name of ‘detail’ is defined below:

```python
...
# the 'name' value as called by the {% url %} template tag
path("<int:question_id>/", views.detail, name="detail"),
...
```

If you want to change the URL of the polls detail view to something else, perhaps to something like ```polls/specifics/12/``` instead of doing it in the template (or templates) you would change it in ```polls/urls.py```:

```python
...
# added the word 'specifics'
path("specifics/<int:question_id>/", views.detail, name="detail"),
...
```

## Namespacing URL names

The tutorial project has just one app, ```polls```. In real Django projects, there might be five, ten, twenty apps or more. How does Django differentiate the URL names between them? For example, the ```polls``` app has a ```detail``` view, and so might an app on the same project that is for a blog. How does one make it so that Django knows which app view to create for a url when using the ```{% url %}``` template tag?

The answer is to add namespaces to your URLconf. In the ```polls/urls.py``` file, go ahead and add an ```app_name``` to set the application namespace in ```polls/urls.py```:

```python
from django.urls import path

from . import views

app_name = "polls"
urlpatterns = [
    path("", views.index, name="index"),
    path("<int:question_id>/", views.detail, name="detail"),
    path("<int:question_id>/results/", views.results, name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```

Now change your ```polls/index.html``` template from:

```html
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

to point at the namespaced detail view:

```html
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

## Write a minimal form

Let’s update our poll detail template (“polls/detail.html”) from the last tutorial, so that the template contains an HTML ```<form>``` element in ```polls/templates/polls/detail.html```:

```html
<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
<fieldset>
    <legend><h1>{{ question.question_text }}</h1></legend>
    {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
    {% endfor %}
</fieldset>
<input type="submit" value="Vote">
</form>
```

A quick rundown:

- The above template displays a radio button for each question choice. The ```value``` of each radio button is the associated question choice’s ID. The ```name``` of each radio button is ```"choice"```. That means, when somebody selects one of the radio buttons and submits the form, it’ll send the POST data ```choice=#``` where # is the ID of the selected choice. This is the basic concept of HTML forms.

- We set the form’s ```action``` to ```{% url 'polls:vote' question.id %}```, and we set ```method="post"```. Using ```method="post"``` (as opposed to ```method="get"```) is very important, because the act of submitting this form will alter data server-side. Whenever you create a form that alters data server-side, use ```method="post"```. This tip isn’t specific to Django; it’s good web development practice in general.

- ```forloop.counter``` indicates how many times the ```for``` tag has gone through its loop

- Since we’re creating a POST form (which can have the effect of modifying data), we need to worry about Cross Site Request Forgeries. Thankfully, you don’t have to worry too hard, because Django comes with a helpful system for protecting against it. In short, all POST forms that are targeted at internal URLs should use the ```{% csrf_token %}``` template tag.

Now, let’s create a Django view that handles the submitted data and does something with it. Remember, in Tutorial 3, we created a URLconf for the polls application that includes this line in ```polls/urls.py```:

```python
path("<int:question_id>/vote/", views.vote, name="vote"),
```

We also created a dummy implementation of the ```vote()``` function. Let’s create a real version. Add the following to ```polls/views.py```:

```python
from django.db.models import F
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question


# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST["choice"])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(
            request,
            "polls/detail.html",
            {
                "question": question,
                "error_message": "You didn't select a choice.",
            },
        )
    else:
        selected_choice.votes = F("votes") + 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse("polls:results", args=(question.id,)))
```

This code includes a few things we haven’t covered yet in this tutorial:

- ```request.POST``` is a dictionary-like object that lets you access submitted data by key name. In this case, ```request.POST['choice']``` returns the ID of the selected choice, as a string. ```request.POST``` values are always strings.

- Note that Django also provides ```request.GET``` for accessing GET data in the same way – but we’re explicitly using ```request.POST``` in our code, to ensure that data is only altered via a POST call.

- ```request.POST['choice']``` will raise ```KeyError``` if ```choice``` wasn’t provided in POST data. The above code checks for ```KeyError``` and redisplays the question form with an error message if ```choice``` isn’t given.

- ```F("votes") + 1``` instructs the database to increase the vote count by 1.

- After incrementing the choice count, the code returns an ```HttpResponseRedirect``` rather than a normal ```HttpResponse```. ```HttpResponseRedirect``` takes a single argument: the URL to which the user will be redirected (see the following point for how we construct the URL in this case).

- As the Python comment above points out, you should always return an ```HttpResponseRedirect``` after successfully dealing with POST data. This tip isn’t specific to Django; it’s good web development practice in general.

- We are using the ```reverse()``` function in the ```HttpResponseRedirect``` constructor in this example. This function helps avoid having to hardcode a URL in the view function. It is given the name of the view that we want to pass control to and the variable portion of the URL pattern that points to that view. In this case, using the URLconf we set up in Tutorial 3, this ```reverse()``` call will return a string like

```html
  "/polls/3/results/"
```

where the ```3``` is the value of ```question.id```. This redirected URL will then call the ```'results'``` view to display the final page.

As mentioned in Tutorial 3, ```request``` is an ```HttpRequest``` object. For more on ```HttpRequest``` objects, see the request and response documentation.

After somebody votes in a question, the ```vote()``` view redirects to the results page for the question. Let’s write that view in ```polls/views.py```:

```python
from django.shortcuts import get_object_or_404, render


def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/results.html", {"question": question})
```
This is almost exactly the same as the ```detail()``` view from Tutorial 3. The only difference is the template name. We’ll fix this redundancy later.

Now, create a ```polls/results.html``` template:

```html
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

Now, go to ```/polls/1/``` in your browser and vote in the question. You should see a results page that gets updated each time you vote. If you submit the form without having chosen a choice, you should see the error message.

## Use generic views: Less code is better

The ```detail()``` (from Tutorial 3) and ```results()``` views are very short – and, as mentioned above, redundant. The ```index()``` view, which displays a list of polls, is similar.

These views represent a common case of basic web development: getting data from the database according to a parameter passed in the URL, loading a template and returning the rendered template. Because this is so common, Django provides a shortcut, called the “generic views” system.

Generic views abstract common patterns to the point where you don’t even need to write Python code to write an app. For example, the ```ListView``` and ```DetailView``` generic views abstract the concepts of “display a list of objects” and “display a detail page for a particular type of object” respectively.

Let’s convert our poll app to use the generic views system, so we can delete a bunch of our own code. We’ll have to take a few steps to make the conversion. We will:

1. Convert the URLconf.

2. Delete some of the old, unneeded views.

3. Introduce new views based on Django’s generic views.

Read on for details.

## Amend URLconf

First, open the ```polls/urls.py``` URLconf and change it like so:

```python
from django.urls import path

from . import views

app_name = "polls"
urlpatterns = [
    path("", views.IndexView.as_view(), name="index"),
    path("<int:pk>/", views.DetailView.as_view(), name="detail"),
    path("<int:pk>/results/", views.ResultsView.as_view(), name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```

Note that the name of the matched pattern in the path strings of the second and third patterns has changed from ```<question_id>``` to ```<pk>```. This is necessary because we’ll use the ```DetailView``` generic view to replace our ```detail()``` and ```results()``` views, and it expects the primary key value captured from the URL to be called ```"pk"```.

## Amend views

Next, we’re going to remove our old ```index```, ```detail```, and ```results``` views and use Django’s generic views instead. To do so, open the ```polls/views.py``` file and change it like so:

```python
from django.db.models import F
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by("-pub_date")[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = "polls/detail.html"


class ResultsView(generic.DetailView):
    model = Question
    template_name = "polls/results.html"


def vote(request, question_id):
    # same as above, no changes needed.
    ...
```

Each generic view needs to know what model it will be acting upon. This is provided using either the ```model``` attribute (in this example, ```model = Question``` for ```DetailView``` and ```ResultsView```) or by defining the ```get_queryset()``` method (as shown in ```IndexView```).

By default, the ```DetailView``` generic view uses a template called ```<app name>/<model name>_detail.html```. In our case, it would use the template ```polls/question_detail.html```. The ```template_name``` attribute is used to tell Django to use a specific template name instead of the autogenerated default template name. We also specify the ```template_name``` for the ```results``` list view – this ensures that the results view and the detail view have a different appearance when rendered, even though they’re both a ```DetailView``` behind the scenes.

Similarly, the ```ListView``` generic view uses a default template called ```<app name>/<model name>_list.html```; we use ```template_name``` to tell ```ListView``` to use our existing ```"polls/index.html"``` template.

In previous parts of the tutorial, the templates have been provided with a context that contains the ```question``` and ```latest_question_list``` context variables. For ```DetailView``` the ```question``` variable is provided automatically – since we’re using a Django model (```Question```), Django is able to determine an appropriate name for the context variable. However, for ListView, the automatically generated context variable is ```question_list```. To override this we provide the ```context_object_name``` attribute, specifying that we want to use ```latest_question_list``` instead. As an alternative approach, you could change your templates to match the new default context variables – but it’s a lot easier to tell Django to use the variable you want.

Run the server, and use your new polling app based on generic views.

For full details on generic views, see the generic views documentation.

When you’re comfortable with forms and generic views, read part 5 of this tutorial to learn about testing our polls app.

## Introducing automated testing

### What are automated tests?

Tests are routines that check the operation of your code.

Testing operates at different levels. Some tests might apply to a tiny detail (does a particular model method return values as expected?) while others examine the overall operation of the software (does a sequence of user inputs on the site produce the desired result?). That’s no different from the kind of testing you did earlier in Tutorial 2, using the ```shell``` to examine the behavior of a method, or running the application and entering data to check how it behaves.

What’s different in automated tests is that the testing work is done for you by the system. You create a set of tests once, and then as you make changes to your app, you can check that your code still works as you originally intended, without having to perform time consuming manual testing.

### Why you need to create tests

So why create tests, and why now?

You may feel that you have quite enough on your plate just learning Python/Django, and having yet another thing to learn and do may seem overwhelming and perhaps unnecessary. After all, our polls application is working quite happily now; going through the trouble of creating automated tests is not going to make it work any better. If creating the polls application is the last bit of Django programming you will ever do, then true, you don’t need to know how to create automated tests. But, if that’s not the case, now is an excellent time to learn.

#### Tests will save you time

Up to a certain point, ‘checking that it seems to work’ will be a satisfactory test. In a more sophisticated application, you might have dozens of complex interactions between components.

A change in any of those components could have unexpected consequences on the application’s behavior. Checking that it still ‘seems to work’ could mean running through your code’s functionality with twenty different variations of your test data to make sure you haven’t broken something - not a good use of your time.

That’s especially true when automated tests could do this for you in seconds. If something’s gone wrong, tests will also assist in identifying the code that’s causing the unexpected behavior.

Sometimes it may seem a chore to tear yourself away from your productive, creative programming work to face the unglamorous and unexciting business of writing tests, particularly when you know your code is working properly.

However, the task of writing tests is a lot more fulfilling than spending hours testing your application manually or trying to identify the cause of a newly-introduced problem.

#### Tests don’t just identify problems, they prevent them

It’s a mistake to think of tests merely as a negative aspect of development.

Without tests, the purpose or intended behavior of an application might be rather opaque. Even when it’s your own code, you will sometimes find yourself poking around in it trying to find out what exactly it’s doing.

Tests change that; they light up your code from the inside, and when something goes wrong, they focus light on the part that has gone wrong - even if you hadn’t even realized it had gone wrong.

#### Tests make your code more attractive

You might have created a brilliant piece of software, but you will find that many other developers will refuse to look at it because it lacks tests; without tests, they won’t trust it. Jacob Kaplan-Moss, one of Django’s original developers, says “Code without tests is broken by design.”

That other developers want to see tests in your software before they take it seriously is yet another reason for you to start writing tests.

#### Tests help teams work together

The previous points are written from the point of view of a single developer maintaining an application. Complex applications will be maintained by teams. Tests guarantee that colleagues don’t inadvertently break your code (and that you don’t break theirs without knowing). If you want to make a living as a Django programmer, you must be good at writing tests!

## Basic testing strategies

There are many ways to approach writing tests.

Some programmers follow a discipline called “test-driven development”; they actually write their tests before they write their code. This might seem counterintuitive, but in fact it’s similar to what most people will often do anyway: they describe a problem, then create some code to solve it. Test-driven development formalizes the problem in a Python test case.

More often, a newcomer to testing will create some code and later decide that it should have some tests. Perhaps it would have been better to write some tests earlier, but it’s never too late to get started.

Sometimes it’s difficult to figure out where to get started with writing tests. If you have written several thousand lines of Python, choosing something to test might not be easy. In such a case, it’s fruitful to write your first test the next time you make a change, either when you add a new feature or fix a bug.

So let’s do that right away.

## Writing our first test

### We identify a bug

Fortunately, there’s a little bug in the ```polls``` application for us to fix right away: the ```Question.was_published_recently()``` method returns ```True``` if the ```Question``` was published within the last day (which is correct) but also if the ```Question```’s ```pub_date``` field is in the future (which certainly isn’t).

Confirm the bug by using the ```shell``` to check the method on a question whose date lies in the future:

```bash
cd djangotutorial
python manage.py shell
```

```python
>>> import datetime
>>> from django.utils import timezone
>>> # create a Question instance with pub_date 30 days in the future
>>> future_question = Question(pub_date=timezone.now() + datetime.timedelta(days=30))
>>> # was it published recently?
>>> future_question.was_published_recently()
True
```

Since things in the future are not ‘recent’, this is clearly wrong.

### Create a test to expose the bug

What we’ve just done in the ```shell``` to test for the problem is exactly what we can do in an automated test, so let’s turn that into an automated test.

A conventional place for an application’s tests is in the application’s ```tests.py``` file; the testing system will automatically find tests in any file whose name begins with test.

Put the following in the ```tests.py``` file in the ```polls``` application in ```polls/tests.py```:

```python
import datetime

from django.test import TestCase
from django.utils import timezone

from .models import Question


class QuestionModelTests(TestCase):
    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() returns False for questions whose pub_date
        is in the future.
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)
```

Here we have created a ```django.test.TestCase``` subclass with a method that creates a ```Question``` instance with a ```pub_date``` in the future. We then check the output of ```was_published_recently()``` - which ought to be False.

### Running tests

In the terminal, we can run our test:

```bash
cd djangotutorial
python manage.py test polls
```

What happened is this:

- ```manage.py test polls``` looked for tests in the ```polls``` application

- it found a subclass of the ```django.test.TestCase``` class

- it created a special database for the purpose of testing

- it looked for test methods - ones whose names begin with ```test```

- in ```test_was_published_recently_with_future_question``` it created a ```Question``` instance whose ```pub_date``` field is 30 days in the future

- … and using the ```assertIs()``` method, it discovered that its ```was_published_recently()``` returns ```True```, though we wanted it to return ```False```

The test informs us which test failed and even the line on which the failure occurred.

### Fixing the bug

We already know what the problem is:
```Question.was_published_recently()``` should return ```False``` if its ```pub_date``` is in the future. Amend the method in ```models.py```, so that it will only return ```True``` if the date is also in the past in ```polls/models.py```:

```python
def was_published_recently(self):
    now = timezone.now()
    return now - datetime.timedelta(days=1) <= self.pub_date <= now
```

and run the test again:

```terminaloutput
Creating test database for alias 'default'...
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
```

After identifying a bug, we wrote a test that exposes it and corrected the bug in the code so our test passes.

Many other things might go wrong with our application in the future, but we can be sure that we won’t inadvertently reintroduce this bug, because running the test will warn us immediately. We can consider this little portion of the application pinned down safely forever.

### More comprehensive tests

While we’re here, we can further pin down the ```was_published_recently()``` method; in fact, it would be positively embarrassing if in fixing one bug we had introduced another.

Add two more test methods to the same class, to test the behavior of the method more comprehensively in ```polls/test.py```:

```python
def test_was_published_recently_with_old_question(self):
    """
    was_published_recently() returns False for questions whose pub_date
    is older than 1 day.
    """
    time = timezone.now() - datetime.timedelta(days=1, seconds=1)
    old_question = Question(pub_date=time)
    self.assertIs(old_question.was_published_recently(), False)


def test_was_published_recently_with_recent_question(self):
    """
    was_published_recently() returns True for questions whose pub_date
    is within the last day.
    """
    time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
    recent_question = Question(pub_date=time)
    self.assertIs(recent_question.was_published_recently(), True)
```

And now we have three tests that confirm that ```Question.was_published_recently()``` returns sensible values for past, recent, and future questions.

Again, ```polls``` is a minimal application, but however complex it grows in the future and whatever other code it interacts with, we now have some guarantee that the method we have written tests for will behave in expected ways.

## Test a view

The polls application is fairly undiscriminating: it will publish any question, including ones whose ```pub_date``` field lies in the future. We should improve this. Setting a ```pub_date``` in the future should mean that the Question is published at that moment, but invisible until then.

### A test for a view

When we fixed the bug above, we wrote the test first and then the code to fix it. In fact that was an example of test-driven development, but it doesn’t really matter in which order we do the work.

In our first test, we focused closely on the internal behavior of the code. For this test, we want to check its behavior as it would be experienced by a user through a web browser.

Before we try to fix anything, let’s have a look at the tools at our disposal.

### The Django test client

Django provides a test ```Client``` to simulate a user interacting with the code at the view level. We can use it in ```tests.py``` or even in the ```shell```.

We will start again with the ```shell```, where we need to do a couple of things that won’t be necessary in ```tests.py```. The first is to set up the test environment in the ```shell```:

```bash
cd djangotutorial
python manage.py shell
```

```python
>>> from django.test.utils import setup_test_environment
>>> setup_test_environment()
```

```setup_test_environment()``` installs a template renderer which will allow us to examine some additional attributes on responses such as ```response.context``` that otherwise wouldn’t be available. Note that this method does not set up a test database, so the following will be run against the existing database and the output may differ slightly depending on what questions you already created. You might get unexpected results if your ```TIME_ZONE``` in ```settings.py``` isn’t correct. If you don’t remember setting it earlier, check it before continuing.

Next we need to import the test client class (later in ```tests.py``` we will use the ```django.test.TestCase``` class, which comes with its own client, so this won’t be required):

```python
>>> from django.test import Client
>>> # create an instance of the client for our use
>>> client = Client()
```

With that ready, we can ask the client to do some work for us:

```python
>>> # get a response from '/'
>>> response = client.get("/")
Not Found: /
>>> # we should expect a 404 from that address; if you instead see an
>>> # "Invalid HTTP_HOST header" error and a 400 response, you probably
>>> # omitted the setup_test_environment() call described earlier.
>>> response.status_code
404
>>> # on the other hand we should expect to find something at '/polls/'
>>> # we'll use 'reverse()' rather than a hardcoded URL
>>> from django.urls import reverse
>>> response = client.get(reverse("polls:index"))
>>> response.status_code
200
>>> response.content
b'\n    <ul>\n    \n        <li><a href="/polls/1/">What&#x27;s up?</a></li>\n    \n    </ul>\n\n'
>>> response.context["latest_question_list"]
<QuerySet [<Question: What's up?>]>
```

### Improving our view

The list of polls shows polls that aren’t published yet (i.e. those that have a ```pub_date``` in the future). Let’s fix that.

In Tutorial 4 we introduced a class-based view, based on ```ListView``` in ```polls/views.py```:

```python
class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by("-pub_date")[:5]
```

We need to amend the ```get_queryset()``` method and change it so that it also checks the date by comparing it with ```timezone.now()```. First we need to add an import in ```polls/views.py```:

```python
from django.utils import timezone
```

and then we must amend the get_queryset method like so:

```python
def create_question(question_text, days):
    """
    Create a question with the given `question_text` and published the
    given number of `days` offset to now (negative for questions published
    in the past, positive for questions that have yet to be published).
    """
    time = timezone.now() + datetime.timedelta(days=days)
    return Question.objects.create(question_text=question_text, pub_date=time)


class QuestionIndexViewTests(TestCase):
    def test_no_questions(self):
        """
        If no questions exist, an appropriate message is displayed.
        """
        response = self.client.get(reverse("polls:index"))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "No polls are available.")
        self.assertQuerySetEqual(response.context["latest_question_list"], [])

    def test_past_question(self):
        """
        Questions with a pub_date in the past are displayed on the
        index page.
        """
        question = create_question(question_text="Past question.", days=-30)
        response = self.client.get(reverse("polls:index"))
        self.assertQuerySetEqual(
            response.context["latest_question_list"],
            [question],
        )

    def test_future_question(self):
        """
        Questions with a pub_date in the future aren't displayed on
        the index page.
        """
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse("polls:index"))
        self.assertContains(response, "No polls are available.")
        self.assertQuerySetEqual(response.context["latest_question_list"], [])

    def test_future_question_and_past_question(self):
        """
        Even if both past and future questions exist, only past questions
        are displayed.
        """
        question = create_question(question_text="Past question.", days=-30)
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse("polls:index"))
        self.assertQuerySetEqual(
            response.context["latest_question_list"],
            [question],
        )

    def test_two_past_questions(self):
        """
        The questions index page may display multiple questions.
        """
        question1 = create_question(question_text="Past question 1.", days=-30)
        question2 = create_question(question_text="Past question 2.", days=-5)
        response = self.client.get(reverse("polls:index"))
        self.assertQuerySetEqual(
            response.context["latest_question_list"],
            [question2, question1],
        )
```

Let’s look at some of these more closely.

First is a question shortcut function, ```create_question```, to take some repetition out of the process of creating questions.

```test_no_questions``` doesn’t create any questions, but checks the message: “No polls are available.” and verifies the ```latest_question_list``` is empty. Note that the ```django.test.TestCase``` class provides some additional assertion methods. In these examples, we use ```assertContains()``` and ```assertQuerySetEqual()```.

In ```test_past_question```, we create a question and verify that it appears in the list.

In ```test_future_question```, we create a question with a ```pub_date``` in the future. The database is reset for each test method, so the first question is no longer there, and so again the index shouldn’t have any questions in it.

And so on. In effect, we are using the tests to tell a story of admin input and user experience on the site, and checking that at every state and for every new change in the state of the system, the expected results are published.

### Testing the DetailView

What we have works well; however, even though future questions don’t appear in the index, users can still reach them if they know or guess the right URL. So we need to add a similar constraint to ```DetailView``` in ```polls/views.py```:

```python
class DetailView(generic.DetailView):
    ...

    def get_queryset(self):
        """
        Excludes any questions that aren't published yet.
        """
        return Question.objects.filter(pub_date__lte=timezone.now())
```

We should then add some tests, to check that a ```Question``` whose ```pub_date``` is in the past can be displayed, and that one with a ```pub_date``` in the future is not in ```polls/tests.py```:

```python
class QuestionDetailViewTests(TestCase):
    def test_future_question(self):
        """
        The detail view of a question with a pub_date in the future
        returns a 404 not found.
        """
        future_question = create_question(question_text="Future question.", days=5)
        url = reverse("polls:detail", args=(future_question.id,))
        response = self.client.get(url)
        self.assertEqual(response.status_code, 404)

    def test_past_question(self):
        """
        The detail view of a question with a pub_date in the past
        displays the question's text.
        """
        past_question = create_question(question_text="Past Question.", days=-5)
        url = reverse("polls:detail", args=(past_question.id,))
        response = self.client.get(url)
        self.assertContains(response, past_question.question_text)
```

### Ideas for more tests

We ought to add a similar ```get_queryset``` method to ```ResultsView``` and create a new test class for that view. It’ll be very similar to what we have just created; in fact there will be a lot of repetition.

We could also improve our application in other ways, adding tests along the way. For example, it’s pointless that a ```Question``` with no related ```Choice``` can be published on the site. So, our views could check for this, and exclude such ```Question``` objects. Our tests would create a ```Question``` without a ```Choice```, and then test that it’s not published, as well as create a similar ```Question``` *with* at least one ```Choice```, and test that it *is* published.

Perhaps logged-in admin users should be allowed to see unpublished ```Question``` entries, but not ordinary visitors. Again: whatever needs to be added to the software to accomplish this should be accompanied by a test, whether you write the test first and then make the code pass the test, or work out the logic in your code first and then write a test to prove it.

At a certain point you are bound to look at your tests and wonder whether your code is suffering from test bloat, which brings us to:

## When testing, more is better

It might seem that our tests are growing out of control. At this rate there will soon be more code in our tests than in our application, and the repetition is unaesthetic, compared to the elegant conciseness of the rest of our code.

```It doesn’t matter```. Let them grow. For the most part, you can write a test once and then forget about it. It will continue performing its useful function as you continue to develop your program.

Sometimes tests will need to be updated. Suppose that we amend our views so that only ```Question``` entries with associated ```Choice``` instances are published. In that case, many of our existing tests will fail - *telling us exactly which tests need to be amended to bring them up to date*, so to that extent tests help look after themselves.

At worst, as you continue developing, you might find that you have some tests that are now redundant. Even that’s not a problem; in testing redundancy is a good thing.

As long as your tests are sensibly arranged, they won’t become unmanageable. Good rules-of-thumb include having:

- a separate ```TestClass``` for each model or view

- a separate test method for each set of conditions you want to test

- test method names that describe their function

## Further testing

This tutorial only introduces some of the basics of testing. There’s a great deal more you can do, and a number of very useful tools at your disposal to achieve some very clever things.

For example, while our tests here have covered some of the internal logic of a model and the way our views publish information, you can use an “in-browser” framework such as Selenium to test the way your HTML actually renders in a browser. These tools allow you to check not just the behavior of your Django code, but also, for example, of your JavaScript. It’s quite something to see the tests launch a browser, and start interacting with your site, as if a human being were driving it! Django includes ```LiveServerTestCase``` to facilitate integration with tools like Selenium.

If you have a complex application, you may want to run tests automatically with every commit for the purposes of continuous integration, so that quality control is itself - at least partially - automated.

A good way to spot untested parts of your application is to check code coverage. This also helps identify fragile or even dead code. If you can’t test a piece of code, it usually means that code should be refactored or removed. Coverage will help to identify dead code. See Integration with coverage.py for details.

Testing in Django has comprehensive information about testing.

# Writing your first Django app, part 6

This tutorial begins where Tutorial 5 left off. We’ve built a tested web-poll application, and we’ll now add a stylesheet and an image.

Aside from the HTML generated by the server, web applications generally need to serve additional files — such as images, JavaScript, or CSS — necessary to render the complete web page. In Django, we refer to these files as “static files”.

For small projects, this isn’t a big deal, because you can keep the static files somewhere your web server can find it. However, in bigger projects – especially those comprised of multiple apps – dealing with the multiple sets of static files provided by each application starts to get tricky.

That’s what ```django.contrib.staticfiles``` is for: it collects static files from each of your applications (and any other places you specify) into a single location that can easily be served in production.

## Customize your app’s look and feel

First, create a directory called ```static``` in your ```polls``` directory. Django will look for static files there, similarly to how Django finds templates inside ```polls/templates/```.

Django’s ```STATICFILES_FINDERS``` setting contains a list of finders that know how to discover static files from various sources. One of the defaults is ```AppDirectoriesFinder``` which looks for a “static” subdirectory in each of the ```INSTALLED_APPS```, like the one in ```polls``` we just created. The admin site uses the same directory structure for its static files.

Within the ```static``` directory you have just created, create another directory called ```polls``` and within that create a file called ```style.css```. In other words, your stylesheet should be at ```polls/static/polls/style.css```. Because of how the ```AppDirectoriesFinder``` staticfile finder works, you can refer to this static file in Django as ```polls/style.css```, similar to how you reference the path for templates.

Put the following code in that stylesheet (```polls/static/polls/style.css```):

```css
li a {
    color: green;
}
```

Next, add the following at the top of ```polls/templates/polls/index.html```:

```html
{% load static %}

<link rel="stylesheet" href="{% static 'polls/style.css' %}">
```

The ```{% static %}``` template tag generates the absolute URL of static files.

That’s all you need to do for development.

Start the server (or restart it if it’s already running):

```bash
cd djangotutorial
python manage.py runserver
```

Reload ```http://localhost:8000/polls/``` and you should see that the question links are green (Django style!) which means that your stylesheet was properly loaded.

## Adding a background-image

Next, we’ll create a subdirectory for images. Create an ```images``` subdirectory in the ```polls/static/polls/``` directory. Inside this directory, add any image file that you’d like to use as a background. For the purposes of this tutorial, we’re using a file named ```background.png```, which will have the full path ```polls/static/polls/images/background.png```.

Then, add a reference to your image in your stylesheet (```polls/static/polls/style.css```):

```css
body {
    background: white url("images/background.png") no-repeat;
}
```

Reload ```http://localhost:8000/polls/``` and you should see the background loaded in the top left of the screen.

These are the ```basics```. For more details on settings and other bits included with the framework see the static files howto and the staticfiles reference. Deploying static files discusses how to use static files on a real server.

When you’re comfortable with the static files, read part 7 of this tutorial to learn how to customize Django’s automatically-generated admin site.

## Customize the admin form

By registering the ```Question``` model with ```admin.site.register(Question)```, Django was able to construct a default form representation. Often, you’ll want to customize how the admin form looks and works. You’ll do this by telling Django the options you want when you register the object.

Let’s see how this works by reordering the fields on the edit form. Replace the ```admin.site.register(Question)``` line with in ```polls/admin.py```:

```python
from django.contrib import admin

from .models import Question


class QuestionAdmin(admin.ModelAdmin):
    fields = ["pub_date", "question_text"]


admin.site.register(Question, QuestionAdmin)
```

You’ll follow this pattern – create a model admin class, then pass it as the second argument to ```admin.site.register()``` – any time you need to change the admin options for a model.

This particular change above makes the “Publication date” come before the “Question” field.

This isn’t impressive with only two fields, but for admin forms with dozens of fields, choosing an intuitive order is an important usability detail.

And speaking of forms with dozens of fields, you might want to split the form up into fieldsets in ```polls/admin.py```:

```python
from django.contrib import admin

from .models import Question


class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None, {"fields": ["question_text"]}),
        ("Date information", {"fields": ["pub_date"]}),
    ]


admin.site.register(Question, QuestionAdmin)
```