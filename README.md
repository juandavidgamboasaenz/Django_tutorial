# Django_tutorial
Writing your first Django app, part 1¶ Let’s learn by example.  Throughout this tutorial, we’ll walk you through the creation of a basic poll application.  It’ll consist of two parts:  A public site that lets people view polls and vote in them.  An admin site that lets you add, change, and delete polls.

### Create a directory djangotutorial with a project called mysite inside. 
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

### Running the development server
```bash
cd djangotutorial
python manage.py runserver
```

### 