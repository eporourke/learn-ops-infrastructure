django = "*


dj-rest-auth = "4.0.1"


django-allauth = "0.54.0"


Uses Digital Ocean to deploy DJango app with postgres, nginx from Github


Top level directories of learn-ops-api:

.github - this folder contains yml workflows that work with the github repository

.vscode - .json files related to vscode extension

config - front end config, based on nginx, called "Digital Ocean"


information to connect to front end learn ops api files?

LearningAPI - 

LearningPlatform
logs
LogViewer
static
staticFiles
templates

LearningAPI Directories:
fixtures
migrations
models
serializers
tests
views - class based or function based views? 


Step 1: Explore the learn-ops-api (Django) projects organization

Find answers to these questions using Django/python docs, make sure to use the versions you identified earlier in the course:

    List top level directories at the root of learn-ops-api. For each folder, explain what purpose it serves in this project.

    List top level directories inside learn-ops-api/LearningAPI. For each folder, explain what responsibility it owns.

    Open Pipfile. Why does this file exist? What is it's purpose?

    Look up django, djangorestframework, and django-allauth in the Pipfile. For each: what functonality does it provide. Why does the project import it?

    Open LearningAPI/decorators.py. What is a decorator and how is it used?

    Open LearningAPI/serializers.py. What do serializers do? Why does a Django REST API need serializers at all? Explain how it fits into the request response cycle.

    Open the models folder inside LearningAPI. What is a Model? Why does it exist? Pick one model and answer what real-world thing it represents and why the API needs to track that data.

    In Django, a view handles one URL and a viewset handles a full set of routes for a resource. Find one example of each: why would you choose a viewset over a plain view?

    Pick a serializer and find the model it belongs to. Explain how this serializer is used and what functionality it's providing.

    Django's pattern is Model-Template-View. This project does not have HTML templates. What takes the template's job here, and why does that make sense for this app?


API is using DJango
The view is what you see when you interact with the app
DJango unhelpfully has a different thing for this - a controller
is a view - I have a django app, code is organized in MTV structure

views vs. view sets