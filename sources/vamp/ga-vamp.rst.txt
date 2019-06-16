Setting up the GA
=================

.. note:: There are a few things you may want to do first. If you're not already familiar with how to run a simple genetic algorithm, it is recommended you look at the :ref:`sphere function demonstration <sphere>`.

    If you're not familiar with using my ``star.py`` class to take interferometric measurements from mcfost models, play around with the ``DEMO_make_star.py`` script which you can download from here_. For further reading, see the accompanying manual ``making_measurements.pdf``.

.. _here: https://github.com/awon8465/VAMPIRES-MCFOST-GA/tree/master/make-measurements-code


It is easiest to write your genetic algorithm script in the same directory as all the supporting scripts so the imports are as follows:

.. code-block:: python
    
    import gaga as ga
    import VAMPIRES_data
    import star
    import numpy as np

Next, define the genes and their ranges. The gene names must match the parameter names in ``default_parameters_star.py``. For a description of the parameters, see the mcfost documentation. 

.. code-block:: python
    
    gene_definition = {"dust_mass": (1e-7, 6e-7),
                    "inner_radius": (1, 20),
                    "radius": (1, 800),
                    "outer_radius": (1, 30)}

The evaluate function
---------------------

The evaluate function is essentially the same as what is in ``DEMO_make_star.py``. We need to generate an mcfost model for our gene set and then calculated the reduced :math:`\chi^2` error to use as our fitness score.

.. code-block:: python
    
    def evaluate(individual):

        WAVELENGTH = 0.75 # wavelength of observation in microns

        # location where intermediate mcfost files are created
        MCFOST_path = "../mcfost-workspace/" 

        # name of the python file containing the Default_parameters class
        default_params = "default_parameters_star"

        # path to the VAMPIRES data
        VAMP_path = ""
        VAMP_data = "diffdata_RLeo_03_20170313_750-50_18holeNudged_0_0.idlvar"
        VAMP_data_info = "cubeinfoMar2017.idlvar"

        VAMP_data = VAMPIRES_data.VAMPIRES_data(VAMP_path + VAMP_data, VAMP_path + VAMP_data_info)

        # remove bias from the observed data i.e. move the average to 1
        VAMP_data.vhvv -= np.mean(VAMP_data.vhvv) - 1
        VAMP_data.vhvvu -= np.mean(VAMP_data.vhvvu) - 1

        # unpack the genes and use them to define the parameters for the model
        dm = individual.genes["dust_mass"]
        Rin = individual.genes["inner_radius"]
        r = individual.genes["radius"]
        Rout = individual.genes["outer_radius"]

        free_params = {"dust_mass": dm, "Rin": Rin, "radius":r, "Rout":Rout}

        # run mcfost
        s = star.Star(free_params, default_params, WAVELENGTH, MCFOST_path, VAMP_data, load=False, verbose=False)

        # calculate the reduced chi^2 error and use it as the fitness score
        s.calculate_reduced_chi2error()
        individual.fitness_score = s.reduced_chi2err

        # Optional: storing some useful data
        individual.data["data_Q"] = s.data_Q
        individual.data["data_U"] = s.data_U
        individual.data["reduced_chi2err_Q"] = s.reduced_chi2err_Q
        individual.data["reduced_chi2err_U"] = s.reduced_chi2err_U
        individual.data["reduced_chi2err"] = s.reduced_chi2err

Now you're ready to run the GA
------------------------------

.. code-block:: python

    sim = ga.ga(gene_definition, evaluate)
    sim.run_simulation()