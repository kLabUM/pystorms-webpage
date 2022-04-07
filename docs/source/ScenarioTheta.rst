Example: Scenario Theta
=======================

This scenario was created to serve as a unit test for control algorithms.
In this scenario, two idealized basins (in parallel) of :math:`1000m^3` are draining into a downstream water body. 
Outlets in the basins (:math:`1m^2`) are at the bottom and can be controlled throughout the duration of the simulation.

.. image:: ./figures/theta.png
  :width: 400
  :align: center

Objective
---------

Maintain the flow of water at the outlet of the stormwater network below :math:`0.5 m^3s^{-1}`.
The degree of success or failure of the algorithm to achieve the objective is computed based on the following metric.

States
------

Water levels (:math:`m`) in the two basins at every step, indexed by the order of the basin are
defined as the states in this scenario.

Control actions
---------------

Percent of valve opening :math:`[0,1]` at the outlet of each basin.

Example: Equal-filling Controller
---------------------------------
.. code:: ipython3

    import pystorms
    import numpy as np
    import matplotlib.pyplot as plt
    %matplotlib notebook

.. code:: ipython3

    env = pystorms.scenarios.theta()
    done = False
    while not done:
        done = env.step(np.ones(2))
    print("Uncontrolled Performance : {}".format(env.performance()))

.. parsed-literal::

    Uncontrolled Performance : 0.1296391721430919

**Lets take a look at the network outflows in the uncontrolled
response**

.. code:: ipython3

    plt.plot(env.data_log["flow"]["8"])
    plt.ylabel("Outflows")

.. parsed-literal::

    Text(0, 0.5, 'Outflows')

.. image:: figures/theta_uncontrolled.png

Now, lets see if we can design a control algorithm to maintain the
flows below :math:`0.5 m^3s^{-1}`

Design of such a control algorithm can be approached in many ways. But
the fundemental idea behind any of these algorithms would be to hold
back water in the basins and coordinate the actions of these basin such
that their cummulative outflows are below the desired threshold. In this
example, we will design a simple algorithm that achives this.

.. code:: ipython3

        def controller(depths,
                       N=2,
                       LAMBDA=0.5,
                       MAX_DEPTH=2.0):
            
            # Compute the filling degree
            f = depths/MAX_DEPTH
            
            # Estimate the average filling degree
            f_mean = np.mean(f)
            
            # Compute psi
            psi = np.zeros(N)
            for i in range(0, N):
                psi[i] = f[i] - f_mean
                if psi[i] < 0.0 - 10**(-4):
                    psi[i] = 0.0
                elif psi[i] >= 0.0 - 10**(-4) and psi[i] <= 0.0 + 10**(-4):
                    psi[i] = f_mean
            
            # Assign valve positions
            actions = np.zeros(N)
            for i in range(0, N):
                if depths[i] > 0.0:
                    k = 1.0/np.sqrt(2 * 9.81 * depths[i])
                    action = k * LAMBDA * psi[i]/np.sum(psi)
                    actions[i] = min(1.0, action)
            return actions

.. code:: ipython3

            env_controlled = pystorms.scenarios.theta()
            done = False 
            while not done:
                state = env_controlled.state()
                actions = controller(state, 0.50)
                done = env_controlled.step(actions)

.. code:: ipython3

            plt.plot(env_controlled.data_log["flow"]["8"], label="Controlled")
            plt.plot(env.data_log["flow"]["8"], label="Uncontrolled")
            plt.ylabel("Outflows")
            plt.legend()

.. image:: figures/theta_controlled.png

.. code:: ipython3

            print("Controlled performance: {} \nUncontrolled performance: {}".format(env_controlled.performance(), env.performance()))

.. parsed-literal::

    Controlled performance: 0.0 
    Uncontrolled performance: 0.1296391721430919

Controller is able to maintain the outflows from the network below the desried threshold.
