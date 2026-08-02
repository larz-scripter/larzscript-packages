# lz-raytrace

A real ray tracer, in the spirit of "Ray Tracing in One Weekend":
ray-sphere intersection by the quadratic formula, real lambertian
(diffuse) and metal (reflective, with fuzz) materials, recursive light
bounces, antialiasing, gamma-corrected output, and a plain PPM (P3) image -
a real, viewable render, not a toy. No image library, no GPU. Install:
`larzscript pkg install raytrace`.

```
import "raytrace" as rt
import "random" as rng
rng.seed(1)
let ground = rt.sphere(rt.vec3(0, -100.5, -1), 100, rt.lambertian(rt.vec3(0.8, 0.8, 0.0)))
let center = rt.sphere(rt.vec3(0, 0, -1), 0.5, rt.lambertian(rt.vec3(0.7, 0.3, 0.3)))
let metal_ball = rt.sphere(rt.vec3(1, 0, -1), 0.5, rt.metal(rt.vec3(0.8, 0.6, 0.2), 0.3))
let ppm = rt.render([ground, center, metal_ball], 100, 50, 20, 8)
write_file("scene.ppm", ppm)
```

See it live, with real captured output, at
[larzos.com/stack/larzraytrace/](https://larzos.com/stack/larzraytrace/).
