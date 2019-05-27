# Python: Getting Started

A barebones Django app, which can easily be deployed to Heroku.

This application supports the [Getting Started with Python on Heroku](https://devcenter.heroku.com/articles/getting-started-with-python) article - check it out.

## Running Locally

Make sure you have Python 3.7 [installed locally](http://install.python-guide.org). To push to Heroku, you'll need to install the [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli), as well as [Postgres](https://devcenter.heroku.com/articles/heroku-postgresql#local-setup).

```sh
$ git clone https://github.com/heroku/python-getting-started.git
$ cd python-getting-started

$ python3 -m venv getting-started
$ pip install -r requirements.txt

$ createdb python_getting_started

$ python manage.py migrate
$ python manage.py collectstatic

$ heroku local
```

Your app should now be running on [localhost:5000](http://localhost:5000/).

## Deploying to Heroku

```sh
$ heroku create
$ git push heroku master

$ heroku run python manage.py migrate
$ heroku open
```
or

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)

## Documentation

For more information about using Python on Heroku, see these Dev Center articles:

- [Python on Heroku](https://devcenter.heroku.com/categories/python)

### Update Django Settings for Production:
1. Update `settings.py`:

    - Change `DEBUG = TRUE` to `DEBUG = FALSE`

    - Add `cfehome.herokuapp.com` to `ALLOWED_HOSTS`:
        ```
        ALLOWED_HOSTS = ['cfehome.herokuapp.com']
        ```

3. Create `heroku` Live Database:

    ```
    heroku addons:create heroku-postgresql:hobby-dev
    ```

4. Configure Live Database on `settings.py`:

    ```
    # keep this
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        }
    }

    # add this
    import dj_database_url
    db_from_env = dj_database_url.config()
    DATABASES['default'].update(db_from_env)
    ```
5. Commit:

    ```
    git add --all
    git commit -m "Update Django for database"
    ```


### Setup Static Files:

We suggest using [Amazon Web Service S3](http://www.kirr.co/exuykp/) for static files in general. However, if you want to use Heroku fr static files, do the following:

1. Install whitenoise and add to `requirements.txt`:
    ```
    pip install whitenoise
    pip freeze > requirements.txt
    ```
2. Update our Production Django Settings file created above called `production.py` (also in `local.py` & `base.py`):
    ```
    MIDDLEWARE = [
        'django.middleware.security.SecurityMiddleware',
        'whitenoise.middleware.WhiteNoiseMiddleware',
        'django.contrib.sessions.middleware.SessionMiddleware',
        'django.middleware.common.CommonMiddleware',
        'django.middleware.csrf.CsrfViewMiddleware',
        'django.contrib.auth.middleware.AuthenticationMiddleware',
        'django.contrib.messages.middleware.MessageMiddleware',
        'django.middleware.clickjacking.XFrameOptionsMiddleware',
    ]

    # Simplified static file serving.
    # https://warehouse.python.org/project/whitenoise/

    STATIC_URL = '/static/'

    STATICFILES_DIRS = (
        os.path.join(BASE_DIR, "static"),
    )

    STATIC_ROOT = os.path.join(BASE_DIR, "live-static-files", "static-root")

    STATICFILES_STORAGE = 'whitenoise.django.GzipManifestStaticFilesStorage'

    #STATIC_ROOT = "/home/cfedeploy/webapps/cfehome_static_root/"

    MEDIA_URL = "/media/"

    MEDIA_ROOT = os.path.join(BASE_DIR, "live-static-files", "media-root")
    ```

3. Using [Amazon Web Service S3](http://www.kirr.co/exuykp/) for static files?

    [Configuring S3](https://github.com/codingforentrepreneurs/Guides/blob/master/all/s3_staticfiles_django.md)

    Ensure you do disable collecstatic from running everytime you push to Heroku (which causes errors). Re-enable after you setup S3 in your Django Project.

    ```
    #disable collectstatic
    heroku config:set DISABLE_COLLECTSTATIC=1

    #enable collectstatic (if needed)
    heroku config:set DISABLE_COLLECTSTATIC=0
    ```
4. Run `collectstatic` locally:

    ```
    python manage.py collectstatic
    ```

5. Commit:
    ```
    git add --all
    git commit -m "Update Django for whitenoise static"
    ```

### Setup Django Project's Email with Gmail:
In your settings file(s) (`local.py`, `base.py`, `production.py`) put in the following:
```
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_HOST_USER = 'youremail@gmail.com' #my gmail username
EMAIL_HOST_PASSWORD = 'yourpassword' #my gmail password
EMAIL_PORT = 587
EMAIL_USE_TLS = True
DEFAULT_FROM_EMAIL = "Justin <hungrypy@gmail.com>"


ADMINS = [('Justin', EMAIL_HOST_USER)]
MANAGERS = ADMINS
```
If email is faiiling, then go to the folloing locations to unlock your gmail address:
- [https://www.google.com/settings/security/lesssecureapps](https://www.google.com/settings/security/lesssecureapps)

- [https://accounts.google.com/DisplayUnlockCaptcha](https://accounts.google.com/DisplayUnlockCaptcha)

### Add Custom Domain Name:

1. heroku domains [Heroku Docs](https://devcenter.heroku.com/articles/custom-domains):

    ```
    heroku domains
    heroku domains:add www.sporproject.com
    ```

2. Setup DNS for your Domain:

    | Type          | Host/name           |  Answer               |  TTL  |
    | ------------- |:-------------------:|:---------------------:|:-----:|
    | CNAME         | www.yourdomain.com  | cfehome.herokuapp.com |  300  |
    | CNAME         | blog.yourdomain.com | cfehome.herokuapp.com |  300  |

3. Update `production.py`:

    ```
    ALLOWED_HOSTS = ['cfehome.herokuapp.com', 'www.yourdomain.com']
    ```

4. Commit:

    ```
    git add --all
    git commit -m "Update Django for custom domain name"
    ```

### Push to Heroku
1. After all `git` commits, run:
    ```
    git push heroku master
    ```

2. Useful Django commands on Heroku:

    `heroku bash` opens the shell to do:

    ```
    python manage.py migrate
    python manage.py createsuperuser
    python manage.py shell
    ```

    **Shortcut commands**:

    `heroku run python manage.py migrate`

    `heroku run python manage.py createsuperuser`

    `heroku run python manage.py shell`

    `heroku restart`

3. When you make changes to Django on your local project:
    1. Ensure `makemigrations` is ran prior to deployment:
        ```
        python manage.py makemigrations
        ```
    2. View local changes:
        ```
        heroku local web
        ```
        open [http://localhost:5000/](http://localhost:5000/)

    3. Commit all changes:

        ```
        git add --all

        git commit -m "Update something"
        ```
    4. Push & Migrate:

        ```
        git push heroku master && heroku run python manage.py migrate
        ```
