# DifferentialEquations.jl

[![Join the chat at https://gitter.im/ChrisRackauckas/DiffEq.jl](https://badges.gitter.im/ChrisRackauckas/DiffEq.jl.svg)](https://gitter.im/ChrisRackauckas/DiffEq.jl?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

This is a package for solving numerically solving differential equations in Julia by Chris Rackauckas. The purpose of this package is to supply efficient Julia implementations of solvers for various differential equations. Equations within the realm of this package include stochastic ordinary differential equations (SODEs or SDEs), stochastic partial differential equations (SPDEs), partial differential equations (with both finite difference and finite element methods), and differential delay equations. For ordinary differential equation solvers, see [ODE.jl](https://github.com/JuliaLang/ODE.jl)

This package is for efficient and parallel implementations of research-level algorithms, many of which are quite recent. These algorithms aim to be optimized for HPC applications, including the use of GPUs, Xeon Phis, and multi-node parallelism.

Currently, finite element solvers for the (Stochastic) Poisson and Heat Equations are supplied. These functions take in a problem specification (with an option for adding stochasticity) and generate solutions to the PDEs. Mesh generation tools currently only work for squares, though the solvers will work on general meshes if they are provided. The mesh layout follows the format of [iFEM](http://www.math.uci.edu/~chenlong/programming.html) and many subroutines in the finite element solver are based off of iFEM algorithms.

If you have any questions, or just want to chat about solvers/using the package, please feel free to message me in the Gitter channel. For bug reports, feature requests, etc., please submit an issue.

# Using the package

To install the package, use the following command inside the Julia REPL:
```julia
Pkg.clone("https://github.com/ChrisRackauckas/DifferentialEquations.jl")
```

To load the package, use the command

```julia
using DifferentialEquations
```

To understand the package in more detail, check out the examples codes in [test/](test/).

## Poisson Equation Finite Element Method Example

In this example we will solve the Poisson Equation Δu=f. The code for this example can be found in [tests/introductionExample.jl](tests/introductionExample.jl). For our example, we will take the linear equation where `f(x) = sin(2π.*x[:,1]).*cos(2π.*x[:,2])`. For this equation we know that solution is `u(x,y,t)= sin(2π.*x).*cos(2π.*y)/(8π*π)` with gradient `Du(x,y) = [cos(2*pi.*x).*cos(2*pi.*y)./(4*pi) -sin(2π.*x).*sin(2π.*y)./(4π)]`. Thus, we define a PoissonProblem as follows:

```julia
"Example problem with solution: u(x,y,t)= sin(2π.*x).*cos(2π.*y)/(8π*π)"
function poissonProblemExample_wave()
  f(x) = sin(2π.*x[:,1]).*cos(2π.*x[:,2])
  sol(x) = sin(2π.*x[:,1]).*cos(2π.*x[:,2])/(8π*π)
  Du(x) = [cos(2*pi.*x[:,1]).*cos(2*pi.*x[:,2])./(4*pi) -sin(2π.*x[:,1]).*sin(2π.*x[:,2])./(4π)]
  gN(x) = 0
  isLinear = true
  return(PoissonProblem(f,sol,Du,gN,isLinear))
end
pdeProb = poissonProblemExample_wave()
```

Note that in this case since the solution is known, the Dirichlet boundary condition `gD` is automatically set to match the true solution. The code for other example problems can be found in [src/examples/exampleProblems.jl](src/examples/exampleProblems.jl). To solve this problem, we first have to generate a mesh. Here we will simply generate a mesh of triangles on the square [0,1]x[0,1] with Δx=2^(-5). To do so, we use the code:

```julia
Δx = 1//2^(5)
femMesh = notime_squaremesh([0 1 0 1],Δx,"Dirichlet")
```

Note that by specifying "Dirichlet" our boundary conditions is set on all boundaries to Dirichlet. This gives an FEMmesh object which stores a finite element mesh in the same layout as [iFEM](http://www.math.uci.edu/~chenlong/programming.html). Notice this code shows that the package supports the use of rationals in meshes. Other numbers such as floating point and integers can be used as well. Finally, to solve the equation we use

```julia
res = fem_solvepoisson(femMesh::FEMmesh,pdeProb::PoissonProblem,solver="GMRES")
```

fem_solvepoisson takes in a mesh and a PoissonProblem and uses the solver to compute the solution. Here the solver was chosen to be GMRES. Other solvers can be found in the documentation. This reurns a FEMSolution object which holds data about the solution, such as the solution values (u), the true solution (uTrue), error estimates, etc. To plot the solution, we use the command

```julia
solplot(res,savefile="introductionExample.png")
```

This gives us the following plot:

![Introduction Example Solution][introductionExampleSolution]

### Stochastic Partial Differential Equation Solvers

We can solve the save stochastic PDE Δu=f+gdW with additive space-time white noise by specifying the problem as:

```julia
"Example problem with deterministic solution: u(x,y,t)= sin(2π.*x).*cos(2π.*y)/(8π*π)"
function poissonProblemExample_noisyWave()
  f(x) = sin(2π.*x[:,1]).*cos(2π.*x[:,2])
  sol(x) = sin(2π.*x[:,1]).*cos(2π.*x[:,2])/(8π*π)
  Du(x) = [cos(2*pi.*x[:,1]).*cos(2*pi.*x[:,2])./(4*pi) -sin(2π.*x[:,1]).*sin(2π.*x[:,2])./(4π)]
  gN(x) = 0
  isLinear = true
  stochastic = true
  σ(x) = 5 #Additive noise
  return(PoissonProblem(f,sol,Du,gN,isLinear,σ=σ,stochastic=stochastic))
end
```

and using the same solving commands. This gives the following output:

![Introduction Stochastic Solution][introductionStochasticSolution]

# Current Supported Equations
* PDE Solvers
  * Finite Element Solvers
    * Linear Poisson Equation
    * Semi-linear Poisson Equation
    * Linear Heat Equation
    * Semi-linear Heat Equation (aka Reaction-Diffusion Equation)

# Roadmap
* SODE Solvers
  * Euler-Maruyama
  * Milstein
  * Rossler-SRK
  * Adaptive-SRK
* PDE Solvers
  * Finite difference solvers:
    * Semi-linear Heat Equation (Reaction-Diffusion Equation)
    * Semi-linear Poisson Equation
    * Wave Equation
    * Transport Equation
    * Stokes Equation
* SPDE Solvers
  * Euler-Maruyama
  * IIF-Maruyama
  * IIF-Milstein

[introductionExampleSolution]: /src/examples/introductionExample.png "Introduction Example Solution" =250x
[introductionStochasticExample]: /src/examples/introductionStochasticExample.png "Introduction Example Solution to the Stochastic Equation" =250x
