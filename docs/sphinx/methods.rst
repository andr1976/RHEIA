.. _lab:methods:

Details on the methods
======================

The math behind the different methods, with appropriate references.

Uncertainty quantification
--------------------------

To perform uncertainty quantification, the Polynomial Chaos Expansion method is applied in RHEIA. 
In the section below, the method is briefly introduced. Additional details on the method are described by Sudret et al. :cite:`Sudret2014`. 

.. _lab:pce:

Polynomial Chaos Expansion
^^^^^^^^^^^^^^^^^^^^^^^^^^

The Polynomial Chaos Expansion (PCE) representation of the system model consists of a series of orthogonal polynomials :math:`\Psi_i` with corresponding coefficients :math:`u_i`:

:math:`\mathcal{M}^{\mathrm{PCE}}(\pmb{X}) = \sum_{\pmb{\alpha} \in \mathcal{A}} u_{\pmb{\alpha}} \Psi_{\pmb{\alpha}} (\pmb{X}) \approx \mathcal{M}(\pmb{X})`, 

where :math:`\pmb{\alpha}` are the multi-indices and :math:`\mathcal{A}` is the considered set of multi-indices, for which the size is defined by a truncation scheme. 
The polynomial family that is orthogonal with respect to the assigned probability distributions are known for classic distributions. 
As an example, uniformly distributed stochastic input parameters associate with the Legendre polynomials.
A typical truncation scheme is adopted, which limits the polynomial order up to a certain degree. This constrains the number of multi-indices in the set to:

:math:`|\mathcal{A}^{M,p}| = \dfrac{(p + M)!}{p!M!}`,

where :math:`p` corresponds to the polynomial order and :math:`M = |\pmb{X}|` is the stochastic dimension, i.e. number of uncertainties (parameters and variables).
Consequently, :math:`|\mathcal{A}^{M,p}|` coefficients are present in a full PCE. To quantify these coefficients, least-square minimization is applied, based on actual system model evaluations. 
To ensure a well-posed Least-Square Minimization, :math:`2|\mathcal{A}^{M,p}|` system model evaluations are usually required. 
When the coefficients are quantified, the mean (:math:`\mu`) and standard deviation (:math:`\sigma`) of the objective follow analytically:

:math:`\mu = u_0`,

:math:`\sigma^2 = \sum_{i=1}^P u_i^2`.


Next to these statistical moments, the contribution of each stochastic parameter to the variance of the objective provides valuable information on the system behavior under uncertainties. 
Generally, this contribution is quantified through Sobol' indices. 
Hence, the total-order Sobol' indices enable to perform a global sensitivity analysis on the stochastic parameters
PCE provides an analytical solution to quantify these Sobol' indices through post-processing of the coefficients (i.e. no additional model evaluations required). 
The total-order Sobol' indices (:math:`S_i^{T,\mathrm{PC}}`) quantify the total impact of a stochastic input parameter, including all mutual interactions. 

:math:`S_i^{T,\mathrm{PC}} = \sum_{\alpha \in A_i^T}^{} u_\alpha^2/\sum_{i=1}^P u_i^2 ~~~~~~ A_i^T = \{\alpha \in A | \alpha_i > 0\}`.

For every coefficient that is characterized by considering input parameter :math:`i`, among others, at an order :math:`> 0` is added to the total Sobol' index :math:`S_i`.

Optimization
------------

The Nondominated Sorting Genetic Algorithm (NSGA-II) is adopted to perform the deterministic and robust design optimization.
A brief introduction to the algorithm is provided below. The details on the method are described in :cite:`Deb2002a`.
Finally, a guideline towards setting the optimization parameters is provided, based on our experience.

.. _lab:ssnsga2:

NSGA-II
^^^^^^^

NSGA-II is a multi-objective genetic algorithm, suitable for optimization of complex, non-linear models :cite:`Deb2002a`. 
First, this algorithm creates a set of design samples (i.e. population), based on Latin Hypercube Sampling. 
Thereafter, a second population (i.e. children) is generated with characteristics based on the previous population (i.e. parents), 
following crossover and mutation rules. Each design sample out of both populations is evaluated in the system model (deterministic design optimization) 
or a PCE is constructed for each quantity of interest (robust design optimization). 
The populations are sorted based on their dominance in the objectives. The top half of the sorted samples remain and represent the next population. 
The algorithm continues until the maximum number of iterations is realized. When the process is finalized, either the solution corresponds to a single design sample, 
which achieves the optimized value for all objectives considered, or a set of optimized design samples where each sample dominates every other design sample in at least one objective.

.. _lab:choosepop:

Choosing the population size
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The computational budget and population size are fixed by the user. Both parameters provide an indication of the number of generations performed,
i.e. number of generations >= computational budget / population size. The population is spread over the design space. The higher the population size,
the higher the exploration of the design space. On the one hand, when the population size is small and the the number of mutations are limited, the population
might converge to a local optimum. On the other hand, when the population size is large, a significant computational budget is spent at each generation,
which limits the exploitation and might result in suboptimal designs. There is no strict rule on the population size, as it highly depends on the number of design variables,
the non-linearity of the input-output relation in the system model and the number of local optima in this relation.
Nevertheless, based on the experience of the authors in engineering optimization (i.e. <10 design variables), the population size is suggested between 20 and 50. Below 20, the possibility of ending in a
local optimum is significant, while a population size larger than 50 does not add significant improvement in design space exploration as opposed to the increase in cost per generation.
Moreover, it is suggested to define the population size based on the number of CPUs available. To illustrate, when 4 CPUs are available, considering defining the population size at 20, 24, 28, 32, ...
such that the CPUs are used at all time during the deterministic design optimization (in robust design optimization, the parallelization is performed on the samples for the PCE, for which the number of samples
is defined by the truncation scheme). 
To ensure sufficient exploitation, we suggest to reach a number of generations of at least 75 generations. Above 250 generations, the gain in exploitation becomes limited. 

The probability of crossover and mutation are user-defined constants which support the exploitation and exploration, respectively.
Typically, the crossover probability is at least 0.85, while the probability of mutation usually remains below 0.1.