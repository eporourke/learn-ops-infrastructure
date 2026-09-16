# learn-ops-api: AI-Assisted Exploration

## 1. Top-level folders in `learn-ops-api`

| Folder | Why does this folder need to exist? |
|--------|-------------------------------------|
| `LearningAPI` | The main Django app. Holds the actual business logic of the platform: models, serializers, views, admin registration, migrations, signals, and tests. This is where most day-to-day feature work happens. |
| `LearningPlatform` | The Django *project* package (created by `django-admin startproject`). Holds project-wide configuration — `settings.py`, root `urls.py`, `wsgi.py`, and `test_settings.py` — that ties the installed apps together and configures the server. |
| `LogViewer` | A small, separate Django app that provides a simple web UI (`log_list.html`) for viewing application logs, rather than requiring direct server/file access. |
| `config` | Deployment and infrastructure configuration that lives alongside the code but isn't Django/Python itself: nginx server configs (`nginx.conf`, `api.conf`) and a YAML config file, used when running the app behind a reverse proxy in Docker/production. |
| `logs` | Output directory where the running application writes its log files (e.g. `learning_platform.json`). Exists so log output has a predictable, git-ignorable location on disk. |
| `static` | Source static assets maintained by developers (e.g. `custom_admin.css` to theme the Django admin). This is the "input" static directory Django's `collectstatic` reads from. |
| `staticfiles` | The "output" directory where `collectstatic` gathers all static assets (admin, django-rest-framework's browsable API assets, and this project's own) into one place to be served in production. |
| `templates` | Project-level HTML templates, here used to override/extend the default Django admin templates (`base.html`, `base_site.html`, `nav_sidebar.html`) for a customized admin site. |
| `.github` | GitHub Actions CI/CD workflow definitions (`main.yml`, `collectstatic.yml`, `seed.yml`) that automate testing, static file collection, and database seeding. |
| `.vscode` | Editor configuration shared with the team — recommended extensions and debugger launch configs — so everyone gets a consistent VS Code setup for this project. |
| `.git` | Git's internal repository data (commit history, refs, config) — not project code, but required for version control to function. |

**Folders inside `LearningAPI`**

| Folder | What responsibility does it own and why? |
|--------|------------------------------------------|
| `models` | Defines the database schema as Python classes (Django ORM). Split into sub-packages by domain — `coursework` (courses, projects, capstones, learning objectives), `people` (users, cohorts, assessments, notes, mentors), and `skill` (skill records, weights, learning records) — plus a top-level `tag.py`. Grouping by domain instead of one flat `models.py` keeps related models discoverable as the schema grows. |
| `serializers` | Converts model instances to/from JSON for the REST API (Django REST Framework). Each file handles one resource (e.g. `cohort_serializer.py`, `user_serializer.py`), defining what fields are exposed and how nested/related data is represented in requests and responses. |
| `views` | Contains the request-handling logic — one file (or sub-package) per resource/feature (e.g. `cohort_view.py`, `course_view.py`, `capstone_view.py`). Also holds `auth.py` and the `github`/`oauth2` sub-packages, which implement third-party login flows (GitHub OAuth) separately from ordinary resource views since they involve external API calls rather than simple CRUD. |
| `migrations` | Auto-generated, version-controlled history of every change to the database schema, in the order they must be applied. Lets the database be recreated or updated incrementally instead of relying on manual SQL. |
| `fixtures` | Static JSON snapshots of data for each model (e.g. `LearningAPI_cohort.json`), used to seed a database with realistic sample/test data (via the `seed.yml` CI workflow or `manage.py loaddata`) without needing a production data dump. |
| `tests` | Automated test suite (pytest/Django test cases), organized by feature (`test_cohort.py`, `test_course.py`, etc.), verifying that models, views, and endpoints behave correctly and catching regressions. |

## 2. Dependencies

**What is the Pipfile?**

`Pipfile` is the dependency manifest for `pipenv`, Python's equivalent of `package.json` in Node. It declares which packages the project needs, split into `[packages]` (runtime dependencies like `django`, `djangorestframework`, `gunicorn`, `psycopg2-binary`) and `[dev-packages]` (tools only needed locally/in CI, like `pytest`, `pylint`, `debugpy`), pins exact versions where compatibility matters (e.g. `django-allauth = "0.54.0"`) and leaves others open (`"*"`) to float to the latest compatible release. It also pins the required Python version (`3.11.11`) under `[requires]` and defines shortcut commands under `[scripts]` (e.g. `pipenv run migrate`). It exists so that anyone setting up the project — a teammate, CI, or a Docker build — installs the same set of packages in a reproducible way, rather than relying on whatever happens to already be installed. Its companion file, `Pipfile.lock`, records the exact resolved versions (including transitive dependencies) for fully reproducible installs.

**Key packages**

