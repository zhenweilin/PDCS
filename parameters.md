+++
title = "Parameters"
hascode = true
rss = ""
rss_title = "More goodies"
rss_pubdate = Date(2019, 5, 1)

tags = ["syntax", "code", "image"]
+++


\toc

# Parameters

## Problem Data
- `n`: Number of variables
- `m`: Number of constraints
- `nb`: Number of box constraints
- `c_cpu`: Cost vector of length `n`
- `G_cpu`: Constraint matrix with `m` rows and `n` columns
- `h_cpu`: Right-hand side vector of length `m`

## Cone Specifications
- `mGzero`: Number of zero constraints
- `mGnonnegative`: Number of nonnegative constraints
- `socG`: Vector of sizes for Second-Order Cones (SOC)
- `rsocG`: Vector of sizes for Rotated Second-Order Cones (RSOC)
- `expG`: Number of exponential cones
- `dual_expG`: Number of dual exponential cones
- `soc_x`: Indices of variables in `x` that belong to SOCs
- `rsoc_x`: Indices of variables in `x` that belong to RSOCs
- `exp_x`: Number of exponential cones in variables, default is `0`
- `dual_exp_x`: Number of dual exponential cones in variables, default is `0`

## Bounds and Scaling
- `bl_cpu`: Lower bounds vector for box constraints (length `nb`)
- `bu_cpu`: Upper bounds vector for box constraints (length `nb`)
- `Dl_cpu`: Constraint scaling vector (length `m`), default is `ones(m)`
- `Dr_cpu`: Variable scaling vector (length `n`), default is `ones(n)`
- `rescaling_method`: Data rescaling method, default is `:ruiz_pock_chambolle`
  - `:none`: No rescaling
  - `:ruiz_pock_chambolle`: Ruiz-Pock-Chambolle rescaling
  - `:ruiz_pock_more`: Ruiz-Pock-More rescaling
- `use_preconditioner`: Whether to use preconditioner, default is `true`

## Solution Initialization
- `primal_sol_cpu`: Initial primal solution vector, default is `zeros(n)`
- `dual_sol_cpu`: Initial dual solution vector, default is `zeros(m)`
- `warm_start`: Enable warm start, default is `false`
    - To successfully warmstart the solver `primal_sol`, `dual_sol` and `slack` must all be provided **and** `warm_start` option must be set to `true`.

## Algorithm Control
- `max_outer_iter`: Maximum number of outer iterations, default is `10000`
- `max_inner_iter`: Maximum number of inner iterations, default is `500000`
- `method`: Solution method, default is `:average`. We will frame this setting in the future version
  - `:average`: average + reflection + halpern
  - `:halpern`: Halpern method
  - `:average_no_restart`: Average method without restart
  - `:halpern_no_restart`: Halpern method without restart
- `use_adaptive_restart`: Whether to use adaptive restart, default is `true`
- `use_adaptive_step_size_weight`: Whether to use adaptive step size weight, default is `true`
- `use_aggressive`: Whether to use aggressive mode, default is `true`
- `use_accelerated`: Whether to use acceleration, default is `false`, `true` is not recommend in author's experience, it will be removed in future version
- `use_resolving`: Whether to use resolving, default is `true`. If the normalized duality gap is too negative, then recover initial constraint matrix to let rescaling not work and run
- `use_kkt_restart`: Whether to use KKT restart, default is `false`, it means kkt restart won't check from the start, but it will check if normalized duality gap does not work
- `kkt_restart_freq`: Frequency of KKT restart check, default is `2000`
- `use_duality_gap_restart`: Whether to use duality gap restart, default is `true`
- `duality_gap_restart_freq`: Frequency of duality gap restart check, default is `2000`

## Termination Criteria
- `abs_tol`: Absolute tolerance, default is `1e-6`
- `rel_tol`: Relative tolerance, default is `1e-6`
- `eps_primal_infeasible_low_acc`: Low accuracy primal infeasibility tolerance, default is `1e-12`
- `eps_dual_infeasible_low_acc`: Low accuracy dual infeasibility tolerance, default is `1e-12`
- `eps_primal_infeasible_high_acc`: High accuracy primal infeasibility tolerance, default is `1e-16`
- `eps_dual_infeasible_high_acc`: High accuracy dual infeasibility tolerance, default is `1e-16`

## Monitoring and Output
- `print_freq`: Information printing frequency, default is `2000`
- `check_terminate_freq`: Termination criteria checking frequency, default is `2000`
- `verbose`: Verbosity level, default is `1`
  - `0`: Minimal output
  - `1`: Standard output
  - `2`: Detailed output
- `time_limit`: Maximum solver runtime, default is `Inf`
- `logfile_name`: Name of the log file, default is `nothing`


## API of function `PDCS_GPU.rpdhg_gpu_solve`
```julia
function rpdhg_gpu_solve(;
    n::Integer,
    m::Integer,
    nb::Integer,
    c_cpu::Vector{rpdhg_float},
    G_cpu::AbstractMatrix{rpdhg_float},
    h_cpu::hType,
    mGzero::Integer, # m of Q for zero cone
    mGnonnegative::Integer, # m of Q for positive cone
    socG::Vector{<:Integer},
    rsocG::Vector{<:Integer},
    expG::Integer,
    dual_expG::Integer,
    bl_cpu::Vector{rpdhg_float},
    bu_cpu::Vector{rpdhg_float},
    soc_x::Vector{<:Integer},
    rsoc_x::Vector{<:Integer},
    exp_x::Integer = 0,
    dual_exp_x::Integer = 0,
    Dl_cpu::Vector{rpdhg_float} = ones(m),
    Dr_cpu::Vector{rpdhg_float} = ones(n),
    rescaling_method::Symbol = :ruiz_pock_chambolle,
    use_preconditioner::Bool = true,
    use_adaptive_restart::Bool = true,
    use_adaptive_step_size_weight::Bool = true,
    use_aggressive::Bool = true,
    use_accelerated::Bool = false,
    use_resolving::Bool = true,
    primal_sol_cpu::Vector{rpdhg_float} = zeros(n),
    dual_sol_cpu::Vector{rpdhg_float} = zeros(m),
    warm_start::Bool = false,
    max_outer_iter::Integer = 10000,
    max_inner_iter::Integer = 500000,
    abs_tol::rpdhg_float = 1e-6,
    rel_tol::rpdhg_float = 1e-6,
    eps_primal_infeasible_low_acc::rpdhg_float = 1e-12,
    eps_dual_infeasible_low_acc::rpdhg_float = 1e-12,
    eps_primal_infeasible_high_acc::rpdhg_float = 1e-16,
    eps_dual_infeasible_high_acc::rpdhg_float = 1e-16,
    print_freq::Integer = 2000,
    check_terminate_freq::Integer = 2000,
    verbose::Integer = 1,
    time_limit::Float64 = Inf,
    method::Symbol = :average,
    logfile_name::Union{String, Nothing} = nothing,
    use_kkt_restart::Bool = false,
    kkt_restart_freq::Integer = 2000,
    use_duality_gap_restart::Bool = true,
    duality_gap_restart_freq::Integer = 2000)
```