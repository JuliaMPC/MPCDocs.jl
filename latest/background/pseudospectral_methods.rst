.. _pseudospectral:

Pseudospectral Methods
======================

Change of Interval
------------------

To can change the limits of the integration (in order to apply Quadrature), we introduce :math:`\tau \in [-1,+1]` as a new independent variable and perform a change of variable for :math:`t` in terms of :math:`\tau`, by defining:

  .. math:: \tau = \frac{2}{t_{{N}_{t}}-t_0}t - \frac{t_{N_t}+t_0}{t_{N_t}-t_0}


Polynomial Interpolation
------------------------
Select a set of :math:`N_t+1` node points:

  .. math:: \mathbf{\tau} = [\tau_0,\tau_1,\tau_2,.....,\tau_{N_t}]


* These none points are just numbers

  * Increasing and distinct numbers :math:`\in [-1,+1]`

A *unique* polynomial :math:`P(\tau)` exists (i.e. :math:`\exists! P(\tau)`) of a maximum degree of :math:`N_t` where:

  .. math:: f(\tau_k)=P(\tau_k),\;\;\;k={0,1,2,....N_t}

* So, the function evaluated at :math:`\tau_k` is equivalent the this polynomial evaluated at that point.

But, between the intervals, we must approximate :math:`f(\tau)` as:

  .. math:: f(\tau) \approx P(\tau)= \sum_{k=0}^{N_t}f(\tau_k)\phi_k(\tau)

with :math:`\phi_k(•)` are basis polynomials that are built by interpolating :math:`f(\tau)` at the node points.

.. _diff_matrix:

Approximating Derivatives
-------------------------
We can also approximate the derivative of a function :math:`f(\tau)` as:

.. math:: \frac{\mathrm{d}f(\tau)}{\mathrm{d}\tau}=\dot{f}(\tau_k)\approx\dot{P}(\tau_k)=\sum_{i=0}^{N_t}D_{ki}f(\tau_i)

With :math:`\mathbf{D}` is a :math:`(N_t+1)\times(N_t+1)` differentiation matrix that depends on:

  * values of :math:`\tau`
  * type of interpolating polynomial

Now we have an approximation of :math:`\dot{f}(\tau_k)` that depends only on :math:`f(\tau)`!

Approximating Integrals
------------------------
The integral we are interested in evaluating is:

.. math:: \int_{t_0}^{t_{N_t}}f(t)\mathrm{d}t=\frac{t_{N_t}-t_0}{2}\int_{-1}^{1}f(\tau_k)\mathrm{d}\tau

This can be approximated using quadrature:

.. math:: \int_{-1}^{1}f(\tau_k)\mathrm{d}\tau\sum_{k=0}^{N_t}w_kf(\tau_k)

where :math:`w_k` are quadrature weights and depend only on:

  * values of :math:`\tau`
  * type of interpolating polynomial

Legendre Pseudospectral Method
------------------------------
* Polynomial

Define an N order Legendre polynomial as:

 .. math:: L_N(\tau) = \frac{1}{2^NN!}\frac{\mathrm{d}^n}{\mathrm{d}\tau^N}(\tau^2-1)^N

* Nodes

.. math::
    :nowrap:

    \begin{equation}
      \tau_k = \left \{
      \begin{aligned}
        &-1, && \text{if}\ k=0 \\
        &\text{kth}\;\text{root}\;of \dot{L}_{N_t}(\tau), && \text{if}\ k = {1, 2, 3, .. N_t-1}\\
        &+1\, && \text{if}\ k = N_t
      \end{aligned} \right.
    \end{equation}

* Differentiation Matrix

* Interpolating Polynomial Function
