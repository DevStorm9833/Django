```python
django-admin startproject leave_management_system .
```
- Creates a Django configuration package

Why the `.` is important

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
Each file has one job.
```