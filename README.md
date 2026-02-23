# Django

Django is a “high-level Python web framework.” It is designed to help developers build web applications fast and correctly

Django uses the **MVT (Model-View-Template)** design pattern:

- Model (Data Layer)
  - Maps directly to database tables
- View (Control + Logic Layer)
  - Receives HTTP requests
- Template (Presentation Layer)
  - HTML + Django template language

**Real-World Analogy**

Faculty applies for leave:

- Model → stores leave request
- View → validates dates, checks balance, saves request
- Template → shows “Pending approval”

**Why Django**

- Authentication, admin panel, ORM, security.
- Reduces SQL injection risk
- Improves portability across databases
- Django has a migration system - version control for your database schema.
- You’d take weeks to build this manually.

**Virtual env:**

- Isolates dependencies
- Prevents version conflicts
- Makes deployments reproducible

Without venv:
- You’re mixing kitchen spices with chemistry lab chemicals.

Once activated, (venv) means:
- pip installs go here
- Django lives here
- System Python stays clean

### Instagram was originally built using Django. Other Eg. Pinterest, NASA
