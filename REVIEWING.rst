Reviewing Tempest Code
======================

To start read the `OpenStack Common Review Checklist
<https://docs.openstack.org/infra/manual/developers.html#peer-review>`_


Ensuring code is executed
-------------------------

For any new or change to a test it has to be verified in the gate. This means
that the first thing to check with any change is that a gate job actually runs
it. Tests which aren't executed either because of configuration or skips should
not be accepted.

If a new test is added that depends on a new config option (like a feature
flag), the commit message must reference a change in DevStack or DevStack-Gate
that enables the execution of this newly introduced test. This reference could
either be a `Cross-Repository Dependency <https://docs.openstack.org/infra/
manual/developers.html#cross-repository-dependencies>`_ or a simple link
to a Gerrit review.


Execution time
--------------
While checking in the job logs that a new test is actually executed, also
pay attention to the execution time of that test. Keep in mind that each test
is going to be executed hundreds of time each day, because Tempest tests
run in many OpenStack projects. It's worth considering how important/critical
the feature under test is with how costly the new test is.


Unit Tests
----------

For any change that adds new functionality to either common functionality or an
out-of-band tool unit tests are required. This is to ensure we don't introduce
future regressions and to test conditions which we may not hit in the gate runs.
Tests, and service clients aren't required to have unit tests since they should
be self verifying by running them in the gate.


API Stability
-------------
Tests should only be added for a published stable APIs. If a patch contains
tests for an API which hasn't been marked as stable or for an API that which
doesn't conform to the `API stability guidelines
<https://wiki.openstack.org/wiki/Governance/Approved/APIStability>`_ then it
should not be approved.


Reject Copy and Paste Test Code
-------------------------------
When creating new tests that are similar to existing tests it is tempting to
simply copy the code and make a few modifications. This increases code size and
the maintenance burden. Such changes should not be approved if it is easy to
abstract the duplicated code into a function or method.


Tests overlap
-------------
When a new test is being proposed, question whether this feature is not already
tested with Tempest. Tempest has more than 1200 tests, spread amongst many
directories, so it's easy to introduce test duplication. For example, testing
volume attachment to a server could be a compute test or a volume test, depending
on how you see it. So one must look carefully in the entire code base for possible
overlap. As a rule of thumb, the older a feature is, the more likely it's
already tested.


Being explicit
--------------
When tests are being added that depend on a configurable feature or extension,
polling the API to discover that it is enabled should not be done. This will
just result in bugs being masked because the test can be skipped automatically.
Instead the config file should be used to determine whether a test should be
skipped or not. Do not approve changes that depend on an API call to determine
whether to skip or not.


Configuration Options
---------------------
With the introduction of the Tempest external test plugin interface we needed
to provide a stable contract for Tempest's configuration options. This means
we can no longer simply remove a configuration option when it's no longer used.
Patches proposed that remove options without a deprecation cycle should not
be approved. Similarly when changing default values with configuration we need
to similarly be careful that we don't break existing functionality. Also, when
adding options, just as before, we need to weigh the benefit of adding an
additional option against the complexity and maintenance overhead having it
costs.


Test Documentation
------------------
When a new test is being added refer to the :ref:`TestDocumentation` section in
hacking to see if the requirements are being met. With the exception of a class
level docstring linking to the API ref doc in the API tests and a docstring for
scenario tests this is up to the reviewers discretion whether a docstring is
required or not.

Release Notes
-------------
Release notes are how we indicate to users and other consumers of Tempest what
has changed in a given release. Since Tempest 10.0.0 we've been using `reno`_
to manage and build the release notes. There are certain types of changes that
require release notes and we should not approve them without including a release
note. These include but aren't limited to, any addition, deprecation or removal
from the lib interface, any change to configuration options (including
deprecation), CLI additions or deprecations, major feature additions, and
anything backwards incompatible or would require a user to take note or do
something extra.

.. _reno: https://docs.openstack.org/reno/latest/

Deprecated Code
---------------
Sometimes we have some bugs in deprecated code. Basically, we leave it. Because
we don't need to maintain it. However, if the bug is critical, we might need to
fix it. When it will happen, we will deal with it on a case-by-case basis.

When to approve
---------------
* Every patch needs two +2s before being approved.
* Its ok to hold off on an approval until a subject matter expert reviews it
* If a patch has already been approved but requires a trivial rebase to merge,
  you do not have to wait for a second +2, since the patch has already had
  two +2s.
