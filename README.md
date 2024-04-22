# QuantumNPA

Code to do NPA in Julia. In development - names of important functions or
even the entire project could change.

In this branch, made in collaboration with Abhishek Mishra, we extended the code to allow the solution of generic polynomial optimization problems with commutative or non-commutative variables.

Prerequisites:
```julia
using Pkg; Pkg.add(["Combinatorics", "JuMP", "SCS", "BlockDiagonals", "Mosek", "MosekTools", "FastGaussQuadrature"])
```

First, import the functions and Mosek solver
```julia
using Mosek,MosekTools
include("qnpa.jl")
```


Start generating your variables. In the commutative case 
```julia
x = [freeop(i,1) for i in 1:2]
```
In the non-commutative case 
```julia
x = freeop(1,1:2)
```

Define the objective
```
julia
obj = x[1] * x[2] + x[2] * x[1]
```

Define operator equality constraints. Every expression needs to be given as follows and it will be assumed equal to zero.
```
julia
op_eq = [ x[1]^2 - x[1] ]
```

Finally, specify operator inequality constraints. In this case, the expressions will be assumed greater or equal than zero.
```
julia
op_ge = [ -x[2]^2 + x[2] + 0.5]
```

To solve the minimization problem at level 2 of the hierarchy (by level 2 we mean level 2 of the localizing matrices).
```
julia
npa_general(obj, 2; op_eq, op_ge)
```

If we need to output also the solution for the moment matrices
```
julia
m,Γ,Γx = npa_general(obj, 2; op_eq, op_ge, show_moments=true)
```

The results obtained from this example match the ones in the documentation of the Python package ncpol2sdpa.

One can also force the values of single variables to be constrained. For example, we can force x[1]=1  and x[2]>=0 writing
```
av_eq = [ [x[1], 1] ]
av_ge = [ [x[2], 0] ]
npa_general = npa_general(obj, 2; op_eq, op_ge, av_eq, av_ge)
```

A further example:
```julia
julia> x = [ freeop(i,1) for i in 1:5 ]
5-element Vector{Monomial}:
 A1
 B1
 C1
 D1
 E1

julia> obj(S) = S^2 + x[1]^2 + x[2]^2 - x[3]^2 - 2*S*(x[4]*x[1] + x[5]*x[2])
obj (generic function with 1 method)

julia> op_eq = [x[4]^2+x[5]^2-1]
g1-element Vector{Polynomial}:
 -Id + D1 D1 + E1 E1

julia> op_ge = [1-x[1]^2-x[2]^2, 1-x[1]^2-x[3]^2]
2-element Vector{Polynomial}:
 Id - A1 A1 - B1 B1
 Id - A1 A1 - C1 C1

julia> fmin(S) = npa_general(obj(S), 2; op_eq, op_ge)
fmin (generic function with 1 method)
```

The results can be compared with the analytical solution of the previous problem for 1<S<=2.

```
julia
julia> fanalytical(S) = S^2-2*S
fanalytical (generic function with 1 method)

julia> fmin(1.9)
-0.1899999993223922

julia> fanalytical(1.9)
-0.18999999999999995
```

## Other objects
Besides free operator, one can use the following objects:

```
PA = projector(1,1:2,1:2,full=true) # generates a projector in the subspace A (1) with two inputs and two ouputs. full=true means that each input sums up to the identity
A = dichotomic(1,1:2) # generates two operators that square to the identity in subspace A
```

