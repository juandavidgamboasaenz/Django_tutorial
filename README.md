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

## Introducing the Django Admin