| Package | What functionality does it provide and why? |
|---------|---------------------------------------------|
| django | The core web framework the whole project is built on: the ORM (models/migrations), URL routing, the admin site, request/response handling, and the built-in auth system. It's the foundation everything else in `LearningAPI`/`LearningPlatform` plugs into. |
| djangorestframework | Layers REST API tooling on top of Django — serializers (`LearningAPI/serializers`), viewsets/generic views, a browsable API UI, pagination, and pluggable authentication/permission classes. In this project it's configured (`settings.py`) to use `TokenAuthentication` and require `IsAuthenticated` by default, and to paginate list responses with `LimitOffsetPagination`. It exists because Django alone doesn't provide REST conventions — DRF is the standard way to expose the models as a JSON API. |
| django-allauth | Handles authentication and account management, including third-party/social login. Here it's installed with `allauth.account` and `allauth.socialaccount` plus the `github` provider enabled, so users can sign in via GitHub OAuth (matching the `views/github` and `views/oauth2` view logic) instead of the project having to implement OAuth flows from scratch. |

## 3. Decorators, serializers, and models

### What does `decorators.py` do?

A decorator is a function that wraps another function to add behavior around it, without changing the wrapped function's own code. In Python it's applied with `@decorator_name` placed above a function (or, for methods on a class, via `django.utils.decorators.method_decorator`). The decorator receives the original function, defines a new function that runs some logic before/after (or instead of) calling it, and returns that new function in its place — so every call to the decorated function actually goes through the wrapper first.

`LearningAPI/decorators.py` defines two authorization decorators, `is_instructor()` and `is_staff()`. Each one:
1. Is a factory function (`is_instructor()`) that returns the actual `decorator`.
2. `decorator(func)` wraps the view method `func` in a `__wrapper` function.
3. `__wrapper` checks `request.user.groups.filter(name='Instructors').exists()` (or `'Staff'` for `is_staff`). If the user belongs to that group, it calls and returns the original view method (`func(request, *args, **kwargs)`) as normal. If not, it short-circuits and returns a `401 Unauthorized` DRF `Response` instead, so the original view logic never runs.

