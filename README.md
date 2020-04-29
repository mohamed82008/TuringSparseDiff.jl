```julia

using Turing, SparsityDetection
using Turing.DynamicPPL: VarInfo, LazyVarInfo, resetlogp!, getlogpvec, Sampler
Turing.setadbackend(:sparseforwarddiff)

@model demo() = begin
    m ~ Normal()
end
model = demo()
varinfo = VarInfo(model)
lvi = LazyVarInfo(varinfo)
resetlogp!(lvi)
spl = Sampler(HMC(0.01, 1), model)
model(lvi, spl)
N = lvi.lastidx[]
resetlogp!(lvi)

function f(out, x)
    new_vi = VarInfo(lvi, spl, x)
    model(new_vi, spl)
    out .= getlogpvec(new_vi)
    return out
end

x = rand(1)
y = zeros(1)
f(y, x)
jacobian_sparsity(f, y, x)

# If the above works, the following line should also work.
# sample(demo(), HMC(0.01, 1), 10)
```
