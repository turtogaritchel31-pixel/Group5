# Schedule Conflict Detection - Quick Reference

## 🚀 Quick Start

### Backend Setup (Laravel Controller)
```php
// File: app/Http/Controllers/ScheduleController.php
public function store(Request $request)
{
    $validated = $request->validate([
        'subject_id' => 'required|exists:subjects,subject_id',
        'YearLevel_id' => 'required|exists:yearlevels,yearlevel_id',
        'room_id' => 'required|exists:classrooms,room_id',
        'instructor_id' => 'required|exists:instructors,instructor_id',
        'course_id' => 'required|exists:courses,course_id',
        'semester' => 'required|string',
        'time_in' => 'required|date_format:H:i',
        'time_out' => 'required|date_format:H:i|after:time_in',
        'Day' => 'required|string',
    ]);

    $conflictCheck = $this->detectConflicts($validated, auth()->id());
    
    if (!$conflictCheck['success']) {
        return response()->json($conflictCheck, 409);
    }

    Schedule::create(array_merge($validated, ['user_id' => auth()->id()]));
    return response()->json(['success' => true, 'message' => 'Added!']);
}
```

### Frontend Setup (JavaScript)
```javascript
// File: public/js/schedule.js
form.addEventListener('submit', e => {
    e.preventDefault();
    const submitBtn = form.querySelector('button[type="submit"]');
    submitBtn.disabled = true;

    fetch('/schedules', {
        method: 'POST',
        headers: {
            'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content,
            'Accept': 'application/json'
        },
        body: new FormData(form)
    })
    .then(res => res.json())
    .then(data => {
        if (data.success) {
            // Success notification
            Swal.fire({icon: 'success', text: data.message, timer: 2500});
            modal.style.display = 'none';
            loadSchedules();
        } else {
            // Show conflict details
            Swal.fire({
                icon: 'error',
                title: 'Conflict!',
                html: data.message
            });
        }
        submitBtn.disabled = false;
    });
});
```

---

## 📋 Conflict Type Priority

| Priority | Type | Trigger | Message |
|----------|------|---------|---------|
| 🔴 1 | Duplicate | Exact same schedule | "This exact schedule already exists" |
| 🟠 2 | Instructor | Instructor overlap | "Instructor **{Name}** has a class at this time" |
| 🟡 3 | Year Level | Year level overlap | "Year Level **{Name}** has a class at this time" |
| 🟢 4 | Room | Room overlap | "This classroom is already occupied" |

---

## 🔍 Time Overlap Algorithm

```
OVERLAP IF:
    (new_time_in < existing_time_out) AND 
    (new_time_out > existing_time_in)

EXAMPLES:
Existing: 10:00-11:00
├─ New: 09:30-10:30 → CONFLICT ✗
├─ New: 10:30-11:30 → CONFLICT ✗
├─ New: 09:00-10:00 → NO CONFLICT ✓ (boundary)
├─ New: 11:00-12:00 → NO CONFLICT ✓ (boundary)
└─ New: 10:00-11:00 → NO CONFLICT ✓ (exact boundary)
```

---

## 📅 Day Pattern Mapping

| Input | Maps To | Usage |
|-------|---------|-------|
| `Mon` | `['Mon']` | Single Monday class |
| `MWF` | `['Mon', 'Wed', 'Fri']` | Three-day class |
| `TTH` | `['Tue', 'Thu']` | Two-day class |
| `Sun` | `['Sun']` | Sunday class |

---

## 🛠️ Development Tips

### Enable Debug Mode
```php
// In detectConflicts method, before return statement
\Log::debug('Conflict detected', [
    'new_schedule' => $newSchedule,
    'existing' => $existing->toArray(),
    'conflict_type' => $conflictType
]);
```

### Test in Browser Console
```javascript
// Run all tests
runAllTests();

// Test specific scenario
testInstructorConflict();
testTimeOverlapLogic();
testDayOverlapLogic();
```

