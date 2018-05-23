:author: Bihan Zhang
:email: bihan.zh@gmail.com
:institution: Red Hat

:author: Austin Macdonald
:email: austin@redhat.com
:institution: Red Hat

:bibliography: mybib

--------------------------------------------------
Reproducible Environments for Reproducible Results
--------------------------------------------------

.. class:: abstract

   Trustworthy results require reproducibility. Publishing source code is
   necessary but not sufficient for complete reproducibility since complex
   programs often depend on external code. [Pulp](https://pulpproject.org/) is
   open source, written in python, and can be used to manage and maintain
   reproducible computational environments. Attendees of any skill level can
   come learn best practices for creating reproducible environments, and
   managing them with Pulp.


.. class:: keywords

   dependency-hell, reproducibility


Introduction
============

Trustworthy results require reproducibility. Publishing code is necessary but
not sufficient for complete reproducibility. Complex programs often depend on
external code. “An article […] in a scientific publication is not the
scholarship itself, it is merely advertising of the scholarship. The actual
scholarship is the complete software development environment and the complete
set of instructions which generated the figures” :cite:`Buckheit` .

Fortunately, this is a common problem and there are a number of best practices
and tools that can make this easier. A common solution for high level
dependencies is to explicitly “pin” them (Wilson). In python this would be
running

.. code-block:: bash

   pip freeze > requirements.txt


But this only manages package dependencies and not system dependencies. And,
because these resources are user managed and exist on 3rd party platforms,
content can be modified or removed making it difficult or impossible to
guarantee reproducibility.

This paper seeks to explore several different methods of managing environmental
reproducibility, and introduces Pulp as a manager for various environments.


Measuring Reproducibility
=========================

Two factors have to be considered when we think about reproducibility.

Complete reproducibility is having the researcher and reviewer share identical
'bits' of the necessary system, program, and dependencies.

Vandewalle identifies several necessities for complete reproducibility
[Vandewalle]: the program's source code, package dependencies, system
requirements and configuration, data source used, and documentation on running
the provided the source code.

On the other side one must determine if these programs and environments are
flexible to change. Software moves fast, and even widely used programs become
legacy and eventually deprecated. Pinning dependencies might accelerate this
process.

A good tool, and management system will balance complete reproducibility and
code entropy.

Tools
=====

Published Source Code
---------------------

Scholarly research containing descriptions of methodology is no longer
sufficient.  For standalone scripts, publishing source code might be
acceptable, But as computational systems grow more complex, this method becomes
more unreliable. Nontrivial research oftentimes depend on other external
libraries for everything from left-padding a line, to building scalable machine
learning. This "has led to an ever larger and more complex black box between
what was actually done and what is described in literature." [Boettinger An
intro to docker for reproducible research]

Publishing source code in papers makes them inflexible to change- bugs fixed
after publication cannot be communicated to the readers of the paper. Code is
not versioned and even if the source code is updated and made available it is
hard to communicate what issues were fixed.

Github and Other Version Control Software
-----------------------------------------

Using an online git repository is a great way to keep track of source code
[Good Enough Practices for Scientific Computing].  With git you can easily
track changes you make to data and software. Git identifies commits by a unique
hash, which can be used to reference a specific point in the source code.

What git lacks is the ability to do environmental management.  Git is not a
package manager. System dependencies in git can only be documented- and need
the user to install them following instructions.  It is recommended that git be
used to store the source code, and that some other package manager be used to
manage the system environment.

Python Packaging
----------------

