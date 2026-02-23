..
   # *******************************************************************************
   # Copyright (c) 2025 Contributors to the Eclipse Foundation
   #
   # See the NOTICE file(s) distributed with this work for additional
   # information regarding copyright ownership.
   #
   # This program and the accompanying materials are made available under the
   # terms of the Apache License Version 2.0 which is available at
   # https://www.apache.org/licenses/LICENSE-2.0
   #
   # SPDX-License-Identifier: Apache-2.0
   # *******************************************************************************

Guideline
=========
.. gd_guidl:: Verification Guideline
   :id: gd_guidl__verification_guide
   :status: valid
   :complies: std_req__isopas8926__445

   This guideline outlines the responsibilities and procedures for developers performing
   verification activities (testcase creation, inspection, and review) for documentation,
   code (like Rust and C/C++) elements of the project/platform and its tooling.

   Note that rust, python and gTest are used for test case creation.

General Principles
------------------

* **Verification is everyone's responsibility:** While dedicated testing responsible may exist, every developer is
  accountable for the quality and correctness of their code and related artifacts.
* **Early and often:** Verification activities should be integrated throughout the development lifecycle,
  not just at the end. This includes unit tests, integration tests, code reviews, and inspections.
* **Traceability:** All verification activities should be traceable to requirements, architectural design, etc.
* **Independence:** Where possible, verification activities should be performed by someone other than the original author of the code or documentation.
* **Documentation:** All verification activities and their results must be documented appropriately.

More details on the test strategy and execution can be found in the verification plan implemented by
:need:`wp__verification_plan` of the project.


Test Case Description
---------------------

A good test description clearly explains the purpose and scope of a test case.
It provides enough information for anyone (including someone unfamiliar with the system) to understand
what is being tested and how.

Basic qualities of a good test case description are that the test is:
  - Clear: Easy to understand and avoids ambiguity.
  - Specific: Provides enough detail to reproduce the test.
  - Measurable: Defines clear pass/fail criteria.
  - Complete: Covers all relevant aspects of the test.

Test specifications should follow :need:`gd_guidl__verification_specification`

Structuring of the Test Case
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To fulfill the demands of the work product :need:`wp__verification_plan` the
templates in :ref:`verification_process_reqs` shall be used and the :need:`gd_guidl__verification_specification`
should be followed . This includes general information and templates for the allowed programming languages.

Test Implementation Best Practices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following best practices provide guidance for writing high-quality, maintainable tests. These are
recommendations that can be deviated from when there is a good reason, such as improved readability or
reduced complexity for a specific case.

**Test Naming**

Test names should be descriptive and clearly indicate what is being tested. Naming conventions:

- Use a consistent naming style (e.g., PascalCase for C++/Rust, snake_case for Python)
- Test suite names should indicate the unit under test
- Test names should describe the scenario being tested (see :need:`gd_guidl__verification_specification` for structure guidance)
- Death tests (tests expecting termination) should be clearly identifiable by naming convention
  (see `Death Tests and Threads <https://github.com/google/googletest/blob/main/docs/advanced.md#death-tests-and-threads>`_)

Example naming patterns:

.. code-block:: text

   // C++ (GoogleTest)
   TEST(RuntimeWithInvalidConfigTest, WhenCallingInitThenAnErrorIsReturned)

   # Python (pytest)
   def test_runtime_with_invalid_config_returns_error():

   // Rust
   #[test]
   fn runtime_with_invalid_config_returns_error()

**Test Content Guidelines**

Each test should verify a single functionality:

- One function call with specific setup returning a particular value
- One function call with specific setup calling a mock function with expected arguments
- One function call with specific setup causing expected termination (death test)

Each test should have a single *semantic* assertion or expectation. Multiple related assertions are
acceptable when they verify a single logical outcome (e.g., checking both that a result has an error
and what that error code is).

**Test Constants and Data**

- If the specific value of a constant is relevant to the test being verified, define it within the test
- If the value is not relevant (just needs to be valid), use a shared constant or fixture
- Avoid using literals directly in assertions; use named constants for clarity
- Avoid global constant objects that use globals in their implementation to prevent
  `Static Initialization Order Fiasco <https://en.cppreference.com/w/cpp/language/siof.html>`_

**Test Fixtures and Setup**

- Keep fixtures focused and minimal; large fixtures may indicate poor architectural design
- Place fixture definitions close to the tests that use them
- Avoid deep inheritance hierarchies in fixtures
- Consider using a builder pattern for complex setup that needs to be configurable per test
  (see `ServiceDiscoveryClientFixture <https://github.com/eclipse-score/communication/blob/main/score/mw/com/impl/bindings/lola/service_discovery/test/service_discovery_client_test_fixtures.h>`_
  for an example implementation)

**Test File Organization**

- Group tests logically, typically one test file per unit under test
- When a test file becomes too large, split based on functionality being tested, not arbitrarily
  (see `service_discovery/client <https://github.com/eclipse-score/communication/tree/main/score/mw/com/impl/bindings/lola/service_discovery/client>`_
  for an example of splitting tests by feature)
- Maintain one test target per production code target in the build system for faster iteration

**Mocking Best Practices**

