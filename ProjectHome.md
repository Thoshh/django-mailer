# django-mailer #

A reusable Django app for queuing and throttling of email sending, scheduled sending, consolidation of multiple notifications into single emails and logging of mail failures.

## Sample Use Cases ##

  * a site admin wants to send a one-off announcement to all users on a site
  * a site admin wants to send a one-off email to a subset of users (e.g. thank you note to users who completed a survey)
  * a site admin wants to send an email but wants to defer it to a particular date or time (e.g. send out at 9am tomorrow a reminder of the scheduled downtime at 10am)
  * a site admin wants to send a regular email to a subset of users (e.g. people who haven't logged in for over a month)
  * a user wants email notification when certain events specific to them happen (e.g. a friend accepts their invitation to join a group, they receive a private message)
  * a user wants email notification when certain events relevant to a subset of users happen (e.g. there has been a reply to a message board thread or blog entry that the user has subscribed to)
  * a user doesn't want individual emails for each notification but wants them in digest form (e.g. a daily digest of new posts or a weekly update on friends who have joined)

## Scope Questions ##

  * **django-mailer** should not manage actual subscriptions to events. Consider the "new reply to message board thread" use case. The overall site would need to keep track of which users subscribed to which threads and then when a thread is updated, **django-mailer** would be asked to send the same message to that list of users. But the [django-notification](http://code.google.com/p/django-notification/) app should keep track of which users subscribed to which threads and similar use cases.

## Other Requirements ##

  * it should be possible for the admin to set soft limits on number of messages to send per minute, maximum size of queue, maximum time a message can wait in queue
  * it should be possible for messages to have different priorities
  * failures at time of send attempt (i.e. exception thrown by SMTP client) should be logged and viewable by site admin
  * mail delivery failures should be captured either by polling a mailbox or by having the incoming mail directly piped to a Python script
  * it should be possible to set policy on what happens to future email addressed to a user whose email has previously failed
  * a system-wide opt-out should be available at the django-mailer level (i.e. a "don't email" list should be maintained and even if a delivery is requested to that address, it won't be sent)
  * both text and HTML email should be supported
  * use should be made of the Django template system for the mails themselves

**NOTE**: Now moved to http://github.com/jtauber/django-mailer/