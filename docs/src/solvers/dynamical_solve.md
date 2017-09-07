# Dynamical, Hamiltonian, and 2nd Order ODE Solvers

These algorithms require an ODE defined in the following ways:

```julia
DynamicalODEProblem{isinplace}(f1,f2,u0,v0,tspan;kwargs...)
SecondOrderODEProblem{isinplace}(f,u0,du0,tspan;kwargs...)
HamiltonianProblem{T}(H,q0,p0,tspan;kwargs...)
```

These correspond to partitioned equations of motion:

```math
\frac{du}{dt} = f_1(v) \\
\frac{dv}{dt} = f_2(t,u) \\
```
The functions should be specified as `f1(t,u,v,dx)` and `f2(t,u,v,dv)`
(in the inplace form), where `f1` is independent of `t` and `u`, and unless
specified by the solver, `f2` is independent of `v`. This includes
discretizations arising from `SecondOrderODEProblem`s where the velocity is not
used in the acceleration function, and Hamiltonians where the potential is
(or can be) time-dependent but the kinetic energy is only dependent on `v`.

Note that some methods assume that the integral of `f1` is a quadratic form. That
means that `f1=v'*M*v`, i.e. ``\int f_1 = \frac{1}{2} m v^2``, giving `du = v`. This is
equivalent to saying that the kinetic energy is related to ``v^2``. The methods
which require this assumption will lose accuracy if this assumption is violated.
Methods listed below make note of this requirement with "Requires quadratic
kinetic energy".

## Recommendations

When energy conservation is required, use a symplectic method. Otherwise the
Runge-Kutta-Nyström methods will be more efficient. Energy is mostly conserved
by Runge-Kutta-Nyström methods, but is not conserved for long time integrations.
Thus it is suggested that for shorter integrations you use Runge-Kutta-Nyström
methods as well.

As a go-to method for efficiency, `DPRKN6` is a good choice. `DPRKN12` is a good
choice when high accuracy, like `tol<1e-10` is necessary. However, `DPRKN6` is
the only Runge-Kutta-Nyström method with a higher order interpolant (all default
to order 3 Hermite, whereas `DPRKN6` is order 6th interpolant) and thus in cases
where interpolation matters (ex: event handling) one should use `DPRKN6`. For
very smooth problems with expensive acceleration function evaluations, `IRKN4`
can be a good choice as it minimizes the number of evaluations.

For symplectic methods, higher order algorithms are the most efficient when higher
accuracy is needed, and when less accuracy is needed lower order methods do better.
Optimized efficiency methods take more steps and thus have more force calculations
for the same order, but have smaller error. Thus the "optimized efficiency"
algorithms are recommended if your force calculation is not too sufficiency large,
while the other methods are recommend when force calculations are really large
(for example, like in MD simulations `VelocityVerlet` is very popular since it only
requires one force calculation per timestep). A good go-to method would be `McAte5`,
and a good high order choice is `KahanLi8`.

## Standard ODE Integrators

The standard ODE integrators will work on Dynamical ODE problems via a
transformation to a first-order ODE. See the [ODE solvers](../ode_solve.html)
page for more details.

## Specialized OrdinaryDiffEq.jl Integrators

Unless otherwise specified, the OrdinaryDiffEq algorithms all come with a
3rd order Hermite polynomial interpolation. The algorithms denoted as having a
"free" interpolation means that no extra steps are required for the
interpolation. For the non-free higher order interpolating functions, the extra
steps are computed lazily (i.e. not during the solve).

### Runge-Kutta-Nyström Integrators

- `Nystrom4`: 4th order explicit Runge-Kutta-Nyström method. Allows acceleration
  to depend on velocity. Fixed timestep only.
- `IRKN3`: 4th order explicit two-step Runge-Kutta-Nyström method. Fixed
  timestep only.
- `IRKN4`: 4th order explicit two-step Runge-Kutta-Nyström method. Can be more
  efficient for smooth problems. Fixed timestep only.
- `ERKN4`: 4th order Runge-Kutta-Nyström method which is integrates the periodic
  properties of the harmonic oscillator exactly. Gets extra efficiency on periodic
  problems.
- `Nystrom4VelocityIndependent`: 4th order explicit Runge-Kutta-Nyström method.
  Fixed timestep only.
- `Nystrom5VelocityIndependent`: 5th order explicit Runge-Kutta-Nyström method.
  Fixed timestep only.
- `DPRKN6`: 6th order explicit adaptive Runge-Kutta-Nyström method. Free 6th
  order interpolant.
- `DPRKN8`: 8th order explicit adaptive Runge-Kutta-Nyström method.
- `DPRKN12`: 12th order explicit adaptive Runge-Kutta-Nyström method.

### Symplectic Integrators

Note that all symplectic integrators are fixed timestep only.

- `SymplecticEuler`: First order explicit symplectic integrator
- `VelocityVerlet`: 2nd order explicit symplectic integrator.
- `VerletLeapfrog`: 2nd order explicit symplectic integrator.
- `PseudoVerletLeapfrog`: 2nd order explicit symplectic integrator.
- `McAte2`: Optimized efficiency 2nd order explicit symplectic integrator.
- `Ruth3`: 3rd order explicit symplectic integrator.
- `McAte3`: Optimized efficiency 3rd order explicit symplectic integrator.
- `CandyRoz4`: 4th order explicit symplectic integrator.
- `McAte4`: 4th order explicit symplectic integrator. Requires quadratic
  kinetic energy.
- `CalvoSanz4`: Optimized efficiency 4th order explicit symplectic integrator.
- `McAte42`: 4th order explicit symplectic integrator. (Broken)
- `McAte5`: Optimized efficiency 5th order explicit symplectic integrator.
  Requires quadratic kinetic energy
- `Yoshida6`: 6th order explicit symplectic integrator.
- `KahanLi6`: Optimized efficiency 6th order explicit symplectic integrator.
- `McAte8`: 8th order explicit symplectic integrator.
- `KahanLi8`: Optimized efficiency 8th order explicit symplectic integrator.
- `SofSpa10`: 10th order explicit symplectic integrator.