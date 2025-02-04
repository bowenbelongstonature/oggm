Bed inversion
=============

To compute the initial ice thickness :math:`h_0`, OGGM follows a methodology
largely inspired from [Farinotti_et_al_2009]_, but fully automated and relying
on different methods for the mass balance and the calibration.

Basics
------

The principle is simple. Let's assume for now that we know the flux of
ice :math:`q` flowing through a cross-section of our glacier. The flowline physics
and geometrical assumptions can be used to solve for the ice thickness
:math:`h`:

.. math::

    q = u S = \left(f_d h \tau^n + f_s \frac{\tau^n}{h}\right) S

With :math:`n=3` and :math:`S = h w` (in the case of a rectangular section) or
:math:`S = 2 / 3 h w` (parabolic section), the equation reduces to
solving a polynomial of degree 5 with one unique solution in
:math:`\mathbb{R}_+`. If we neglect sliding (the default in OGGM and in
[Farinotti_et_al_2009]_), the solution is even simpler.


Ice flux
--------

.. ipython:: python
   :suppress:

    fpath = "_code/prepare_climate.py"
    with open(fpath) as f:
        code = compile(f.read(), fpath, 'exec')
        exec(code)

If we consider a point on the flowline and the catchment area :math:`\Omega`
upstream of this point we have:

.. math::

    q = \int_{\Omega} (\dot{m} - \rho \frac{\partial h}{\partial t}) \ dA = \int_{\Omega} \widetilde{m} \ dA

with :math:`\dot{m}` the mass balance, and
:math:`\widetilde{m} = \dot{m} - \rho \partial h / \partial t` the
"apparent mass balance" after [Farinotti_et_al_2009]_. If the glacier is in
steady state, the apparent mass balance is equivalent to the the actual (and
observable) mass balance. Unfortunately, that is rarely the case, hence :math:`\partial h / \partial t` is not
known and there is no easy way to compute it. In order to continue, we have
to make the assumption that our geometry is in equilibrium.

This however has a very useful consequence: indeed, for the calibration
of our :doc:`mass-balance` model it is required to find a date :math:`t^*`
at which the glacier would be in equilibrium with its average climate
**while conserving its modern geometry**. Thus, we have
:math:`\widetilde{m} = \dot{m}_{t^*}`, where :math:`\dot{m}_{t^*}` is the
31-yr average mass balance centered at :math:`t^*` (which is known since
the mass balance model calibration).

The plot below shows the mass flux along the major flowline of Hintereisferner
glacier at :math:`t^*`. By construction, the flux is maximal at the equilibrium line and
zero at the glacier tongue.

.. ipython:: python
   :okwarning:

    @savefig example_plot_massflux.png width=100%
    example_plot_massflux()


Calibration
-----------

A number of climate and glacier related parameters are fixed prior to
the inversion, leaving only one free parameter for the calibration of the
bed inversion procedure: the inversion factor :math:`f_{inv}`. It is defined
such as:

.. math::

    A = f_{inv} \, A_0

With :math:`A_0` the standard creep parameter (:math:`2.4^{-24}`). Currently,
there is no "optimum" :math:`f_{inv}` parameter in the model. There is a high
uncertainty in the "true" :math:`A` parameter as well as in all other processes
affecting the ice thickness. Therefore, we cannot make any recommendation for
the "best" parameter. Global sensitivity analyses show that the default value
is a good compromise [Maussion_et_al_2019]_,
but very likely leads to overestimated ice volume [Farinotti_et_al_2019]_.

.. admonition:: **New in version 1.4!**

   As of OGGM v1.4, the user can choose to calibrate :math:`A` to match the
   consensus volume estimate from [Farinotti_et_al_2019]_ on any number
   of glaciers. We recommend to use a large number of glaciers (we match
   at the regional level) in order to allow some freedom to the model
   (it is not guaranteed that the consensus really is better for each glacier),
   but we assume that it is more accurate at large scales.


Distributed ice thickness
-------------------------

To obtain a 2D map of the glacier ice thickness and bed, the flowline thicknesses need to be
interpolated to the glacier mask. The current implementation of this
step in OGGM is currently very simple, but provides nice looking maps:


.. ipython:: python
   :okwarning:

    tasks.catchment_area(gdir)
    @savefig plot_distributed_thickness.png width=80%
    graphics.plot_distributed_thickness(gdir)


References
----------

.. [Farinotti_et_al_2009] Farinotti, D., Huss, M., Bauder, A., Funk, M., &
    Truffer, M. (2009). A method to estimate the ice volume and
    ice-thickness distribution of alpine glaciers. Journal of Glaciology, 55
    (191), 422–430.

.. [Farinotti_et_al_2019] Farinotti, D., Huss, M., Fürst, J. J., Landmann, J.,
   Machguth, H., Maussion, F. and Pandit, A.: A consensus estimate for the
   ice thickness distribution of all glaciers on Earth, Nat. Geosci., 12(3),
   168–173, doi:10.1038/s41561-019-0300-3, 2019.

.. [Maussion_et_al_2019] Maussion, F., Butenko, A., Champollion, N., Dusch, M.,
   Eis, J., Fourteau, K., Gregor, P., Jarosch, A. H., Landmann, J.,
   Oesterle, F., Recinos, B., Rothenpieler, T., Vlug, A., Wild, C. T. and
   Marzeion, B.: The Open Global Glacier Model (OGGM) v1.1, Geosci. Model Dev.,
   12(3), 909–931, doi:10.5194/gmd-12-909-2019, 2019.
