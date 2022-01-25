=========
Scenarios
=========

Scenarios are the primary components of the ``pystorms`` library. These scenarios are designed to be used for quantitatively evaluating the performance of stormwater control algorithms.

.. image:: ./figures/fig_scenarioComponents.png
        :width: 400
        :align: center

Each is scenario comprises of the following:
        1. An underlying stormwater network
        2. A system driver (e.g., rain event)
        3. A set of controllable assets
        4. A set of observed states
        5. A control objective

``pystorms`` includes 7 scenarios with diverse set of objectives; these are summarized in the table below.

.. csv-table:: Scenarios
        :file: ./tables/scenarios.csv
        :widths: 5, 10, 10
        :header-rows: 1

Please refer to manuscript for additional details on the individual scenarios.
