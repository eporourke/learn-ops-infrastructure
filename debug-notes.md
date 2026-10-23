# Debug Notes: The Invisible Save

## 1. Bug report summary

When entering and trying to update the Slack ID in the student interface, it does not update the ID. Despite the failure, there's no error. This is the same behavior I observed.

## 2. What the debugger showed

| Question | Answer |
|----------|--------|
| Value of `student.slack_handle` before line 87 | `None` |
| Value of `student.slack_handle` after line 88 | `'U0262U94LTU'` |
| Next line executed after line 90 | Line 92 (`return Response(...)`) |

## 3. Root cause

The code changed 'student.slack_handle', but did not save that change to the database before returning a response.
## 4. The fix

Added `student.save()` to the `update()` method in `LearningAPI/views/student_view.py`, after the attribute assignments and before the response is returned.

## 5. What the test verifies

The test verifies that the value the database previously had is different from the new saved value.