### Monitor Database Performance
```sql
-- Check active schedules
SELECT COUNT(*) FROM schedules WHERE semester = '1st Semester';

-- Check for actual conflicts (for debugging)
SELECT i.Name, COUNT(*) 
FROM schedules s
JOIN instructors i ON s.instructor_id = i.instructor_id
GROUP BY s.instructor_id
HAVING COUNT(*) > 5;  -- Instructors with many classes
```

---

## 📊 Response Format Reference

### Success (200 OK)
```json
{
    "success": true,
    "message": "Schedule added successfully!"
}
```

### Conflict (409 Conflict)
```json
{
    "success": false,
    "conflict_type": "instructor",
    "message": "Instructor <strong>John Doe</strong> has a class at this time",
    "conflicting_schedule": {
        "instructor": "John Doe",
        "time_in": "10:00",
        "time_out": "11:30",
        "day": "Mon"
    }
}
```

### Validation Error (422 Unprocessable Entity)
```json
{
    "message": "The given data was invalid.",
    "errors": {
        "time_in": ["The time in field is required."]
    }
}
```

---

## 🧪 Test Checklist

- [ ] Instructor can't be double-booked
- [ ] Year level can't be double-booked
- [ ] Classroom can't be double-booked
- [ ] Duplicate schedules are rejected
- [ ] Back-to-back times are allowed (10:00-11:00, 11:00-12:00)
- [ ] MWF pattern conflicts detected on Mon/Wed/Fri
- [ ] TTH pattern conflicts detected on Tue/Thu
- [ ] Different semesters don't conflict
- [ ] Same hour but different days don't conflict
- [ ] Form button shows loading state
- [ ] Success message auto-closes after 2.5s
- [ ] Conflict keeps modal open for editing
- [ ] Table refreshes after successful add

---

## 🔧 Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| Always returns conflict | Eager loading wrong | Check `.with()` relationships |
| Button not re-enabling | Exception in JS | Check `.finally()` block |
| CSRF token error | Missing in form | Add `@csrf` to blade template |
| Query too slow | No indexes | Run SQL index creation script |
| Wrong conflict type | Logic order | Check priority in `detectConflicts` |

---

## 📁 File Structure

```
Project Root/
├── app/Http/Controllers/
│   └── ScheduleController.php          ← Main logic
├── app/Models/
│   ├── Schedule.php
│   ├── Instructor.php
│   ├── YearLevel.php
│   └── Classroom.php
├── public/js/
│   ├── schedule.js                     ← AJAX handler
│   └── conflict-detection-tests.js     ← Test utilities
├── resources/views/
│   └── class_schedule/
│       └── class_schedule.blade.php    ← Form UI
├── database/
│   └── indexes_and_performance.sql     ← Optimization
├── routes/
│   └── web.php                         ← Routes
└── CONFLICT_DETECTION_GUIDE.md         ← Full docs
```

---

## 🎯 Common Use Cases

### Scenario 1: Add Class
```
1. User fills form (Subject, Instructor, Room, Time, Day)
2. Clicks "Submit"
3. AJAX sends to /schedules POST
4. Backend validates then checks conflicts
5. If OK: Save to DB, return success
6. If Conflict: Return conflict type + message
7. Frontend shows appropriate notification
```

### Scenario 2: Bulk Import
```
1. Admin uploads CSV with 50+ schedules
2. Backend iterates each row
3. Validates and checks conflicts
4. Saves valid ones, logs failures
5. Returns report: "45 added, 5 conflicts"
```

### Scenario 3: Reschedule
```
1. User clicks "Edit" on existing schedule
2. Modal opens with current data
3. User changes time/day
4. Conflict check runs on new values
5. If OK: Saves, updates table
6. If Conflict: Shows alternatives
```

---

## 📞 Support

For issues:
1. Check browser console for JS errors
2. Check Laravel logs: `storage/logs/laravel.log`
3. Run test suite: `runAllTests()` in console
4. Review CONFLICT_DETECTION_GUIDE.md for detailed docs

---

**Last Updated:** December 2024  
**Version:** 1.0 (Production Ready)