When using mock objects (see `Setting Expectations <https://google.github.io/googletest/gmock_cook_book.html#setting-expectations>`_
for GoogleMock details):

- Use mock defaults (e.g., ``ON_CALL`` in GoogleMock) for behavior that applies to most tests in a fixture
- Use explicit expectations (e.g., ``EXPECT_CALL``) only when the mock interaction is what you are testing
- When testing interfaces that require ownership transfer (e.g., ``unique_ptr``), consider using a
  facade pattern (see `InotifyInstanceFacade <https://github.com/eclipse-score/baselibs/blob/main/score/os/utils/inotify/inotify_instance_facade.h>`_
  for an example) or setting expectations before transferring ownership
- For complex assertions on mock parameters, consider using
  `custom matchers <https://google.github.io/googletest/reference/matchers.html#defining-matchers>`_


Verify Requirements Execution Work Flow
---------------------------------------

Simplified in a nutshell:

#. Implement test case
#. Link test case to requirements and specify metatags
#. Confirm requirement test coverage by creating linkage document
#. Set requirement attribute [testcovered=YES] during software build

More information on the concept of requirements verification can be found in :ref:`requirement_verification_workflow`

A more detailed description of how to link code to requirements is available here: :need:`gd_req__verification_link_tests`

Traceability matrix and consistency checks will be automatically established with tool support.

Two properties exists; one to show partial and one to show full coverage of a requirement.
For multiple test cases having a "partial coverage" a review has to be conducted to confirm
that a requirement is fully covered. The pull request description should indicate which requirements
are fully covered by the PR commits and which test cases are needed to fully cover the test case.
This is important, as multiple PRs may be needed to fully verify a single requirement.


Test case execution
-------------------

The execution of the tests is based on a full automation defined by build pipelines.
The analysis of the test results needs to be performed by the contributor.

In order to check the test results for the impact of a change or addition, it is recommended to
execute affected test cases locally upfront using the execution framework of the build tooling
following basically the steps the CI does locally.

Automated tests can also be executed locally, as the sources and binaries are available for re-execution.
Failing test cases during re-execution can be reported following the guide :need:`gd_temp__problem_template`.


Execution of manual test cases
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There may be the need for limited number of manually executed test cases.
These manually executed test cases are execution script driven, where a script guides through the
test cases and reports the result in the same logging format as automated tests do.
This enables parsable execution logs which can be used in automated test result collection.
Failing test cases during manual execution are reported by following the guide :need:`gd_temp__problem_template`.

Reporting of failing test cases
-------------------------------

Any failing test case requires an ISSUE.
The passing rate of safety-critical test cases need to be 100% in order to release the affected component.
In case of a lower pass rate than 100% for QM level tests, the :need:`rl__project_lead` and
:need:`rl__project_lead` can decide, if the platform is in a releasable state. The accepted minimal
path rate is defined in the :need:`wp__verification_plan`. Due to the high degree of automation, a
it is recommended that a path rate lower 95% is not acceptable.

In case an existing test case is failing due to regression in the CI, the respective issuer of the
PR in their role as :need:`rl__contributor` is responsible for fixing the test case as part of
respective PR.

Reuse of existing test cases
----------------------------

In case pre-existing test cases from components can be used, they have to be reviewed and checked
for their fit to the defined requirements. The test cases should get patch files to cover missing
specification parts following :need:`gd_guidl__verification_specification` and have the necessary
:need:`gd_req__verification_link_tests` followed. These patches are applied on top of the untouched actual
implementation of the software code.

Additionally needed test cases should be added as standalone parts. They are developed as any
other test case as part of the platform. If upstreaming of the newly created tests is judged as
useful, this shall be planned and added to the project milestone plan.

Verification types and methods
------------------------------

Verification types and methods are described in the :need:`gd_meth__verification_methods` and the
derivation techniques in :need:`gd_meth__verification_derivation`. The detailed method guideline
helps to get an understanding what the different methods and derivation techniques mean and how to
create test cases using the same.

Tailoring
=========
.. gd_guidl:: Verification Requirements Tailored
   :id: gd_guidl__verification_req_tailored
   :status: valid
   :complies: std_req__iso26262__software_945,
              std_req__iso26262__software_1045, std_req__iso26262__software_1046, std_req__iso26262__software_1047,
              std_req__iso26262__software_1141, std_req__iso26262__software_1142, std_req__iso26262__software_1143, std_req__iso26262__software_1144

   This part of the guideline links to all the requirements which are not fulfilled by the
   verification process. Make sure these are tailored out in the safety/security/quality plans
   for your project (documented in the PMP). Reasoning given below must be confirmed there.

   The reasoning is:

   - For SW requirement 945 & 1047 tests are executed in target environment regarding code and hardware architecture.
     An integrator of the software can re-execute them on source or unmodified binary in the product environment.
   - For SW requirement 1045 "Function and call coverage" are not considered explicitly due to lower target safety integrity level (ASIL_B).
   - For SW requirement 1046 the software is not a production release, but work for the system integrator or distributor with a production release.
   - For SW requirement 1141, 1143, 1144 the SW only points to AoUs, but will not be executed on the final product environment as part of the project scope.
   - For SW requirement 1142 there will be :need:`wp__verification_platform_int_test` available for re-use, but they are not supposed to deliver full coverage.
