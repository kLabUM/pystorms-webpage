========
Advanced
========

Multi-processing
================

pystorms environments can be seamlessly adopted for multiprocessing

.. code:: ipython3

    import pystorms
    import numpy as np
    from multiprocessing import Pool

``worker``
----------

This function takes a controller as an argument, enabling us to evaluate
multiple control strategies simultaniously.

.. code:: ipython3

    def worker(config):
        env = pystorms.scenarios.gamma()
        done = False
        # different controllers
        controller = config["controller"]
        while not done:
            actions = controller(env.state())
            done = env.step(actions)
        return env.performance()

``swarm``
---------

This function maps the worker function onto multiple processors and
return the performance

.. code:: ipython3

    def generate_swarm(config, worker, processors, jobs):
        """
        Generate workers based on the environment and controller
        """
        if type(config) == list:
            swarm_inputs = config
        else:
            swarm_inputs = [config for i in range(0, jobs)]
    
        with Pool(processors) as p:
            data = p.map(worker, swarm_inputs)
        return data

Example:
--------

.. code:: ipython3

    # Define two generic controllers
    def control_1(state):
        return np.ones(11)
    
    def control_2(state):
        return np.zeros(11)
    
    # Create the config file
    config = [{"controller": control_1}, {"controller": control_2}]

.. code:: ipython3

    generate_swarm(config, worker, 2, 2)




.. parsed-literal::

    [15736.942557976538, 282982.5873304134]



Lets time it to check that the function is running on mutiple
processors. If sucessful, simulation time should be half.

Serial and Parallel
^^^^^^^^^^^^^^^^^^^

.. code:: ipython3

    %%timeit
    worker(config[0])
    worker(config[1])


.. parsed-literal::

    15.1 s ± 434 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


.. code:: ipython3

    %%timeit
    generate_swarm(config, worker, 2, 2)


.. parsed-literal::

    10.8 s ± 42.6 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


Thats 3 seconds more than what i expected. This might be due the
initialization cost. This should go down as the number of simulations
increase.

.. code:: ipython3

    %%timeit
    worker(config[0])
    worker(config[1])
    worker(config[0])
    worker(config[1])
    worker(config[0])
    worker(config[1])


.. parsed-literal::

    44.1 s ± 79.6 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


.. code:: ipython3

    %%timeit
    generate_swarm(config, worker, 3, 6)


.. parsed-literal::

    10.8 s ± 68.9 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


.. code:: ipython3

    %%timeit
    generate_swarm(config, worker, 6, 6)


.. parsed-literal::

    11.1 s ± 451 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


Thats consistent!


``pyswmm`` functionality
========================

``pystorms`` is built on ``pyswmm``. It uses ``pyswmm`` as its back-end
for interacting with EPA-SWMM’s computational engine. Hence, all the
functionality in ``pyswmm`` is inherently available in ``pystorms``.
``pystorms`` by defaults supports a subset of the states though its API.
These subsets of states (i.e. depth, flows, inflows) were chosen to
represent frequently used parameters for making control decisions. Refer
to the documentation on states for more details on the supported
parameters. ``pystorms`` architecture is designed to enable users easy
access to all the existing pyswmm functionality.

.. code:: ipython3

    import pystorms
    import pyswmm.toolkitapi as tkai

This example demonstrates how pyswmm functionality can be invoked from
`pystorms`.

.. code:: ipython3

    env = pystorms.scenarios.theta()

All function calls being used in the environment for populating state
vector are listed in the ``env.env.methods`` dictionary.

.. code:: ipython3

    env.env.methods




.. parsed-literal::

    {'depthN': <bound method environment._getNodeDepth of <pystorms.environment.environment object at 0x11929d690>>,
     'depthL': <bound method environment._getLinkDepth of <pystorms.environment.environment object at 0x11929d690>>,
     'flow': <bound method environment._getLinkFlow of <pystorms.environment.environment object at 0x11929d690>>,
     'flooding': <bound method environment._getNodeFlooding of <pystorms.environment.environment object at 0x11929d690>>,
     'inflow': <bound method environment._getNodeInflow of <pystorms.environment.environment object at 0x11929d690>>,
     'pollutantN': <bound method environment._getNodePollutant of <pystorms.environment.environment object at 0x11929d690>>,
     'pollutantL': <bound method environment._getLinkPollutant of <pystorms.environment.environment object at 0x11929d690>>}



Let us say, we want to use volume as a state. All we have to do is add
the function call reading volume to the dict.

.. code:: ipython3

    def _getNodeVolume(NodeID):
        return env.env.sim._model.getNodeResult(NodeID, tkai.NodeResults.newVolume.value)

