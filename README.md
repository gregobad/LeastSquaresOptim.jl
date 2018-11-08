[![Build Status](https://travis-ci.org/matthieugomez/LeastSquaresOptim.jl.svg?branch=master)](https://travis-ci.org/matthieugomez/LeastSquaresOptim.jl)
[![Coverage Status](https://coveralls.io/repos/matthieugomez/LeastSquaresOptim.jl/badge.svg?branch=master&service=github)](https://coveralls.io/github/matthieugomez/LeastSquaresOptim.jl?branch=master)
## Motivation

This package solves non linear least squares optimization problems. The package is inspired by the [Ceres library](http://ceres-solver.org/nnls_solving.html).  This package is written with large scale problems in mind (in particular for sparse Jacobians). 


## Simple Syntax
The symple syntax mirrors the `Optim.jl` syntax

```julia
using LeastSquaresOptim
function rosenbrock(x)
	[1 - x[1], 100 * (x[2]-x[1]^2)]
end
x0 = zeros(2)
optimize(rosenbrock, x0, Dogleg())
optimize(rosenbrock, x0, LevenbergMarquardt())
```
You can also add the options : `x_tol`, `f_tol`, `g_tol`, `iterations`, `Δ` (initial radius), and `autodiff`.


## Alternative Syntax

Use the alternative syntax is useful when specifying in-place functions or the jacobian. The alternative syntax requires a `LeastSquaresProblem` object with:
 - `x` an initial set of parameters.
 - `f!(out, x)` that writes `f(x)` in `out`.
 - the option `output_length` to specify the length of the output vector. 
 - Optionally, `g!` a function such that `g!(out, x)` writes the jacobian at x in `out`. Otherwise, the jacobian will be computed following the `:autodiff` argument.

```julia
using LeastSquaresOptim
function rosenbrock_f!(out, x)
 out[1] = 1 - x[1]
 out[2] = 100 * (x[2]-x[1]^2)
end
optimize!(LeastSquaresProblem(x = zeros(2), f! = rosenbrock_f!, output_length = 2, autodiff = :central), Dogleg())

# if you want to use gradient
function rosenbrock_g!(J, x)
    J[1, 1] = -1
    J[1, 2] = 0
    J[2, 1] = -200 * x[1]
    J[2, 2] = 100
end
optimize!(LeastSquaresProblem(x = zeros(2), f! = rosenbrock_f!, g! = rosenbrock_g!, output_length = 2), Dogleg())
```

## Choice of Optimizer / Least Square Solver
- You can specify two least squares optimizers, `Dogleg()` and `LevenbergMarquardt()`
- You can specify three least squares solvers, used within the optimizer
	- `LeastSquaresOptim.QR()`. Available for dense jacobians
	- `LeastSquaresOptim.Cholesky()`. Available for dense jacobians
	- `LeastSquaresOptim.LSMR()`. A conjugate gradient method ([LSMR]([http://web.stanford.edu/group/SOL/software/lsmr/) with diagonal preconditioner). 	You can optionally specifying a function `preconditioner!` and a matrix `P` such that `preconditioner(x, J, P)` updates `P` as a preconditioner for `J'J` in the case of a Dogleg optimization method, and such that `preconditioner(x, J, λ, P)` updates `P` as a preconditioner for `J'J + λ` in the case of LevenbergMarquardt optimization method. By default, the preconditioner is chosen as the diagonal of of the matrix `J'J`. The preconditioner can be any type that supports `A_ldiv_B!(x, P, y)`

The optimizers and solvers are presented in more depth in the [Ceres documentation](http://ceres-solver.org/nnls_solving.html). For dense jacobians, the default option is `Doglel(QR())`. For sparse jacobians, the default option is  `LevenbergMarquardt(LSMR())`

```julia
optimize!(LeastSquaresProblem(x = x, f! = rosenbrock_f!, output_length = 2), Dogleg())
optimize!(LeastSquaresProblem(x = x, f! = rosenbrock_f!, output_length = 2), Dogleg(LeastSquaresOptim.QR()))
```


## Duck Typing
When using the solver `LeastSquaresOptim.LSMR()`, the jacobian can be any type that defines the following interface:
	    - `mul!(y, A, x, α::Number, β::Number)` updates y to αAx + βy
		- `mul!(x, A', y, α::Number, β::Number)` updates x to αA'y + βx
		- `colsumabs2!(x, A)` updates x to the sum of squared elements of each column
		- `size(A, d)` returns the nominal dimensions along the dth axis in the equivalent matrix representation of A.
		- `eltype(A)` returns the element type implicit in the equivalent matrix representation of A.
Similarly, `x` or `f(x)` may be custom types. An example of the interface to define can be found in the package [SparseFactorModels.jl](https://github.com/matthieugomez/SparseFactorModels.jl).


## Related packages
Related:
- [LSqfit.jl](https://github.com/JuliaOpt/LsqFit.jl) is a higher level package to fit curves (i.e. models of the form y = f(x, β))
- [Optim.jl](https://github.com/JuliaOpt/Optim.jl) solves general optimization problems.
- [NLSolve.jl](https://github.com/EconForge/NLsolve.jl) solves non linear equations by least squares minimization.


## References
- Nocedal, Jorge and Stephen Wright *An Inexact Levenberg-Marquardt method for Large Sparse Nonlinear Least Squares*  (1985) The Journal of the Australian Mathematical Society
- Fong, DC. and Michael Saunders. (2011) *LSMR: An Iterative Algorithm for Sparse Least-Squares Problems*.  SIAM Journal on Scientific Computing
- Agarwal, Sameer, Keir Mierle and Others. (2010) *Ceres Solver*

## Installation
To install the package,
```julia
using Pkg
Pkg.add("LeastSquaresOptim")
```
