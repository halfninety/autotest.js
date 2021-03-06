Testmatic
===========

**Note: this project is still in development, and descriptions on this page are
more a plan than a reality.**

Automatic tests as a complement to unit tests.

This module provides the following functionalities:

1. By simply wrapping a function in a program, all calls to this function are
   recorded as test cases.
2. Test cases can also be added manually, unit testing style (with manually
   specified conditions and verified results). In other words, this module can
   be used solely as a helper for writing unit tests.
3. All added test cases can be run at once as part of a testing setup.
4. This module also acts as a test case manager. For example, when the code
   changes and a group of test cases are invalidated, these tests can be updated
   by issuing a single command instead of having to manually update each of them.


## Philosophy ##

This module is written to address the author's frustrations in the following
aspects:

1. Untested input combinations. Admit it, manually created unit tests may be
   able to cover every source line, but they can never cover all potentially
   interesting input combinations. This module reduces the chance of regressions
   by greatly expanding the program's test coverage on input combinations.
   What's better, the new input combinations are taken from real life usage.

2. Asserting a complex result. Traditionally, in unit testing, you execute an
   operation and then assert on its result. If this result is complex (contains
   many new and changed fields), you end up with many assert statements. With
   this module, you take a different approach: the result is presented to you
   for verification, and after that stored in the database, to be compared with
   future results on the same inputs. Now this approach isn't without its
   problems: it may be overly specific and easily broken, and sometimes you
   might prefer to have the assert statements in the test code for readability
   reasons. All these are valid concerns, but the power lies in choices.

3. Managing test cases. The problem with having many test cases is maintenance.
   When the code changes and many test cases need to be updated, you have to
   update them one by one, manually. With this module, however, you have choices.
   You can choose, for example, to manually verify the new results of a handful
   of tests, and let the other tests update automatically with the new results.


## Tests ##

Individual tests are divided into groups. There are two kinds of groups:
`static` and `dynamic`.

Static tests are created manually, by calling the `unitTest()` method, one test
at a time; while dynamic tests are created by calling the `wrapFunction()`
method, one group at a time. Tests in a dynamic group are created automatically
when the wrapped function is called.

A test is identified by its group name and its own id (must be unique inside the
group). A static test's id is specified by the user when creating it, while a
dynamic test's id is a hash computed from its conditions (a.k.a. inputs).

Group names are either specified by the user or deduced from the source code
(e.g. function name).

Conditions are the variable states that are required for the result to occur.
For a mathematical function, these are the arguments of the function, but it may
not be the case for a programming function. Therefore, the user is responsible
for guaranteeing that the specified conditions are both correct and complete.

When side effects are at play, e.g., when dealing with an external database, the
same input object may lead to different results, because data are fetched from
the external database; and reversely, different input objects may lead to the
same result, when difference in the connection object's internal states doesn't
affect the fetched data.

Static tests don't contain conditions. The user is responsible for guaranteeing
that the actual conditions are always the same when this test is run. This is
standard unit-testing style, in which the user builds a specific test setup and
asserts on the results.

On the other hand, tests in a dynamic group have differing conditions. The user
is responsible for guaranteeing that the same specified conditions always yield
to the same result (for the same code).

Each test has an `importance` which decides how we deal with it:

1. `high`. This test notifies the user when it's first added to the database,
           and fails whenever its result changes afterwards.
2. `medium`. This test fails whenever its result changes after it's added.
3. `low`. This test fails whenever its result changes when the major and minor
          versions of the program haven't changed. In other words, result
          changes after major or minor version changes are silently ignored.


## File Structure and Workflow ##

All files are JSON files, stored inside a user specified root directory, and can
be version controlled in either the main repository, or a sub-repository.

Static tests are stored inside the `static` subdirectory of the root directory,
while dynamic tests are stored inside the `dynamic` subdirectory.

Each test group is stored as a directory, using its name as the directory name;
and each test is stored as a JSON file under its group's directory, using its id
as the file name.

One particular trick is supported to allow the user to control the directory
structure: if a group name contains slashes in it, for example, `path/to/test`,
three nested directories `path`, `to`, and `test` are created instead of one,
and the group is stored inside directory `test`.

These files can be considered additional unit tests, and can use the same
workflow as normal unit tests. The only differences are that these tests can be
created automatically vs manually, and can be "created" by a deployment process
vs a developer.

But the same principals apply. For example, if two developers work on the same
part of code, and their changes make the same test to change in different ways,
then a manual merge of the test is needed.


## Library Methods ##

The following methods are provided:

1. `unitTest(name, id, res, opt)`. Create a static test, or check it if the test
                                   already exists. Default to high importance.
2. `wrapFunction(name, func, opt)`. Create a dynamic test group by wrapping a
                                    function. Default to low importance.
3. `runTestsInGroup(name, opt)`. Run all tests in a dynamic group at once.
                                 Currently only supports groups created using
                                 `wrapFunction()`.

## Command Line Tool ##

The following commands are supported:

1. `view`. View result(s) of a given test or test group.
2. `silence`. Declare a test or test group's current result(s) stale, so that
              these tests are silently updated without failing when results
              change.
3. `remove`. Remove the given tests and test groups.
