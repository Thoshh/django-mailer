I'm a big believer in a step-by-step approach so I was going to
postpone support for deferred sending (e.g. send this email out at
3am tomorrow morning), recurring email (e.g. password change
reminders) and for box-carring (i.e. putting multiple notifications
in a single email) until later releases -- especially as I believe
these can be cleanly layered on top of the basic functionality.

With these features initially removed, the models I have in mind is
fairly straight forward.

The main model will just be a queue of fully resolved messages to
send as soon as possible. By "fully resolved" I mean that any
template processing will take place before the message gets added to
this model. So, in other words, the model will contain the literal
to, from, subject and message body as well as a datetime of when the
message was added to the queue. I'm also thinking of a three-way
priority -- high, medium, low (see below for what this means).

I'm thinking of implementing the "don't send" list at this level, so
even if you put an email to someone in this low-level queue, it won't
get sent if the email address is on the don't send list. From
experience, it's worth implementing this kind of opt-out support
right from the beginning, so the initial version will have this
second model, a simple list of email address not to send to, perhaps
along with a datetime of when the opt-out was added.

The third model for this initial version is a log. Regardless of
whether a send succeeds or not, it is logged in this model. A
separate process can be responsible for retries, rolling to file-
based logs, archiving or whatever. I am inclined to include
everything in this model that is included in the first model
(including the message body). In additional there would be a datetime
of when the send was attempted, a status code and some free-text
field to capture an exception.

I believe that all other features including retries, deferral and box-
carring can be implemented on top of this layer with little or no
modification.

One thing I haven't thought enough about is how this bottom layer
would handle (or integrate with handling of) mail delivery failures
that come back as email (rather than exceptions thrown by SMTP
subsystem). Things to consider are how to correlate the mail delivery
failure email and exactly what to correlate it with (the entry in the
log model?)

One thing the above models do not consider is the possibility of
multiple queues. I'm calling YAGNI on that at the moment.

So there would need to be one process that, when invoked would simply
go through the queue, attempting to send each email (after checking
the don't send list) and logging the result.

My thoughts about priorities are that the "sender process" should
first go through any "high" priority sorted by age (how long they'd
been in the queue), then any "medium" sorted by age, then any "low"
similarly sorted by age.

Finally there should be a view and template for showing basic status
of the django-mailer system.

Obviously later releases will add functionality on top, mostly
concerned with how to get things in the queue and when to put them
there.

So my initial plan is to implement what I've described above.

But there needs to still be, for the system to be usable for more
than just one-off emails to individuals, a basic mechanism for adding
bulk mail to the queue. The simplest case is one where an identical
(i.e. no templating) text email is sent to all users. Adding
templating support is only a small step from that, assuming that the
template just receives a user object in the context. Then what is
needed is a query set of what users to mail to. So bulk email is
achieved by running a choice of query set + template and adding each
result to the message queue in the layer described above.

So after a first pass of the lowest layer (described above) is done,
I plan to add this kind of bulk email support per the previous
paragraph.