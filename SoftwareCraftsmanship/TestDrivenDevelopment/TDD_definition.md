
Test-Driven Development (TDD) is a programming discipline whereby programmers drive the design and implementation of their code by using unit tests. There are three simple laws:

1. You can't write any production code until you have first written a failing unit test.
2. You can't write more of a unit test than is sufficient to fail, and not compiling is failing.
3. You can't write more production code than is sufficient to pass the currently failing unit test.

The benefits when you follow these laws are :

* The debug time shrinking.
* You have a suite of of high-coverage tests that you trust, then you are not afraid to make changes, clean up the design and the architecture.
* These tests are small, focused ''documents'' that describe how to use the production code.
* They are ''design documents'' that are written in a language that the programmers understand; are unambiguous; are so formal that they ''execute''; and cannot get out of sync with the production code.
* Every module will be ''testable'' or ''decoupled'' by definition. This means your designs are more flexible, more maintainable, and just cleaner.



