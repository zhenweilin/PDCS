+++
title = "Examples"
hascode = true
date = Date(2019, 3, 22)
rss = "A short description of the page which would serve as **blurb** in a `RSS` feed; you can use basic markdown here but the whole description string must be a single line (not a multiline string). Like this one for instance. Keep in mind that styling is minimal in RSS so for instance don't expect maths or fancy styling to work; images should be ok though: ![](https://upload.wikimedia.org/wikipedia/en/3/32/Rick_and_Morty_opening_credits.jpeg)"

tags = ["syntax", "code"]
+++


# Examples

\toc

Here, we give some examples about using pdcs api. 



### exponential cone examples
##### Fisher market equilibrium
$$
min_{\mathbf{X}\in\mathbb{R}^{m\times n}} - \sum_{i\in[m]} w_i \left( \log \left( \sum_{j\in[n]}\mathbf{U}_{ij}\mathbf{X}_{ij}\right) \right) \text{s.t.} \sum_{i\in[m]}\mathbf{X}_{ij}=b_j, \mathbf{X}_{ij}\geq 0
$$
```julia
using LinearAlgebra
using SparseArrays
using Random
using SparseArrays
include("PDCS_GPU.jl")
m = 100
n = 300
rng = Random.MersenneTwister(1)
w = rand(rng, m)
u = rand(rng, m * n)
u .= abs.(u)
u[rand(rng, m * n) .< 0.8] .= 0.0
u = reshape(u, m, n)
sparse_u = sparse(u)
nnz_u = nnz(sparse_u)
println(nnz_u)
btilde = 0.25*m*ones(n)

m, n = size(u)
b = zeros(m + n + 3m)
b[1:n] .= btilde
for i in 1:m
    b[n + m + 3(i - 1) + 2] = -1.0
end
u = vec(u')
sparse_u = sparse(u)
nnz_u = nnz(sparse_u)
println(nnz_u)
row_indices = zeros(Int, nnz_u)
for i = 1:nnz_u
    row_indices[i] = (sparse_u.nzind[i] - 1) ÷ n + 1 + n
end

c = zeros(m*n + 2*m)
@views c[m*n+1:2:m*n+2*m-1] .= -w
rows = repeat(1:n, m)
cols = repeat((0:(m-1)) * n, inner=n) .+ repeat(1:n, m)
values = ones(n * m)
row2 = row_indices
col2 = sparse_u.nzind
val2 = sparse_u.nzval
row = vcat([rows, row2]...)
col = vcat([cols, col2]...)
val = vcat([values, val2]...)
row3 = (n+1) : (n+m)
col3 = m * n .+ 2 .* (1:m)  
val3 = fill(-1.0, m)

row = vcat([row, row3]...)
col = vcat([col, col3]...)
val = vcat([val, val3]...)

row4 = vcat([[3*i + 1, 3i+3] for i in 0:(m-1)]...) .+ (m + n)
col4 = Vector{Integer}(1:2m)
col4 = col4 .+ (m * n)
val4 = fill(1.0, 2m)
row = vcat([row, row4]...)
col = vcat([col, col4]...)
val = vcat([val, val4]...)
A = sparse(row, col, val, m + n + 3m, m*n + m + m)
mA, nA = size(A)
bl = fill(0.0, nA)
bl[m*n+1:m*n+2*m] .= -Inf
bu = fill(Inf, nA)
# expG = Vector{Integer}(ones(m) * 3)
sol_res = PDCS_GPU.rpdhg_gpu_solve(
    n = nA,
    m = mA,
    nb = nA,
    c_cpu = c,
    G_cpu = A,
    h_cpu = b,
    mGzero = m + n,
    mGnonnegative = 0,
    socG = Vector{Integer}([]),
    rsocG = Vector{Integer}([]),
    expG = m,
    dual_expG = 0,
    bl_cpu = bl,
    bu_cpu = bu,
    soc_x = Vector{Integer}([]),
    rsoc_x = Vector{Integer}([]),
    exp_x = 0,
    dual_exp_x = 0,
    print_freq = 100,
    use_preconditioner = true,
    rescaling_method = :ruiz_pock_chambolle,
    method = :average,
    time_limit = 1000000.0,
    use_adaptive_restart = true,
    use_adaptive_step_size_weight = true,
    use_resolving = true,
    use_accelerated = false,
    use_aggressive = true,
    verbose = 2,
    rel_tol = 1e-6,
    abs_tol = 1e-6,
);
```