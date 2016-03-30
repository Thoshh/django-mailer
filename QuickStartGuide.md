# Quick Start Guide #

django-mailer is asynchronous so in addition to putting mail on the queue you need to periodically tell it to clear the queue and actually send the mail.

The latter is done via a command extension.

## Putting Mail On The Queue ##

Because django-mailer currently uses the same function signature as django's core mail support you can do the following in your code

```
from django.conf import settings

# favour django-mailer but fall back to django.core.mail
if "mailer" in settings.INSTALLED_APPS:
    from mailer import send_mail
else:
    from django.core.mail import send_mail
```

and then just call `send_mail` like you normally would in django:

```
send_mail(subject, message_body, settings.DEFAULT_FROM_EMAIL, recipients)
```

Additionally you can send all the admins as specified in the `ADMIN`
setting by calling::

```
mail_admins(subject, message_body)
```

or all managers as defined in the `MANAGERS` setting by calling::

```
mail_managers(subject, message_body)
```

## Clear Queue With Command Extensions ##

With mailer in your INSTALLED\_APPS, there will be two new manage.py commands you can run:

`send_mail` will clear the current message queue. If there are any failures, they will be marked deferred and will not be attempted again by `send_mail`.

`retry_deferred` will move any deferred mail back into the normal queue (so it will be attempted again on the next `send_mail`.

I use the following crontab entries on `pinax.hotcluboffrance.com`:

```
* * * * * (cd $PINAX; /usr/local/bin/python2.5 manage.py send_mail >> $PINAX/cron_mail.log 2>&1)
0,20,40 * * * * (cd $PINAX; /usr/local/bin/python2.5 manage.py retry_deferred >> $PINAX/cron_mail_deferred.log 2>&1)
```

This attempts to send mail every minute with a retry on failure every 20 minutes.

`manage.py send_mail` uses a lock file in case clearing the queue takes longer than the interval between calling `manage.py send_mail`.