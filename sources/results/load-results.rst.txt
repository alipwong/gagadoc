Loading results
===============

Set up simulation
------------------

.. code :: python

    import gaga as ga

    genes = {'x':(-2, 2),
            'y':(-1, 3)}

    def evaluate(individual):

        a = 1
        b = 100

        x = individual.genes['x']
        y = individual.genes['y']

        individual.fitness_score = (pow(a - x, 2) + b * pow(y - pow(x, 2), 2))
    
    sim = ga.ga(genes, evaluate)
    sim.run_simulation()

Direct access
-------------

If you have just run the genetic algorithm simulation you can access the results directly with ``sim.results``.

Example usage:

.. code :: python

    sim.results.animate('x', 'y')
    sim.results.plot_fitness()

Loading from a results file 
---------------------------

By default, when you run a simulation your results will be saved in ``results_folder``. By default, ``results_folder = results`` and will have the following file structure.

.. code :: text

    results
    |    history.txt 
    |    results_obj

To load the results object:

.. code :: python

    import bz2
    import pickle

    with bz2.open("./results/results_obj", "rb") as f:
        results = pickle.load(f)

Example usage:

.. code :: python

    results.animate('x', 'y')
    results.plot_fitness()

:ref:`Return Home <home>`