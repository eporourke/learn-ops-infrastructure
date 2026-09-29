# Bug Fix: Student Assessment Retrieval

## The Bug
[Describe what caused the `'NoneType' object has no attribute 'user'` error.
Which variable was None? Under what condition?]

The variable that was missing was the instructor name.

## Log Statements Added
[Paste the log statements you added, including the log level and message for each.]

Just as the request is received, but before it begins:

      log.info("StudentAssessmentView create request received")

For each variable that is being defined once the request begins:

log.info("StudentAssessmentView create request received")
log.debug("Book ID assigned", book_id=book_id)
log.debug("Source URL assigned", source_url=source_url)
log.debug("Name assigned", name=name)
log.debug("Objectives assigned", objectives=objectives)

To show that it will be created if the following code checks out:

log.info("StudentAssessmentView create if")

For success:

log.debug("Student assessment created", student_assessment=student_assessment)

If it fails and the else block is triggered, this logs the error

log.error(
    "Error creating student assessment",
    student_id=request.data.get("studentId"),
    assessment_id=request.data.get("assessmentId"),
    error=str(ex),
    exc_info=True
)

## What the Logs Revealed
[What did you observe in the logs that pointed you to the root cause?]

The error referenced "get_instructor_username\n return obj.instructor.user.username", particularly this line - 

    def get_instructor_username(self, obj):
        """Return the instructor's username"""
        return obj.instructor.user.username

It was referencing something that hadn't been defined.

## The Fix
[Describe the change you made to resolve the bug. One or two sentences is fine.]

In the else block, the code that runs if it passes the other checks, I added a line that gets the instructor name that is linked to the studentassessment class(?)