Python has a strong community, and many libraries and tools are hosted on the
Python Package Index.  Currently, the standard tool for installing packages is
[pip](https://pip.pypa.io/en/stable/), which installs Python packages and their
Python dependencies. For development, it is strongly recommended to use pip
with virtual environments [0]_. Doing so will allow the developed
projects to use the newest stable versions of their dependencies, and well
maintained dependencies should work correctly together.

.. code-block:: bash

   $ mkvirtualenv venv-demo (venv-demo)
   $ pip install scipy

After development is complete and analysis begins, the need for reproducibility
overtakes the need for keeping dependencies up to date. Though many projects strive
to maintain backwards compatibility, a researcher would not want to use
numpy-1.13.1 for part of their analysis and numpy-1.14.2 for another, the
stakes are simply too high. At this point, users can “pin” their versions.

.. code-block:: bash

   $ workon venv-demo (venv-demo)
   $ pip freeze > scipy-requirements.txt

Pip can use [requirements
files](https://pip.readthedocs.io/en/1.1/requirements.html) to achieve more
stability. Creating a requirements file in this way specifies the exact version
of each dependency.

.. code-block:: bash

   numpy==1.14.3 scipy==1.1.0

The requirements file can now be used to recreate the same environment using
the same versions.

.. code-block:: bash

   $ mkvirtualenv separate-env
   (separate-env) $ pip install -r scipy-requirements.txt

For Python users who need to guarantee deterministic builds, another step is
suggested. Adding hashes to a requirements.txt provides the guarantee that the
exact bits are installed. PyPI now supports sha256, which is strongly
recommended over md5, which has known vulnerabilities. Pip can be used to
calculate the hashes, which are then added to the requirements file.

.. code-block:: bash

   $ pip download numpy==1.14.3
   Collecting numpy==1.14.3
   Saved ./numpy-1.14.3-cp27-cp27mu-manylinux1_x86_64.whl
   Successfully downloaded numpy

.. code-block:: bash

   $ pip hash ./numpy-1.14.3-cp27-cp27mu-manylinux1_x86_64.whl
   ./numpy-1.14.3-cp27-cp27mu-manylinux1_x86_64.whl:
   --hash=sha256:0db6301324d0568089663ef2701ad90ebac0e975742c97460e89366692bd0563

Add these hashes to your requirements file, and use the `--require-hashes`
option. Note that these files are specific to architecture and python package type.
For code that should run in more than one environment, multiple hashes can be
specified.

.. code-block:: bash

   numpy==1.14.3 \
       --hash=sha256:0db6301324d0568089663ef2701ad90ebac0e975742c97460e89366692bd0563
   scipy==1.1.0 \
       --hash=sha256:08237eda23fd8e4e54838258b124f1cd141379a5f281b0a234ca99b38918c07a

.. code-block:: bash

   $ mkvirtualenv deterministic-venv (deterministic-venv) $ pip install --require-hashes -r
   scipy_requirements.txt

Guarantees:
 - All Python dependencies installed this way will contain exactly the same
   bits
 - Hashes safeguard against man in the middle attacks
 - Hashes safeguard against malicious modification of packages on PyPI

Limitations: Packages on PyPI can be removed at any time by their maintainer.
pip is only useful for managing python dependencies, and cannot be used for
system dependencies and environment configuration.

Pip was selected because it is the standard tool, and it is most likely to
maintain backward compatibility. However, there are other tools with rich
feature sets that simplify the process. In particular,
[pipenv](https://docs.pipenv.org/) uses hashing and virtual environments by
default for a smooth experience.


Ansible
-------

Ansible is an IT automation tool. It can configure systems, deploy software,
and orchestrate more advanced tasks [ansible website] With ansible it is
possible to install python dependencies and system dependencies.

"The approach is characterized by scripting, rather than documenting, a
description of the necessary dependencies for software to run, usually from the
Operating System [...] on up" [Clark berkley’s common scientific compute
environments for research and education]


With ansible you write an ansible playbook that executes a set of tasks. Each
task is idempotent.


.. code-block:: yaml

   - name: Install python3-virtualenvwrapper (Fedora)
     package:
     name:
       - which
       - python3-virtualenvwrapper
     when:
       - pulp_venv is defined
       - ansible_distribution == 'Fedora'

   - name: Create a virtualenv
     command: 'python3 -m venv my_venv'
     args:
       creates: 'my_venv'
     register: result

   - pip:
     name: scipy
     version: 1.1.0

   - dnf:


Ansible is only as good as your playbook. To make your environment
reproducible, your playbook has to follow best practices like pinning packages
to a version. A default host OS also should be specified when the playbook is
written: ansible uses separate plugins to install system dependencies, and to
be multiplatform the researcher needs to do some ansible host checking to use
the right plugins.

Ansible playbook and roles are yaml files that can be called with:

.. code-block:: bash

    ansible-playbook playbook.yml

Containers
----------

Containers[1] "are technologies that allow you to package and isolate
applications with their entire runtime environment—all of the files
necessary to run." [https://www.redhat.com/en/topics/containers]
Applied to the scienctific field this means that each container will contain
an image of your system, a copy of your source code, installed dependencies,
and data used. These are stored in a static file called an Image.

This Image can be given to peer reviewers and other collaborators as a baseline
to run your research. However the Image itself is opaque, and it is hard to tell
what dependencies have been installed on the image without substantial inspection.
It is recommended that the Image is built from a Dockerfile for full transparency.

A Dockerfile is a text document that contains all the commands a user could call
on the command line to assemble an image [https://docs.docker.com/engine/reference/builder/].

This example dockerfile creates an ubuntu image and installs scipy and numpy on it.

.. code-block:: text

   FROM ubuntu:16.04
   RUN pip install scipy --hash=sha256:0db6301324d0568089663ef2701ad90ebac0e975742c97460e89366692bd0563


An Dockerfile can be built by running

.. code-block:: bash

   docker build


Note that while the Docker image is immutable, running `docker build` on the
same Dockerfile does not guarantee an identical image, unless best practices
were followed.

Dockerfiles can be kept in github, and linked to DockerHub so that the
image is rebuilt with every change to the Dockerfile. This is the best of both
worlds- an immutable image is managed by DockerHub, but documentation on how
that image was built is kept under version control.

DockerHub identifies images by their digest, so the chance of collision is low.
Sharing a DockerHub managed image can be done by providing your docker repository
and a digest.

.. code-block:: bash

    docker pull internal-registry/my-project@sha256:b2ea388fdbabb22f10f2e9ecccaccf9efc3a11fbd987cf299c79825a65b62751


The downside of Docker Images is that docker is high in entropy. The Docker
Engine has no long-term support version [https://github.com/moby/moby/issues/20424].
This could result in `docker load` suddenly not working [https://github.com/moby/moby/issues/20380]
after upgrading system docker to a later version.



Multi Environmental Management
==============================

Pulp
----

Artifactory
-----------

Summary
=======

Acknowledgements
================

References
==========


.. [0] A virtual environment, often abbreviated “virtualenv” or “venv”,
    is an isolated python environments that is used to prevent projects and their
    dependencies from interfering with with each other. Under the hood, virtual
    environments work by managing the PYTHON_PATH (TODO: is this the right var
    name?) Another benefit of virtual environments is that they do not require root
    privileges and are safer to use.


.. [1] Most often people think of docker containers when the word
    container is mentioned. Docker is the most well known, however docker schema,
    and standards are not well documented.  Containers in this case can refer to
    Linux Container which is a superset of Docker Containers, Rkt, LXC, and other
    implementations. While most of the ideas discussed here will be generic
    across containers, the docker container, and DockerHub will be used as
    examples, due largely in part to their popularity.
