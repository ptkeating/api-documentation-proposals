# Test Coverage Proposal

This is a proposal to improve the test coverage of the Upmind API.

## Purpose and Benefits

Test coverage guarantees the functionality of the codebase.

Increased test coverage permits full CI/CD as the system runs through regression testing with each deployment.

Contract testing makes the API sufficiently reliable for third-parties wishing to integrate.

## The Problem Space

The API is very large, with an endpoint count in excess of 1400. The test coverage of this API is limited.

Test coverage should be incrementally improved until it reaches an acceptable percentage of the codebase.

## Proposed Solution

**Steps**
1. Identify the business-critical elements of the codebase, here assumed to be Authentication, Billing, Authorization, Client Segregation and Client Configuration.
1. Create a priority list for these elements of the codebase.
1. For the first element, assumed to be Authentication, write user stories for a threat actor and then write acceptance tests to mimic some of the strategies such an actor might employ.
1. Use a code analysis tool like XDebug to see which parts of the system are touched by these tests. Write unit tests for each of the parts of the codebase that are not part of the Laravel framework or other dependencies.
1. If missing, write factories for the models that are utilised in these tests, e.g. User, Organisation etc.
1. If missing, write a simple seeder for these models to generate a random but representative set of objects.
1. Write some invalid input and happy-path user stories and the corresponding acceptance tests for these. Use a code analysis tool to identify all the units affected and write unit tests validating the functionality of these methods.
1. Repeat for each successive element of the codebase.
1. Add integration tests as and when the factories exist to create the required database state for these types of test.
1. For any new work added to the codebase, insist that there are tests to validate that the requirements of the feature request or bugfix issue have been met. Code without tests should not be merged.

