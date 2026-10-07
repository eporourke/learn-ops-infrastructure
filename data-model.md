\# Data Model



\## 1. Database Diagram



!\[Diagram](./data-model.png)



\## 2. Database Info



\*\*Database type:\*\* PostgreSQL 16



\*\*ORM:\*\* Django



\## 3. Model to Table Mapping



&#x20; Model Name           Table Name

&#x20; -------------------- --------------------------------

&#x20; NssUser              LearningAPI\_nssuser

&#x20; StudentPersonality   LearningAPI\_studentpersonality



\### NssUser



&#x20; Property Name   Column Name     Data Type

&#x20; --------------- --------------- -------------

&#x20; user            user\_id         integer

&#x20; slack\_handle    slack\_handle    varchar(55)

&#x20; github\_handle   github\_handle   varchar(55)



\### StudentPersonality



&#x20; Property Name           Column Name             Data Type

&#x20; ----------------------- ----------------------- ------------

&#x20; student                 student\_id              integer

&#x20; briggs\_myers\_type       briggs\_myers\_type       varchar(6)

&#x20; bfi\_extraversion        bfi\_extraversion        integer

&#x20; bfi\_agreeableness       bfi\_agreeableness       integer

&#x20; bfi\_conscientiousness   bfi\_conscientiousness   integer

&#x20; bfi\_neuroticism         bfi\_neuroticism         integer

&#x20; bfi\_openness            bfi\_openness            integer



\## 4. Relationship Examples



\*\*One-to-one\*\* (field name: student)



&#x20; ---------------------------------------------------------------------------------------

&#x20; Model Name           Table Name                       PK Column        FK Column

&#x20; -------------------- -------------------------------- ---------------- ----------------

&#x20; NssUser              LearningAPI\_nssuser              id               



&#x20; StudentPersonality   LearningAPI\_studentpersonality   id               student\_id

&#x20; ---------------------------------------------------------------------------------------



\*\*One-to-many\*\* (field name: course)



&#x20; Model Name     Table Name                 PK Column   FK Column

&#x20; -------------- -------------------------- ----------- -----------

&#x20; Course         LearningAPI\_course         id          

&#x20; CohortCourse   LearningAPI\_cohortcourse   id          course\_id



\*\*Many-to-many\*\* (field name: students)



&#x20; ------------------------------------------------------------------------------

&#x20; Model Name         Table Name                PK Column        FK Column

&#x20; ------------------ ------------------------- ---------------- ----------------

&#x20; NssUser            LearningAPI\_nssuser       id               



&#x20; StudentTeam        LearningAPI\_studentteam   id               



&#x20; NSSUserTeam        LearningAPI\_nssuserteam   id               student\_id,

&#x20; (junction)                                                    team\_id

&#x20; ------------------------------------------------------------------------------



