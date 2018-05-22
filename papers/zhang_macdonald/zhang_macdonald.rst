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
reproducibility, and introduces Pulp as a tool to manage different packages.


Measuring Reproducibility
=========================

Tools
=====

Zip n Ship
----------

Github
------

Python
------

Ansible
-------

Docker
------

Management
==========

Tool Specific
-------------

Tool Agnostic
-------------


References
==========

.. [Atr03] P. Atreides. *How to catch a sandworm*,
           Transactions on Terraforming, 21(3):261-300, August 2003.