This is used in views to gate access without repeating the same permission-check `if` statement in every method. For example, in [course_view.py](../learn-ops-api/LearningAPI/views/course_view.py#L16-L27), `CourseViewSet.create` is decorated with `@method_decorator(is_instructor())`, so only users in the "Instructors" group can create a course — anyone else hitting that endpoint gets a `401` before `create`'s own code (saving the course, serializing it) ever executes.

### What is a serializer, and how does it fit the request/response cycle?

A serializer is a Django REST Framework class that converts between complex data — Django model instances/querysets — and simple, JSON-compatible formats (Python dicts/lists), and vice versa. It sits between the ORM and the outside world so a view never has to hand-build JSON or manually parse an incoming request body.

`LearningAPI/serializers` isn't a single `serializers.py` file but a package, with one module per resource (e.g. `cohort_serializer.py`, `user_serializer.py`) re-exported through `__init__.py` so they can be imported as `from LearningAPI.serializers import CohortSerializer`. Most, like [cohort_serializer.py](../learn-ops-api/LearningAPI/serializers/cohort_serializer.py), are simple `ModelSerializer`s:

```python
class CohortSerializer(serializers.ModelSerializer):
    class Meta:
        model = Cohort
        fields = '__all__'
```

`ModelSerializer` inspects the `Cohort` model and auto-generates fields matching its columns, so it knows how to turn a `Cohort` instance into JSON and how to validate/build a `Cohort` from incoming JSON, without the fields being listed by hand.

In the request/response cycle (used in [cohort_view.py](../learn-ops-api/LearningAPI/views/cohort_view.py#L89)):
1. **Incoming request** — a view receives a request; for a write (POST/PUT), it passes `request.data` into a serializer, which validates the raw JSON and, on `.save()`, creates/updates the model instance.
2. **Outgoing response** — for a read, the view fetches the model instance(s) from the database and passes them into a serializer (e.g. `CohortSerializer(cohort, context={'request': request})`). Accessing `serializer.data` produces plain Python data that DRF then renders to JSON for the `Response`.

So the serializer is the translation layer at both ends: it's what lets a view work in terms of Python model objects while the client only ever sees/sends JSON.

### What is a Django model, and what does one represent?

A Django model is a Python class that defines a database table: each class attribute (`models.CharField`, `models.DateField`, `models.BooleanField`, etc.) becomes a column, and each instance of the class is a row. Django's ORM uses the model to generate migrations (the actual SQL schema) and to let the rest of the codebase query and manipulate that data as normal Python objects instead of writing raw SQL.

[`LearningAPI/models/people/cohort.py`](../learn-ops-api/LearningAPI/models/people/cohort.py) defines the `Cohort` model, which represents a real-world cohort — one group of students going through the program together on a shared schedule (e.g. "day cohort 55"). Its fields capture exactly what the school needs to know about that group: a `name` and `slack_channel` to identify and communicate with it, `start_date`/`end_date` and `break_start_date`/`break_end_date` to know when it runs, and `active` to flag whether it's currently in session. It also adds computed properties like `coaches` (looked up via the related `NssUserCohort` model) and `is_active_on_date()`, which derive useful information from that stored data rather than storing it redundantly.

The API needs to track cohorts because almost everything else in the system — which students belong to which group, which courses/assessments apply to them, which Slack channel to notify, whether they're currently active — is scoped to a cohort. Without a `Cohort` model, there'd be no way to group students, schedule courses per group, or answer "is this cohort on break right now?" (`is_active_on_date`), which other features (like course/assessment views) depend on.

## 4. Views, viewsets, and the MTV pattern

### Views vs. viewsets

| Type | Example class | When to use it |
|------|--------------|----------------|
| View | `popular_queries` in [popular_query.py](../learn-ops-api/LearningAPI/views/popular_query.py#L16-L17) — a plain function decorated with `@api_view(['GET'])` | For a single, one-off endpoint that doesn't map cleanly to a model's CRUD operations — here, reading a cached "popular searches" value out of Valkey rather than reading/writing a database row. |
| ViewSet | `CohortViewSet` in [cohort_view.py](../learn-ops-api/LearningAPI/views/cohort_view.py#L26) | For a resource that needs the standard set of operations (list, retrieve, create, update, delete) around one model — here, `Cohort`. |

A plain view (whether a function with `@api_view` or a class-based `APIView`) handles exactly the URL(s) it's wired to and whatever HTTP methods it declares — `popular_queries` only answers `GET /queries/popular`, and is mapped with one explicit `path()` in [urls.py](../learn-ops-api/LearningPlatform/urls.py#L60). A `ViewSet`, by contrast, is a class that groups several related actions (`list`, `retrieve`, `create`, `update`, `destroy`, plus any custom `@action` methods) together for one resource, and gets registered once with a DRF `router` (`router.register(r'cohorts', views.CohortViewSet, 'cohort')` in [urls.py](../learn-ops-api/LearningPlatform/urls.py#L33)) — the router then auto-generates the full set of RESTful URLs (`GET/POST /cohorts`, `GET/PUT/DELETE /cohorts/{id}`, etc.) instead of them being hand-written.

**When to choose which:** reach for a `ViewSet` (or `ModelViewSet`, which auto-implements the CRUD methods from a serializer + queryset) whenever an endpoint is really "the standard operations on a model" — it's less code and keeps routing consistent, which is why the vast majority of `LearningAPI/views` are viewsets. Reach for a plain view when the endpoint doesn't represent CRUD on a single resource at all — a cache lookup (`popular_queries`), a notification trigger (`notify`), or another action-shaped operation where forcing it into `list`/`create`/etc. would be artificial.

### What replaces templates and why?

**Django's Model-Template-View (MTV) pattern**

Django's own take on MVC is called MTV: the **Model** owns the data and database access, the **View** contains the request-handling logic (what to fetch, what to do, which data to send back), and the **Template** is what turns that data into the actual HTTP response body — traditionally an HTML page, built by taking a `.html` file with placeholders and filling them in with context data from the view (`render(request, 'page.html', {'cohort': cohort})`). The view's job ends at deciding *what* data to send back; the template's job is deciding *how* to present it to whoever's on the other end of the request.

**What takes the Template's role in this project**

In `LearningAPI`, that presentation job is done by the **serializers**, not HTML templates. A view still decides what data to fetch and hands it off, exactly like in MTV — but instead of calling `render()` with a `.html` file, it calls a serializer (e.g. `CohortSerializer(cohort, context={'request': request})`) and returns `serializer.data` as JSON via `Response(...)`. The serializer plays the same structural role a template plays: it's the layer between the view's raw model data and the format actually sent over HTTP.

**Why that makes sense with no HTML templates**

`LearningAPI` is a REST API — its consumer is `learn-ops-client` (or any other frontend), not a browser rendering server-generated pages. There's no HTML page to fill in, so an HTML template would have nothing meaningful to do. What the client actually needs instead is a *data contract*: a predictable JSON shape it can parse. That's exactly what a serializer defines — field names, types, and nesting — while also handling the reverse direction (validating incoming JSON on writes), which a template can't do at all since templates are output-only. So the "presentation" step doesn't disappear in a REST API, it just changes from "render data into markup for a human to view" to "render data into JSON for a client program to parse" — and a serializer is built for exactly that.

(The project does still use real HTML templates in a couple of places — the Django admin site's `templates/admin/*.html`, and `LogViewer`'s `log_list.html` — but those are internal, browser-facing tools, not part of the `LearningAPI` REST endpoints themselves.)