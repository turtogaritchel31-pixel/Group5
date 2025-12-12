# Schedule Conflict Detection - Developer Checklist

## 📋 Pre-Deployment Verification

### Code Review Checklist

- [ ] **Backend Controller** (`ScheduleController.php`)
  - [ ] `store()` method validates all 8 input fields
  - [ ] `store()` calls `detectConflicts()` before saving
  - [ ] `store()` returns HTTP 409 for conflicts
  - [ ] `store()` returns HTTP 200 for success
  - [ ] `detectConflicts()` has correct time overlap formula
  - [ ] `detectConflicts()` checks conflicts in priority order
  - [ ] `detectConflicts()` uses eager loading with relationships
  - [ ] Instructor names properly fetched from relationship
  - [ ] Year level names properly fetched from relationship

- [ ] **Frontend JavaScript** (`schedule.js`)
  - [ ] Form submit handler prevents default
  - [ ] CSRF token properly retrieved
  - [ ] Submit button disabled during processing
  - [ ] Loading spinner displayed
  - [ ] Response properly parsed as JSON
  - [ ] Success path: Modal closes, table refreshes
  - [ ] Error path: Modal stays open, error shown
  - [ ] Conflict details displayed when available
  - [ ] Button re-enabled in all code paths
  - [ ] No JavaScript console errors

### Database Checklist

- [ ] Indexes created (optional but recommended)
  ```sql
  CREATE INDEX idx_schedules_semester_user 
  ON schedules(semester, user_id);
  ```
- [ ] No old conflicting data in schedules table
- [ ] Foreign keys intact on all related tables
- [ ] User isolation working (each user only sees their schedules)

### Dependencies Checklist

- [ ] **Backend**
  - [ ] Laravel 11+ installed
  - [ ] PHP 8.0+ available
  - [ ] All models have proper relationships

- [ ] **Frontend**
  - [ ] jQuery included
  - [ ] SweetAlert2 included (via CDN or npm)
  - [ ] DataTables library available
  - [ ] Bootstrap CSS loaded
  - [ ] Font Awesome icons available

### Configuration Checklist

- [ ] CSRF protection enabled in `app/Http/Middleware/VerifyCsrfToken.php`
- [ ] Authentication working (`auth()->id()` returns user)
- [ ] Error logging configured
- [ ] Database connection working
- [ ] API accepts JSON responses

---

## 🧪 Testing Checklist

### Unit Tests (Backend Logic)

- [ ] Time overlap detection works correctly
- [ ] Day pattern mapping works (MWF, TTH, single days)
- [ ] Conflict priority order is correct
- [ ] Validation rejects invalid input
- [ ] Authorization prevents other users' schedules
- [ ] Eager loading works (no N+1 queries)

### Integration Tests

- [ ] Form submission → Database save works
- [ ] Conflict detection blocks duplicate schedules
- [ ] Conflict detection blocks instructor conflicts
- [ ] Conflict detection blocks year level conflicts
- [ ] Conflict detection blocks room conflicts
- [ ] Back-to-back schedules are allowed
- [ ] Different semesters don't conflict
- [ ] Different days don't conflict (when appropriate)

### Manual Browser Tests

- [ ] ✅ Open `/schedules` page
- [ ] ✅ Click "Add Schedule" button
- [ ] ✅ Modal opens with all dropdowns populated
- [ ] ✅ Fill form with valid data
- [ ] ✅ Click Submit
- [ ] ✅ Button shows spinner
- [ ] ✅ Request sent (check Network tab)
- [ ] ✅ Success message appears
- [ ] ✅ Modal closes
- [ ] ✅ Table refreshes with new schedule

### Conflict Detection Tests

- [ ] **Test 1: Instructor Conflict**
  - [ ] Add: Mon 10:00-11:00, Instructor A
  - [ ] Try: Mon 10:30-11:30, Instructor A
  - [ ] Result: ❌ Conflict shown

- [ ] **Test 2: Year Level Conflict**
  - [ ] Add: Tue 14:00-15:00, Year 1 Block A
  - [ ] Try: Tue 14:30-15:30, Year 1 Block A
  - [ ] Result: ❌ Conflict shown

- [ ] **Test 3: Room Conflict**
  - [ ] Add: Wed 09:00-10:00, Room 101
  - [ ] Try: Wed 09:30-10:30, Room 101
  - [ ] Result: ❌ Conflict shown

- [ ] **Test 4: Duplicate Schedule**
  - [ ] Add: Subject A, Instructor B, Year C, Course D, Mon
  - [ ] Try: Same exact schedule again
  - [ ] Result: ❌ Duplicate conflict shown

- [ ] **Test 5: Back-to-Back (Should Succeed)**
  - [ ] Add: Thu 09:00-10:00
  - [ ] Try: Thu 10:00-11:00
  - [ ] Result: ✅ Schedule added

- [ ] **Test 6: MWF Pattern**
  - [ ] Add: MWF 10:00-11:00
  - [ ] Try: Mon 10:30-11:30
  - [ ] Result: ❌ Conflict on Monday

- [ ] **Test 7: TTH Pattern**
  - [ ] Add: TTH 10:00-11:00
  - [ ] Try: Wed 10:00-11:00
  - [ ] Result: ✅ No conflict (different days)

- [ ] **Test 8: Different Semesters**
  - [ ] Add: 1st Semester, Instructor A, Mon 10:00-11:00
  - [ ] Try: 2nd Semester, Instructor A, Mon 10:00-11:00
  - [ ] Result: ✅ No conflict (different semesters)

