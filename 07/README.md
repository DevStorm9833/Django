# Step 1

```python
# Authentication - Are you logged in?. Authorization - Are you allowed?
from django.contrib.auth.decorators import login_required, user_passes_test

from django.shortcuts import render, redirect, get_object_or_404
from django.contrib import messages
from django.utils import timezone
from .models import LeaveRequest, LeaveBalance
from .forms import LeaveReviewForm

def is_staff_user(user):
    """Check if user is staff"""
    return user.is_staff

@login_required
@user_passes_test(is_staff_user)
def pending_requests(request):
    """View all pending leave requests"""
    leave_requests = LeaveRequest.objects.filter(
        status='pending'
    ).order_by('-applied_on')
    
    context = {
        'leave_requests': leave_requests,
    }
    return render(request, 'leaves/pending_requests.html', context)


@login_required
@user_passes_test(is_staff_user)
def review_leave(request, leave_id):
    """Review and approve/reject leave request"""
    leave_request = get_object_or_404(LeaveRequest, id=leave_id)
    
    if request.method == 'POST':
        form = LeaveReviewForm(request.POST, instance=leave_request)
        if form.is_valid():
            leave = form.save(commit=False)
            leave.reviewed_by = request.user
            leave.reviewed_on = timezone.now()
            
            # Update leave balance if approved
            if leave.status == 'approved':
                try:
                    leave_balance = LeaveBalance.objects.get(
                        faculty=leave.faculty,
                        leave_type=leave.leave_type,
                        year=leave.start_date.year
                    )
                    leave_balance.used_leaves += float(leave.number_of_days)
                    leave_balance.save()
                    
                    messages.success(request, f'Leave request approved for {leave.faculty.user.get_full_name()}.')
                except LeaveBalance.DoesNotExist:
                    messages.warning(request, 'Leave approved but balance not found.')
            else:
                messages.info(request, f'Leave request rejected for {leave.faculty.user.get_full_name()}.')
            
            leave.save()
            return redirect('leaves:pending_requests')
    else:
        form = LeaveReviewForm(instance=leave_request)
    
    context = {
        'form': form,
        'leave_request': leave_request,
    }
    return render(request, 'leaves/review_leave.html', context)


@login_required
@user_passes_test(is_staff_user)
def all_leaves(request):
    """View all leave requests (admin dashboard)"""
    # Get filter parameters
    status_filter = request.GET.get('status', '')
    department_filter = request.GET.get('department', '')
    
    # Base query
    leave_requests = LeaveRequest.objects.all()
    
    # Apply filters
    if status_filter:
        leave_requests = leave_requests.filter(status=status_filter)
    if department_filter:
        leave_requests = leave_requests.filter(faculty__department=department_filter)
    
    # Order by latest
    leave_requests = leave_requests.order_by('-applied_on')
    
    # Get statistics
    total_requests = LeaveRequest.objects.count()
    pending_count = LeaveRequest.objects.filter(status='pending').count()
    approved_count = LeaveRequest.objects.filter(status='approved').count()
    rejected_count = LeaveRequest.objects.filter(status='rejected').count()
    
    # Get unique departments for filter
    from .models import FacultyProfile
    departments = FacultyProfile.objects.values_list('department', flat=True).distinct()
    
    context = {
        'leave_requests': leave_requests,
        'total_requests': total_requests,
        'pending_count': pending_count,
        'approved_count': approved_count,
        'rejected_count': rejected_count,
        'departments': departments,
        'current_status': status_filter,
        'current_department': department_filter,
    }
    return render(request, 'leaves/all_leaves.html', context)
```
