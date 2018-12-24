# django 代码片段



##  ## Finding the user from the session

If the session still exists we can find it, unpickle the data it contains and get the user id. Here’s a short script to do just that：

```python
from django.contrib.sessions.models import Session
from django.contrib.auth.models import User

session_key = '8cae76c505f15432b48c8292a7dd0e54'

session = Session.objects.get(session_key=session_key)
uid = session.get_decoded().get('_auth_user_id')
user = User.objects.get(pk=uid)

print user.username, user.get_full_name(), user.email
```

and delete the session with save user：

```python

```