### Error Handling Tests

- [ ] Missing required field → Validation error shown
- [ ] Invalid time format → Validation error shown
- [ ] CSRF token missing → 419 error
- [ ] Database error → 500 error with message
- [ ] Network timeout → Appropriate error shown

### Performance Tests

- [ ] ⚡ Form page loads in < 1s
- [ ] ⚡ Conflict detection completes in < 500ms
- [ ] ⚡ No more than 2 database queries per request
- [ ] ⚡ Table refresh completes in < 1s
- [ ] ⚡ No memory leaks (test multiple submissions)

---

## 📊 Monitoring Checklist

### Log Monitoring

- [ ] Check `storage/logs/laravel.log` for errors
- [ ] Monitor database query performance
- [ ] Track CSRF token validation failures
- [ ] Monitor failed validations
- [ ] Track 409 conflict responses

### Database Monitoring

```sql
-- Run these queries periodically
SELECT COUNT(*) FROM schedules;
SELECT COUNT(*) FROM schedules WHERE semester = '1st Semester';
SELECT instructor_id, COUNT(*) as count 
FROM schedules GROUP BY instructor_id ORDER BY count DESC;
```

### User Feedback

- [ ] Conflict messages are clear
- [ ] Success messages are encouraging
- [ ] Error messages are helpful
- [ ] No confusing duplicate messages
- [ ] Modal behavior is intuitive

---

## 🚀 Deployment Checklist

### Pre-Deployment

- [ ] All tests pass
- [ ] No console errors
- [ ] No database warnings
- [ ] Code reviewed by team member
- [ ] Backup created
- [ ] Rollback plan documented

### Deployment Steps

```bash
# 1. Backup current database
mysqldump -u user -p database > backup_$(date +%Y%m%d_%H%M%S).sql

# 2. Pull latest code
git pull origin main

# 3. Install dependencies
composer install

# 4. Run migrations (if any)
php artisan migrate

# 5. Add database indexes (optional)
# Copy SQL from database/indexes_and_performance.sql

# 6. Clear caches
php artisan cache:clear
php artisan config:clear
php artisan view:clear

# 7. Verify deployment
php artisan tinker
> Schedule::count()
> exit

# 8. Test in production
# - Open website
# - Try adding schedule
# - Check for errors in logs
```

### Post-Deployment

- [ ] Test scheduling in production
- [ ] Monitor error logs for 1 hour
- [ ] Verify database indexes are applied
- [ ] Test with real data
- [ ] Get user feedback
- [ ] Document any issues

---

## 📝 Code Quality Metrics

### Before Deployment, Verify:

- [ ] **Cyclomatic Complexity:** `detectConflicts()` is straightforward (low complexity)
- [ ] **Duplication:** No duplicate conflict checking code
- [ ] **Test Coverage:** 7+ test scenarios provided
- [ ] **Documentation:** 5 comprehensive documentation files
- [ ] **Code Style:** PSR-12 standards (Laravel convention)
- [ ] **Comments:** Complex logic is well-commented
- [ ] **Error Handling:** All error paths handled
- [ ] **Security:** User isolation, CSRF protection, input validation

---

## 🔐 Security Checklist

- [ ] CSRF token required on all form submissions
- [ ] User can only see/modify their own schedules
- [ ] Input validation prevents SQL injection
- [ ] Relationships are verified (foreign keys)
- [ ] No sensitive data in error messages (logs only)
- [ ] Rate limiting considered (if needed)
- [ ] XSS protection: HTML entities escaped

### SQL Injection Prevention

- [ ] Using Laravel query builder (not raw SQL)
- [ ] All user input validated before queries
- [ ] Foreign key constraints in place

### Authorization

- [ ] `where('user_id', auth()->id())` on all queries
- [ ] Users can't access other users' schedules
- [ ] Destroy method checks user_id

---

## 📚 Documentation Checklist

- [ ] README_CONFLICT_DETECTION.md created ✓
- [ ] CONFLICT_DETECTION_GUIDE.md created ✓
- [ ] QUICK_REFERENCE.md created ✓
- [ ] FLOW_DIAGRAMS.md created ✓
- [ ] IMPLEMENTATION_SUMMARY.md created ✓
- [ ] Code comments added to complex logic
- [ ] Function docstrings present
- [ ] Database indexes documented

---

## 🎯 Success Criteria

System is ready for production when:

✅ All tests pass  
✅ No console errors  
✅ All conflict types detected correctly  
✅ Performance acceptable (< 500ms)  
✅ Error messages helpful and clear  
✅ Documentation complete  
✅ Team reviewed and approved  
✅ Backup created  
✅ Monitoring set up  

---

## 📞 Support Contact

If issues arise:

1. Check **QUICK_REFERENCE.md** for troubleshooting
2. Review **CONFLICT_DETECTION_GUIDE.md** for details
3. Run `runAllTests()` in console
4. Check **storage/logs/laravel.log**
5. Review FLOW_DIAGRAMS.md for system understanding

---

## 📋 Sign-Off

- [ ] Developer: __________________ Date: ________
- [ ] Code Reviewer: __________________ Date: ________
- [ ] QA Tester: __________________ Date: ________
- [ ] Product Owner: __________________ Date: ________

---

**Ready to Deploy? ✅**

When all checkboxes are checked, system is production-ready!

---

*Generated: December 2024*  
*Version: 1.0*
