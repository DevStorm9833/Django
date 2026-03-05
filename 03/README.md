

ORM = Object Relational Mapper 

1️⃣ Database independence

It maps:
- Python objects  ⇄  Database tables

2️⃣ Less bugs, more safety

- No SQL injection
- No Runtime query failures

Without ORM:
- You manually track table changes
- No history

With ORM, You get:
- Versioned DB schema
- Rollbacks (Think of rollback as the "Undo" button for SQL commands.)

```python
from django.db import models                            # gives ORM functionality , Without this → no database layer.
from django.contrib.auth.models import User             # built-in authentication user model. Contains: username, password (hashed), email, first_name, last_name, permissions
from django.core.validators import MinValueValidator    # Used for: MinValueValidator(1) - Value cannot be less than 1, Prevents negative leave days.
from django.utils import timezone

# Create your models here.

class FacultyProfile(models.Model):
    """
    Extended profile for faculty members
    Linked to Django's built-in User model
    """
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='faculty_profile') One User → One FacultyProfile, If user is deleted → faculty profile deleted.
    employee_id = models.CharField(max_length=20, unique=True)
    department = models.CharField(max_length=100)
    designation = models.CharField(max_length=100)
    phone_number = models.CharField(max_length=15, blank=True)
    date_of_joining = models.DateField()
    profile_picture = models.ImageField(upload_to='profile_pics/', blank=True, null=True)
    
    def __str__(self):
        return f"{self.user.get_full_name()} ({self.employee_id})"
    
    class Meta:
        verbose_name = "Faculty Profile"            # human-readable singular name of your model.Leave request → Leave Request 
        verbose_name_plural = "Faculty Profiles"    # you don’t define it, Django just adds “s” : FacultyProfile → Faculty profiles, Category → Categorys ❌ (wrong)
        ordering = ['user__first_name']             # Default sorting. __ (double underscore) means: FacultyProfile → User → first_name

class LeaveType(models.Model):
    """
    Different types of leaves available
    """
    name = models.CharField(max_length=50, unique=True)
    description = models.TextField(blank=True)
    max_days_per_year = models.IntegerField(validators=[MinValueValidator(1)])
    requires_document = models.BooleanField(default=False)
    is_active = models.BooleanField(default=True)
    
    def __str__(self):
        return self.name
    
    class Meta:
        verbose_name = "Leave Type"
        verbose_name_plural = "Leave Types"
        ordering = ['name']


class LeaveBalance(models.Model):
    """
    Track leave balance for each faculty member
    """
    faculty = models.ForeignKey(FacultyProfile, on_delete=models.CASCADE, related_name='leave_balances')
    leave_type = models.ForeignKey(LeaveType, on_delete=models.CASCADE)
    year = models.IntegerField()
    total_leaves = models.IntegerField(validators=[MinValueValidator(0)])
    used_leaves = models.DecimalField(max_digits=4, decimal_places=1, default=0)
    
    def remaining_leaves(self):
        """Calculate remaining leaves"""
        return self.total_leaves - float(self.used_leaves)
    
    def __str__(self):
        return f"{self.faculty.user.username} - {self.leave_type.name} ({self.year})"
    
    class Meta:
        verbose_name = "Leave Balance"
        verbose_name_plural = "Leave Balances"
        unique_together = ['faculty', 'leave_type', 'year']
        ordering = ['-year', 'leave_type']


class LeaveRequest(models.Model):
    """
    Individual leave request submitted by faculty
    """
    STATUS_CHOICES = [
        ('pending', 'Pending'),
        ('approved', 'Approved'),
        ('rejected', 'Rejected'),
        ('cancelled', 'Cancelled'),
    ]
    
    faculty = models.ForeignKey(FacultyProfile, on_delete=models.CASCADE, related_name='leave_requests')
    leave_type = models.ForeignKey(LeaveType, on_delete=models.PROTECT)
    start_date = models.DateField()
    end_date = models.DateField()
    number_of_days = models.DecimalField(max_digits=4, decimal_places=1)
    reason = models.TextField()
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='pending')
    applied_on = models.DateTimeField(auto_now_add=True)
    reviewed_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='reviewed_leaves')
    reviewed_on = models.DateTimeField(null=True, blank=True)
    admin_remarks = models.TextField(blank=True)
    supporting_document = models.FileField(upload_to='leave_documents/', blank=True, null=True)
    
    def __str__(self):
        return f"{self.faculty.user.username} - {self.leave_type.name} ({self.start_date} to {self.end_date})"
    
    def save(self, *args, **kwargs):
        """Override save to calculate number of days"""
        if self.start_date and self.end_date:
            delta = self.end_date - self.start_date
            self.number_of_days = delta.days + 1  # Include both start and end dates
        super().save(*args, **kwargs)
    
    class Meta:
        verbose_name = "Leave Request"
        verbose_name_plural = "Leave Requests"
        ordering = ['-applied_on']
        get_latest_by = 'applied_on'
```

```
from django.contrib import admin
from .models import FacultyProfile, LeaveType, LeaveBalance, LeaveRequest

# Register your models here.

@admin.register(FacultyProfile)
class FacultyProfileAdmin(admin.ModelAdmin):
    list_display = ['employee_id', 'get_full_name', 'department', 'designation', 'date_of_joining']
    list_filter = ['department', 'designation', 'date_of_joining']
    search_fields = ['employee_id', 'user__first_name', 'user__last_name', 'user__email']
    
    def get_full_name(self, obj):
        return obj.user.get_full_name()
    get_full_name.short_description = 'Full Name'


@admin.register(LeaveType)
class LeaveTypeAdmin(admin.ModelAdmin):
    list_display = ['name', 'max_days_per_year', 'requires_document', 'is_active']
    list_filter = ['is_active', 'requires_document']
    search_fields = ['name', 'description']


@admin.register(LeaveBalance)
class LeaveBalanceAdmin(admin.ModelAdmin):
    list_display = ['faculty', 'leave_type', 'year', 'total_leaves', 'used_leaves', 'get_remaining']
    list_filter = ['year', 'leave_type']
    search_fields = ['faculty__user__first_name', 'faculty__user__last_name', 'faculty__employee_id']
    
    def get_remaining(self, obj):
        return obj.remaining_leaves()
    get_remaining.short_description = 'Remaining Leaves'


@admin.register(LeaveRequest)
class LeaveRequestAdmin(admin.ModelAdmin):
    list_display = ['faculty', 'leave_type', 'start_date', 'end_date', 'number_of_days', 'status', 'applied_on']
    list_filter = ['status', 'leave_type', 'applied_on', 'start_date']
    search_fields = ['faculty__user__first_name', 'faculty__user__last_name', 'reason']
    readonly_fields = ['applied_on', 'number_of_days']
    date_hierarchy = 'applied_on'
    
    fieldsets = (                            # Instead of one long form → fields are grouped. Sections are created
        ('Leave Information', {
            'fields': ('faculty', 'leave_type', 'start_date', 'end_date', 'number_of_days', 'reason')
        }),
        ('Status', {
            'fields': ('status', 'reviewed_by', 'reviewed_on', 'admin_remarks')
        }),
        ('Documents', {
            'fields': ('supporting_document',),
            'classes': ('collapse',)
        }),
    )
    ```
