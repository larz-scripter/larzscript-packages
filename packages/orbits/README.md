# lz-orbits

Real two-body orbital mechanics and rocket-propulsion math: Kepler's third
law, the vis-viva equation, escape velocity, the Tsiolkovsky rocket
equation, Hohmann transfers, and impact kinetic energy. Install:
`larzscript pkg install orbits`.

```
import "orbits" as orbits

print(orbits.orbital_period(6779) / 60)          # ISS: ~92.7 minutes
print(orbits.escape_velocity(orbits.EARTH_RADIUS_KM))  # ~11.19 km/s
print(orbits.rocket_delta_v(4130, 50000, 12000)) # m/s, Tsiolkovsky
print(orbits.hohmann_transfer(6771, 42164))      # LEO -> GEO, ~3.86 km/s total
```

Every formula is checked against a well-known real reference figure - the
ISS's real orbital period, Earth's real escape velocity, the textbook
LEO->GEO Hohmann transfer - not just transcribed from a textbook and
trusted. See
[`larzscript-missionbudget`](https://github.com/larz-scripter/larzscript-missionbudget)
for a full showcase app (real rocket-equation propellant costs driving a
real `wallet`/`pay`/`require` budget), and
[`neows`](../neows) + [`larzscript-impactwatch`](https://github.com/larz-scripter/larzscript-impactwatch)
for running these same formulas against NASA's live asteroid-tracking data.
Try both live, no install needed:
[larzos.com/rocket-fuel-calculator/](https://larzos.com/rocket-fuel-calculator/)
and [larzos.com/asteroid-watch/](https://larzos.com/asteroid-watch/).
