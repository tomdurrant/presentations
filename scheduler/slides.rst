
Hindcast Scheduler
===================

----


----

Intercycle Dependences
=======================

- Multiple workflows
- Cycle can now depend on tasks from past cycles

----

Preprocessing/Model/Post
=========================

- SWAN nested configurations
- WW3 
      - preprocessing 15 mins
      - model 1 hour
      - postprocessing 45 mins


----

Preprocessing/Model/Post
=========================

Template
---------


.. code-block:: yaml

    pycallable:     ww3.ww3.WW3HC

    id:             era5_glob-st4_prod
    description:    Global WW3 0.5 by 0.5
    title:          WW3 Global %Y%m%d_%Hz
    modelexec:      ./ww3_shel

    ....


    schedule:
        docker:
            image:      metocean/ww3:mpich-v5.5.0.2
        allocate:   [28]
        mpi:        true
        cycles:
            frequency:  MONTHLY
            interval:   1


----

Preprocessing/Model/Post
=========================

Preprocessing
-------------

.. code-block:: yaml

    pycallable:     ww3.ww3.WW3HC
    template:       model.ww3_era5_glob-st4_prod
    tasks:          {prep: True, mod: False, post: False, clean: False}
    schedule:
        hard_dependency:
            - model.ww3_era5_glob-st4_prod-model:-2
        routing_key: "#.sexo.#"
        allocate:    [1]

----

Preprocessing/Model/Post
=========================

Model
-------------

.. code-block:: yaml

    pycallable:     ww3.ww3.WW3HC
    template:       model.ww3_era5_glob-st4_prod
    tasks:          {prep: False, mod: True, post: False, clean: False}
    schedule:
        hard_dependency:
            - model.ww3_era5_glob-st4_prod-pre
        routing_key: "#.xeon.#"
        allocate:   [27]

----

Preprocessing/Model/Post
=========================

Postprocessing
--------------

.. code-block:: yaml

    pycallable:     ww3.ww3.WW3HC
    template:       model.ww3_era5_glob-st4_prod
    tasks:          {prep: False, mod: False, post: True, clean: True}
    schedule:
        hard_dependency:
            - model.ww3_era5_glob-st4_prod-model
        routing_key: "#.xeon.#"
        allocate:    [1]
----

Preprocessing/Model/Post
=========================

Workflow
-----------


- ww3_cfsr_glob-st4.yml
    -  model.ww3_cfsr_glob-st4-pre
    -  model.ww3_cfsr_glob-st4-model
    -  model.ww3_cfsr_glob-st4-post


.. code::
    
    $ sched cycle 20120101z 20120301z -w ww3_cfsr_glob-st4


----




Current State
==============


.. code::

    [metocean@aotea3:~]
    % sched nodes
    ()
    | Node        | Total | Free cores | Allocated | Busy direct | 
    |-------------|-------|------------|-----------|-------------| 
    | xeon01      |    24 |         24 |         0 |             |
    | xeon02      |    28 |          0 |        28 |             |
    | xeon03      |    28 |          1 |        27 |             |
    | xeon04      |    28 |         28 |         0 |             |
    | xeon05      |    28 |          1 |        27 |             |
    | xeon06      |    28 |          1 |        27 |             |
    | sexo02      |     6 |          6 |         0 |             |
    | sexo01      |     6 |          6 |         0 |           0 |
    ()
    Available cores in this cluster:   67 / 176 

