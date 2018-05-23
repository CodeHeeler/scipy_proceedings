:author: Bihan Zhang
:email: bihan.zh@gmail.com
:equal-contributor:
:bibliography: mybib


:author: Austin Macdonald
:email: austin@redhat.com
:institution: Red Hat
:equal-contributor:


--------------------------------------------------
Reproducible Environments for Reproducible Results
--------------------------------------------------

.. class:: abstract

   Trustworthy results require reproducibility. Publishing source code is
   necessary but not sufficient for complete reproducibility since complex
   programs often depend on external code. [Pulp](https://pulpproject.org/)
   is open source, written in python, and can be used to manage and maintain
   reproducible computational environments. Attendees of any skill level can
   come learn best practices for creating reproducible environments, and
   managing them with Pulp.


.. class:: keywords

   dependency-hell, reproducibility


Introduction
============

Trustworthy results require reproducibility. Publishing code is necessary
but not sufficient for complete reproducibility. Complex programs often depend
on external code. “An article […] in a scientific publication is not the
scholarship itself, it is merely advertising of the scholarship. The actual
scholarship is the complete software development environment and the complete
set of instructions which generated the figures”(Buckheit, Emphasis Added).

Fortunately, this is a common problem and there are a number of best practices
and tools that can make this easier. A common solution for high level dependencies
is to explicitly “pin” them (Wilson). In python this would be running

.. code-block:: bash

   pip freeze > requirements.txt


But this only manages package dependencies and not system dependencies. And,
because these resources are user managed and exist on 3rd party platforms,
content can be modified or removed making it difficult or impossible to
guarantee reproducibility.

This paper seeks to explore several different methods at managing environmental
reproducibility, and introduces Pulp as a manager for various environments.


Measuring Reproducibility
=========================

Two factors have to be considered when we think about reproducibility.

Complete reproducibility is having the researcher and reviewer share identical 'bits' of the necessary system, program, and dependencies.

Vandewalle identifies several necessity for complete reproducibility [Vandewalle]: the program's source code,
package dependencies, system requirements and configuration, data source used, and documentation on running the provided the source code.

On the other side one must determine if these programs and environments are flexible to change. Software moves fast, and even widely used programs become
legacy and eventually deprecated. Pinning dependencies might accelerate this process.

A good tool, and management system will balance complete reproducibility and code rot.

Tools
=====

Published Source Code
---------------------

Scholarly research containing descriptions of methodology is no longer sufficient.
For standalone scripts, publishing source code might be acceptable, But as computational systems grow more complex,
this method becomes more unreliable. Nontrivial research oftentimes depend on other external libraries for everything from left-padding
a line, to building scalable machine learning. This "has led to an ever larger and more complex
black box between what was actually done and what is described in literature." [Boettinger An intro to docker for reproducible research]

Publishing source code in papers makes them inflexible to change- bugs fixed after publication
cannot be communicated to the readers of the paper. Code is not versioned and even if the source code is updated and
made available it is hard to communicate what issues were fixed.

Git
----

Using an online git repository is a great way to keep track of source code [Good Enough Practices for Scientific Computing].
With git you can easily track changes you make to data and software. Git identifies commits by a unique hash, which can be used
to reference a specific point in the source code.

What git lacks is the ability to do environmental management.
Git is not a package manager. System dependencies in git can only be documented- and need the user to install them following instructions.
It is recommended that git be used to store the source code, and that some other package manager be used to manage the system environment.

Python
------

Ansible
-------

Containers
----------
OCI




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

.. [Atr03] P. Atreides. *How to catch a sandworm*,
           Transactions on Terraforming, 21(3):261-300, August 2003.


