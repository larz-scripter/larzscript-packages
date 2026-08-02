# lz-genetic

A real genetic algorithm: a population of gene-sequences evolves toward
whatever `fitness_fn` rewards, via tournament selection, single-point
crossover, per-gene mutation, and elitism (the current best always
survives unchanged). Genes are opaque to this package, so it optimizes over
any search space you can describe that way, not just the classic
string-matching demo. Install: `larzscript pkg install genetic`.

```
import "genetic" as ga
import "random" as rng
rng.seed(1)
let target = "LARZ"
fn random_letter() { return chr(65 + rng.randint(0, 25)) }
fn fitness(genes) {
  let score = 0
  let i = 0
  while i < len(genes) { if genes[i] == target[i:i+1] { score = score + 1 } i = i + 1 }
  return score
}
let pop = ga.population_random(60, len(target), random_letter)
let final = ga.evolve(pop, fitness, random_letter, {"generations": 60})
let best = ga.best_of(final)
print(join(best["genes"], ""), best["fitness"])   # LARZ 4
```

See it live at
[larzos.com/stack/larzgenetic/](https://larzos.com/stack/larzgenetic/).
