```python
django-admin startproject leave_management_system .
```
- Creates a Django configuration package

### Why the `.` is important

Without the dot:
```
faculty-leave-management/
└── leave_management_system/
    └── leave_management_system/
```

Let’s trace a real request:
```
Browser → urls.py → views.py → models.py → views.py → template → Browser

Example:
User applies leave
URL maps to apply_leave view
View checks LeaveBalance (model)
View saves LeaveRequest
View renders template
User sees confirmation
```

---

# Step 7

### Why You MUST Register 'leaves'

You created an app using:
```
python manage.py startapp leaves
```
That only creates files. Django does NOT auto-detect apps.

Until you add:
```
'leaves',
```

Order in INSTALLED_APPS Matters
```
# Django core apps
django.contrib.*

# Third-party apps
rest_framework
crispy_forms

# Your apps
leaves
accounts
```
This avoids subtle bugs later.

---

# Step 8:

- A view is just a Python function that: Takes a request & Returns a response

`request` contains:
- HTTP method (GET / POST)
- Logged-in user
- Cookies
- Session data
