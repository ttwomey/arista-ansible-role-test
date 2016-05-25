Arista Roles for Ansible Development Guidelines
===============================================

#### Table of Contents

1. [Running Role Tests] (#running-role-tests)
    * [Overview] (#overview)
    * [Details] (#details)
2. [Developing Arista Roles For Ansible] (#developing-arista-roles-for-ansible)
    * [Role Development Guidelines] (#role-development-guidelines)
    * [Role Test Development] (#role-test-development)

Running role tests
------------------

#### Overview

To execute a role test suite:

- Update the test/fixtures/hosts file with the name(s) of your test devices.
- Update the test/arista-ansible-role-test/group_vars/all.yml file with
  the connection information for your devices.
- Execute `make tests` from the root of the role directory.

#### Details

This test framework should be used in a cloned copy of an Arista
ansible-eos-* Ansible role. The framework will *not* execute properly in an
ansible-galaxy installation of the role.

The framework should be loaded as a submodule within the Arista role,
in the test directory. If the test/arista-ansible-role-test directory of
the role is empty, update the role repository by issuing the following
commands from within the role repository:

    git submodule init
    git submodule update

Alternatively, the repository may be cloned using the --recursive option,
which will pull in the arista-ansible-role-test submodule as part of the
clone process.

To use the test framework in your local environment, you will first need
to update the test/fixtures/hosts file (in the role repository) and the
test/arista-ansible-role-test/group_vars/all.yml file (in the test framework
repository). The hosts file should list your testing devices under the
[test_hosts] section. The all.yml file should reflect the proper connection
parameters for your devices under the provider mapping.

Once the files have been updated for your local environment, execute
`make tests` from the root of the role directory to run the test suite against
the role.

To run a specific set of tests from the test suite, set the environment
variable `ANSIBLE_ROLE_TEST_CASES` to the name(s) of the file(s) under
test/testcases that you wish to execute (excluding the yml extension). So,
for example, if the testcases folder contains test files named first.yml,
second.yml, and third.yml, setting `ANSIBLE_ROLE_TEST_CASES=first,third`
would run only the tests in first.yml and third.yml.

The test framework executes the following steps when processing a test suite:
- The current state of each device is backed /mnt/flash directory on the
  device using the `copy running-config <backup_file>` command.
- Test cases are gathered from every file under test/testcases that matches
  the `ANSIBLE_ROLE_TEST_CASES` pattern, or all files if the variable is unset.
- Each test case is executed:
  - Setup for the test case is performed, if any exists.
  - The test case is executed against the role, verifying idempotency and
    any present or absent configuration that should exist on the device.
  - Test case teardown is performed, if any exists.
- The configuration for each device is restored from the previously stored
  backup.
- The backup files are removed from each device, leaving the device in the
  state in which it was before the tests.

To prevent restoring device configuration after tests have run (to debug
a failing test case, for example), set the environment variable
`NO_ANSIBLE_ROLE_TEST_TEARDOWN` to True (or any value that would evaluate to true).
In this case, restoring the device configuration may be accomplished manually
by issuing the command `configure replace <backup_file>` on each device. A
message should have been printed in the test output indicating the file name
used for the backup, as well as how to restore the device and delete the backup.


Developing Arista roles for Ansible
-----------------------------------

#### Role development guidelines

* Copy the following files from an existing Ansible EOS role into the
  current role:

  * .gitignore
  * Makefile*
  * defaults/main.yml*
  * files/README.md
  * filter_plugins/config_block.py
  * handlers/main.yml*
  * meta/main.yml*
  * tasks/main.yml*
  * templates/README.md
  * vars/main.yml*

  ```
  Note: Asterisk (*) indicates file should be reviewed for changes specific
  to the new role, such as updating the role name.
  ```


*XXX File structure, formatting guidelines, and other info goes here*



#### Role test development

* Edit the role's README.md file to include the **Developer Information**
  section. This should be copied from an existing role's README file.

* Create/copy the following files and folders from an existing Ansible EOS
  role into the current role:

  * test/fixtures/hosts
  * test/testcases/

* Import the arista-ansible-role-test repository into the role as a new
  submodule. From the root of the role directory, issue the following commands:

  * git submodule add https://github.com/arista-eosplus/arista-ansible-role-test.git test/arista-ansible-role-test
  * git diff --cached --submodule
  * git commit -am "Add arista-ansible-role-test submodule"

        --roletest-- >> git submodule add https://github.com/arista-eosplus/arista-ansible-role-test.git test/arista-ansible-role-test  
        Cloning into 'test/arista-ansible-role-test'...  
        remote: Counting objects: 29, done.  
        remote: Compressing objects: 100% (21/21), done.  
        remote: Total 29 (delta 10), reused 23 (delta 4), pack-reused 0  
        Unpacking objects: 100% (29/29), done.  
        Checking connectivity... done.  

        --roletest-- >> git diff --cached --submodule  
        diff --git a/.gitmodules b/.gitmodules  
        new file mode 100644  
        index 0000000..15da1b3  
        --- /dev/null  
        +++ b/.gitmodules  
        @@ -0,0 +1,3 @@  
        +[submodule "test/arista-ansible-role-test"]  
        \+ path = test/arista-ansible-role-test  
        \+ url = https://github.com/arista-eosplus/arista-ansible-role-test.git  
        Submodule test/arista-ansible-role-test 0000000...f13f684 (new submodule)  

        --roletest-- >> git commit -am "Add arista-ansible-role-test submodule"  
        [roletest 8305130] Add arista-ansible-role-test submodule  
         2 files changed, 4 insertions(+)  
         create mode 100644 .gitmodules  
         create mode 160000 test/arista-ansible-role-test  

* Add test cases for the role:

  Create a yml file for each group of test cases.
  Each group test file name should reflect the type of tests being run in
  that group. When a role contains several template modules, it is a good
  idea to have at least one test group for each template. Templates that
  perform multiple configuration changes may also be separated into several
  test group files.

  Each test group file should contain a defaults section and a testcases
  section. The defaults defines the module name reported by the test
  framework, and this should be the same as the filename without an extension.


*XXX Information specific to writing test cases and ensuring the test framework has been included as a submodule goes here*


License
-------

Copyright (c) 2016, Arista Networks EOS+
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of Arista nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

Author Information
------------------

Please raise any issues using our GitHub repo or email us at ansible-dev@arista.com
