Advanced Usage
==============

Automatic Account Creation
--------------------------

``django-browserid`` will automatically create a user account for new
users. The user account will be created with the verified
email returned from the BrowserID verification service, and a URL safe
base64 encoded SHA1 of the email with the padding removed as the
username.

To provide a customized username, you can provide a different
algorithm via your settings.py::

   # settings.py
   BROWSERID_CREATE_USER = True
   def username(email):
       return email.rsplit('@', 1)[0]
   BROWSERID_USERNAME_ALGO = username

You can can provide your own function to create users by setting
``BROWSERID_CREATE_USER`` to a string path pointing to a function::

   # module/util.py
   def create_user(email):
       return User.objects.create_user(email, email)

   # settings.py
   BROWSERID_CREATE_USER = 'module.util.create_user'

You can disable account creation, but continue to use the
``browserid_verify`` view to authenticate existing users with the
following::

    BROWSERID_CREATE_USER = False


Custom Verification
-------------------

If you want full control over account verification, don't use
django-browserid's ``browserid_verify`` view. Create your own view and
use :func:`django_browserid.verify` to manually verify a
BrowserID assertion with something like the following::

   from django_browserid import get_audience, verify
   from django_browserid.forms import BrowserIDForm


   def myview(request):
      # ...
      if request.method == 'POST':
          form = BrowserIDForm(data=request.POST)
          if not form.is_valid():
              result = verify(form.cleaned_data['assertion'], get_audience(request))
              if result:
                  # check for user account, create account for new users, etc
                  user = my_get_or_create_user(result.email)

``result`` will be ``False`` if the assertion failed, or a dictionary
similar to the following::

   {
      u'audience': u'https://mysite.com:443',
      u'email': u'myemail@example.com',
      u'issuer': u'browserid.org',
      u'status': u'okay',
      u'expires': 1311377222765
   }

You are of course then free to store the email in the session and
prompt the user to sign up using a chosen identifier as their
username, or whatever else makes sense for your site.

.. autofunction:: django_browserid.verify

.. autofunction:: django_browserid.get_audience


Signals
-------

.. automodule:: django_browserid.signals
   :members:


Custom User Model
-----------------

Django 1.5 allows you to specify a custom model to use in place of the built-in
User model with the ``AUTH_USER_MODEL`` setting. ``django-browserid`` supports
custom User models, but you will most likely need to add a few extra
customizations to make things work properly:

* ``django_browserid.BrowserIDBackend`` has three methods that deal with User
  objects: ``create_user``, ``get_user``, and ``filter_users_by_email``. You may
  have to subclass ``BrowserIDBackend`` and override these methods to work with
  your custom User class.

* ``browserid_button`` assumes that your custom User class has an attribute
  called ``email`` that contains the user's email address. You can either add
  an email field to your model, or add a `property`_ to the model that returns
  the user's email address.

.. _property: http://docs.python.org/2/library/functions.html#property