`env` refers to the scenario being initialized.
For example, if the scenario was initialized as `sce` the first class in the return statement would be `sce`. env in `<scenario class>.env` refers to the environment class used to communicate with pyswmm/EPA-SWMM.
`sim._model` refers to the EPA-SWMM simulation initialized by invoking the scenario class.
`getNodeResult` is the functional call that queries the volume from the EPA-SWMM.

.. code:: ipython3

    env.env.methods["volumeN"] = _getNodeVolume

Lets add volume to the state vector

.. code:: ipython3

    env.config["states"]




.. parsed-literal::

    [('P1', 'depthN'), ('P2', 'depthN')]



.. code:: ipython3

    env.config["states"].append(('P1', 'volumeN'))
    env.config["states"].append(('P2', 'volumeN'))

*NOTE:* Arguments to the volume function are tuple appended to the config dict. Refer to `environment.py <https://github.com/kLabUM/pystorms/blob/ffa3564ef5f80811ca246b10ffeb0e38f36befdb/pystorms/environment.py#L74>`_ for more details on how state vector is populated. 


.. code:: ipython3

    env.config["states"]




.. parsed-literal::

    [('P1', 'depthN'), ('P2', 'depthN'), ('P1', 'volumeN'), ('P2', 'volumeN')]



Now when ``env.state()`` is called, it returns all both depth and volume
in nodes

.. code:: ipython3

    env.state()




.. parsed-literal::

    array([0., 0., 0., 0.])



Refer to pyswmm documentation for details on the all supported
parameters.

Example
-------

.. code:: python

        import pystorms
        import pandas as pd
        import pyswmm.toolkitapi as tkai
        import matplotlib.pyplot as plt


        # Create the function call for reading volume
        def getNodeVolume(NodeID):
            return env.env.sim._model.getNodeResult(NodeID, tkai.NodeResults.newVolume.value)


        # Initalize scenario
        env = pystorms.scenarios.theta()

        # Update the methods dict
        env.env.methods["volumeN"] = getNodeVolume
        # Update state vector
        env.env.config["states"].append(("P1", "volumeN"))
        env.env.config["states"].append(("P2", "volumeN"))

        done = False
        data = []
        while not done:
            state = env.state()
            done = env.step([1, 1])
            data.append(state)

        data = pd.DataFrame(data, columns=["depthP1", "depthP2", "volumeP1", "volumeP2"])
        data.plot()
        plt.show()


Defining States
===============

.. code:: python

   import pystorms
   import numpy as np
   import matplotlib.pyplot as plt

In scenario gamma, states are defined as the depths in the basins of the network.
If needed custom states can be defined by the users, overwriting or
appending to the default states provided in the library. Consider the following example, 

.. code:: python

   env = pystorms.scenarios.gamma()

States defined in gamma scenario can be queried as follows,

.. code:: python

   env.config["states"]

::

   [('1', 'depthN'),
    ('2', 'depthN'),
    ('3', 'depthN'),
    ('4', 'depthN'),
    ('5', 'depthN'),
    ('6', 'depthN'),
    ('7', 'depthN'),
    ('8', 'depthN'),
    ('9', 'depthN'),
    ('10', 'depthN'),
    ('11', 'depthN')]

Appending the state list with the new state choice would update the
state vector returned by the ``env.state()`` function.

For example, consider the case when we want to add outflow from the 4th
basin to the state.

.. code:: python

   env.config["states"].append(("O4", "flow"))

.. code:: python

   env.config["states"]

::

   [('1', 'depthN'),
    ('2', 'depthN'),
    ('3', 'depthN'),
    ('4', 'depthN'),
    ('5', 'depthN'),
    ('6', 'depthN'),
    ('7', 'depthN'),
    ('8', 'depthN'),
    ('9', 'depthN'),
    ('10', 'depthN'),
    ('11', 'depthN'),
    ('O4', 'flow')]

When we run the simulation, ``env.state()`` function should return a
vector of length 12.

.. code:: python

   done = False
   while not done:
       state = env.state()
       done = env.step(np.ones(11))

.. code:: python

   len(state)

::

   12

Users are not limited to the flows in the network. They are allowed to
access any value computed by the swmm network.

Supported state queries: 
        * ``depthN``     : Depth in nodes
        * ``depthL``     : Depth in links 
        * ``flow``       : Flow in links/orifices/weir 
        * ``flooding``   : Flooding in nodes 
        * ``pollutantN`` : Pollutant in nodes 
        * ``pollutantL`` : Pollutant in links
        * ``inflow``     : Inflow into nodes
