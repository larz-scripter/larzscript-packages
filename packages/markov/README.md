# lz-markov

A small Markov-chain text generator, pure Larzscript - zero network, zero
deps. `larzscript larzpkg.lz install markov`.

```
import "markov" as markov
let chain = markov.train(["the quick brown fox", "the lazy dog sleeps"])
print(markov.generate(chain, 12))
```
