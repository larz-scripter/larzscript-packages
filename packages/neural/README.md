# lz-neural

A tiny autograd engine and neural network, in the spirit of Karpathy's
micrograd: every number is a `Value` node that remembers how it was
computed, forward and backward - reverse-mode automatic differentiation,
the same algorithm every real deep learning framework runs, shown at the
scalar level where you can watch it work. Install:
`larzscript pkg install neural`.

```
import "neural" as nn
import "random" as rng

let a = nn.value(-4.0)
let b = nn.value(2.0)
let c = nn.add(nn.mul(a, b), nn.tanh_(b))
nn.backward(c)
print(round(a["grad"], 4), round(b["grad"], 4))   # dc/da, dc/db

rng.seed(1)
let m = nn.mlp(2, [4, 1])            # 2 inputs -> 4 hidden -> 1 output
let out = nn.mlp_call(m, [nn.value(1.0), nn.value(-1.0)])
```

See it live, with a real trained-loss example, at
[larzos.com/stack/larzneural/](https://larzos.com/stack/larzneural/).
