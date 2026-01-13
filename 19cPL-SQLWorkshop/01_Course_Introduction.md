## Lesson 1 - Course Introduction (in which we meet the syllabus, the HR schema, and the sheer audacity of PL/SQL)

Welcome to Oracle Database 19c: PL/SQL Workshop. This is the introduction chapter, which means we are about to spend a suspiciously long time talking about what we are going to talk about.

By the end of this lesson, you should be able to:

- Discuss the goals of the course
- Describe the HR schema used throughout the course
- Identify the UI environments available for PL/SQL work
- Reference appendixes, documentation, and other resources

---

## 1. Course Objectives and Agenda (aka the polite warning label)

By the end of the course, you should be able to:

- Identify the programming extensions PL/SQL adds to SQL
- Write PL/SQL code that interfaces cleanly with the database
- Design anonymous blocks that execute efficiently
- Use PL/SQL programming constructs and conditional logic
- Handle runtime errors without rage quitting
- Describe and create stored procedures and functions
- Create packages
- Create triggers

Here is the roadmap, because we are civilized and also because this is a lot:

**Unit 1 - Introducing PL/SQL**
- Lesson 1: Introduction to PL/SQL
- Lesson 2: Declaring PL/SQL Variables
- Lesson 3: Writing Anonymous PL/SQL Blocks
- Lesson 4: Using SQL Statements in PL/SQL Blocks

**Unit 2 - Programming in PL/SQL**
- Lesson 5: Writing Control Structures
- Lesson 6: Working with Composite Data Types
- Lesson 7: Using Explicit Cursors

**Unit 3 - Working with PL/SQL Code**
- Lesson 8: Handling Exceptions
- Lesson 9: Introducing Procedures and Functions

**Unit 4 - Working with Subprograms**
- Lesson 10: Creating Stored Procedures
- Lesson 11: Creating Functions
- Lesson 12: Debugging Subprograms
- Lesson 13: Creating Packages
- Lesson 14: Working with Packages
- Lesson 15: Using Oracle-Supplied Packages
- Lesson 16: Using Dynamic SQL

**Unit 5 - Working with Triggers**
- Lesson 17: Creating Triggers
- Lesson 18: Creating Compound DDL and Event-Based Triggers

**Unit 6 - Advanced PL/SQL Code Management**
- Lesson 19: Design Considerations for PL/SQL Code
- Lesson 20: Using the PL/SQL Compiler
- Lesson 21: Managing Dependencies

---

## 2. The HR Schema (your new workplace sitcom)

This course uses the **HR (Human Resources) schema**, which ships with a set of tables and relationships that make for excellent PL/SQL practice.

At a high level:

- Employees work in departments
- Departments live in locations
- Locations are in countries
- Countries roll up into regions
- Managers are employees who also manage departments

So yes, it is a circle. Welcome to corporate life.

---

## 3. Course Agenda (five days, zero naps)

**Day 1**
- Introduction to PL/SQL
- Declaring variables
- Executable statements
- SQL inside PL/SQL blocks
- Control structures

**Day 2**
- Composite data types
- Explicit cursors
- Exceptions
- Intro to procedures and functions

**Day 3**
- Creating procedures
- Creating functions
- Debugging subprograms
- Creating packages

**Day 4**
- Working with packages
- Oracle-supplied packages
- Dynamic SQL
- Creating triggers

**Day 5**
- Compound DDL and event-based triggers
- Design considerations for PL/SQL
- PL/SQL compiler tuning
- Managing dependencies

---

## 4. Class Accounts (because HR has been cloned)

The HR schema is cloned into three accounts:

- `ora41`
- `ora61`
- `ora62`

You will use `ora41` and `ora61` for the full course. `ora62` is for extra practice.

Passwords are in the Activity Guide (usually around pages 5 or 6). Each machine has the same account setup.

---

## 5. Appendixes and Activity Guide (your emergency exits)

Additional appendixes include:

- Appendix A: Table Descriptions and Data
- Appendix B: Using SQL Developer
- Appendix C: Using SQL*Plus
- Appendix D: Commonly Used SQL Commands
- Appendix E: Managing PL/SQL Code
- Appendix F: Implementing Triggers
- Appendix G: Using DBMS_SCHEDULER and HTP Packages

The Activity Guide contains the hands-on practices and also the solutions, so you can verify your work without having to retype everything like it is 1997.

---

## 6. Tools and Environments (how you actually write the code)

**Oracle SQL Developer**
- Free graphical tool for database development
- Java-based, cross-platform
- Uses JDBC Thin driver
- Connects to Oracle 9.2.0.1 and later
- Works with Oracle databases on Cloud

**SQL*Plus**
- Command line tool, because sometimes you want to feel powerful and slightly haunted

**SQL Developer Web**
- Browser-based app via ORDS
- Requires schema-based authentication
- Not enabled in the lab, but good to know it exists

---

## 7. Documentation and Resources (the big bookshelf)

Key references include:

- Oracle Database PL/SQL Language Reference
- Oracle Database Reference
- Oracle Database SQL Language Reference
- Oracle Database Concepts
- Oracle Database PL/SQL Packages and Types Reference
- Oracle Database SQL Developer User's Guide
- Two-Day Developer's Guide
- Getting Started with Oracle Cloud

We will also point you to additional resources during the course, including:

- Oracle Database New Features (self study)
- What's new in PL/SQL on OTN
- SQL Developer home page and tutorial

---

## 8. Wrap-Up (we have a plan, now we do the thing)

You should now be able to:

- Discuss the goals of the course
- Describe the HR schema used in the course
- Identify available UI environments
- Reference appendixes, documentation, and other resources

That concludes the introduction lesson. Next up: actual PL/SQL, where the fun begins and the semicolons quietly wait for their moment.