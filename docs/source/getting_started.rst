``pystorms``
============

``pystorms`` is created to aid in the evaluation and development of algorithms for the control of storm water networks.
It is an open-source Python library that enables the quick evaluation of control strategies on real world-inspired smart storm water networks.


``pystorms`` provides users with,

        1. A collection of real-world inspired stormwater networks (see Scenarios).
        2. A streamlined programming interface to analyze performance of control algorithms on the included networks (see Environment).

Installation
------------

This package is built in python 3+ and is supported on all operating systems (Windows, Mac OS, Linux).

.. code:: bash

   pip install pystorms

``pystorms`` dependencies:

- PyYAML 5.1+
- python 3+
- numpy
- pyswmm
 
If you run into any issues installing the package, please refer to the advanced section for additional installation instructions or feel free to contact us.

Getting Started
---------------

All necessary dependencies for the ``pystorms`` package should be installed via the ``pip`` package installer. To ensure ``pystorms`` has been installed successfully--while also providing the basics for use of ``pystorms``--, the following code snippet can be run, with details of its components provided below.

.. code:: python

   import pystorms

   # (3) Define the control algorithm
   def controller(state):
        actions = np.ones(2)
        if state[0] > 0.5:
                actions[0] = 0.5
        if state[1] > 0.5:
                actions[1] = 0.5
        return actions

   # (1) Call and define the scenario
   env = pystorms.scenarios.theta()

   # (2) Run the simulation
   done = False
   while not done:
       state = env.state()
       actions = controller(state)
       done = env.step(actions)

We break the above code snippet into three parts:

1. Define a scenario

To run a stormwater control simulation using ``pystorms``, a **scenario** must first be defined. **Scenarios are the fundamental components of the pystorms library**, and each consist of (i) a stormwater network, (ii) a driving precipitation event, and (iii) a control objective.

As seen in the code above, a scenario is called by defining a class (e.g. :code:`env`) that initializes the default simulation modules specific for the corresponding scenario. In the code above, the corresponding scenario is *theta*, which is a basic stormwater control scenario developed specifically for testing and debugging of new code and/or control algorithms in :code:`pystorms`.

For the example above, the scenario class is defined using the default precipitation event and control objective of the theta scenario; it is defined fully by the following line of code: :code:`env = pystorms.scenarios.theta()`).

2. Delineate a simulation

A **simulation** for the scenario is then delineated using the method calls of the scenario class (i.e. :code:`env` in this example). For ``pystorms`` the method calls of the scenario class were defined to follow the typical actions that occur in-situ for *actual* control systems. Namely, there are two specific components: (i) querying the state of the system, and (ii) implementing control actions based on the system's state.

i. States in the stormwater network (e.g. water levels, flows, pollutant concentrations) can be queried at any point of the simulation using the :code:`env.state()` method.
ii. Control actions can be implemented in the network, and the simulation can be progressed forward for a specified time-step using the :code:`env.step(<your actions here>)` call. Please refer to the scenarios section for more information.

3. Implementing the control algorithm

The simulation set up in this way eases the ability to explicitly segregate the **control algorithm** that determines what control actions are to be implemented. By separating out the control algorithm in this way, the user is able to focus on testing various control strategies and their computational implementation via their algorithms.

As can be seen by the case provided here, the control algorithm is such that all settings corresponding to the control assets are set to ``1.0`` (the proportional equivalent of ``100%``). If the states at either of the state locations read greater than ``0.5``, then the corresponding control asset setting is changed to ``0.5``. (The details of what physical parameters the state and control setting values correspond to are discussed in the Scenario Theta section).

We refer to the case when no control is implemented as the **uncontrolled case**. To run the uncontrolled case, simply progress the simulation without defining any actions in the step call (i.e. :code:`env.step()`).

Background
----------

``pystorms`` is built on `pyswmm <https://github.com/OpenWaterAnalytics/pyswmm>`_, which is a Python wrapper developed to interface with the United State's Environmental Protection Agency Storm Water Management Model `EPA-SWMM <https://www.epa.gov/water-research/storm-water-management-model-swmm>`_, thus allowing users to bypass the need for utilizing the EPA-SWMM GUI (available only on Windows operating systems) or compiling the EPA-SWMM source code for usage.


``pystorms`` is developed by Open-Storm, the open-source outreach efforts of a consortium of university researchers, industry experts, and municipal practitioners, and lead by the `Real-Time Water Systems Lab <http://www-personal.umich.edu/~bkerkez/>`_ at the University of Michigan.

Citation
--------

.. code:: latex

        @article{rimer2021pystorms,
          title={pystorms: A simulation sandbox for the development and evaluation of stormwater control algorithms},
          author={Rimer, Sara P and Mullapudi, Abhiram and Troutman, Sara C and Ewing, Gregory and Bowes, Benjamin D and Akin, Aaron A and Sadler, Jeffrey and Kertesz, Ruben and McDonnell, Bryant and Montestruque, Luis and others},
          journal={arXiv preprint arXiv:2110.12289},
          year={2021}
        }

License
-------
``pystorms`` is licensed under a GNU General Public License (v3).

`Quick Summary of GNU v3 <https://tldrlegal.com/license/gnu-general-public-license-v3-(gpl-3)>`_: *You may copy, distribute and modify the software as long as you track changes/dates in source files. Any modifications to or software including (via compiler) GPL-licensed code must also be made available under the GPL along with build & install instructions.*

