# Data Model

## 1. Database Diagram

![Diagram](./data-model.png)

## 2. Database Info

**Database type:** PostgreSQL 16

**ORM:** Django

## 3. Model to Table Mapping

| Model Name | Table Name |
|------------|------------|
| NssUser | LearningAPI_nssuser |
| StudentPersonality | LearningAPI_studentpersonality |

### NssUser

| Property Name | Column Name | Data Type |
|---------------|-------------|-----------|
| user | user_id | integer |
| slack_handle | slack_handle | varchar(55) |
| github_handle | github_handle | varchar(55) |

### StudentPersonality

| Property Name | Column Name | Data Type |
|---------------|-------------|-----------|
| student | student_id | integer |
| briggs_myers_type | briggs_myers_type | varchar(6) |
| bfi_extraversion | bfi_extraversion | integer |
| bfi_agreeableness | bfi_agreeableness | integer |
| bfi_conscientiousness | bfi_conscientiousness | integer |
| bfi_neuroticism | bfi_neuroticism | integer |
| bfi_openness | bfi_openness | integer |

## 4. Relationship Examples

**One-to-one** (field name: student)

| Model Name | Table Name | PK Column | FK Column |
|------------|------------|-----------|-----------|
| NssUser | LearningAPI_nssuser | id | |
| StudentPersonality | LearningAPI_studentpersonality | id | student_id |

**One-to-many** (field name: course)

| Model Name | Table Name | PK Column | FK Column |
|------------|------------|-----------|-----------|
| Course | LearningAPI_course | id | |
| CohortCourse | LearningAPI_cohortcourse | id | course_id |

**Many-to-many** (field name: students)

| Model Name | Table Name | PK Column | FK Column |
|------------|------------|-----------|-----------|
| NssUser | LearningAPI_nssuser | id | |
| StudentTeam | LearningAPI_studentteam | id | |
| NSSUserTeam (junction) | LearningAPI_nssuserteam | id | student_id, team_id |