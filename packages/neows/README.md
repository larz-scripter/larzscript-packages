# lz-neows

A real client for NASA's Near-Earth-Object Web Service (NeoWs) - every
asteroid NASA has tracked making a close approach to Earth in a date range,
live off `api.nasa.gov`. Install: `larzscript pkg install neows`.

```
import "neows" as neows
import "time" as time

let objects = neows.feed(time.today(), time.today())
for o in objects {
  print(o["name"] + ": " + str(o["velocity_km_s"]) + " km/s, " +
        str(int(o["miss_distance_km"])) + " km miss distance")
}
```

Works out of the box against NASA's public `DEMO_KEY` (rate-limited, no
signup). Pass your own key from https://api.nasa.gov as the last argument
to `feed()`/`lookup()` for production use.

Pairs with [`orbits`](../orbits) - `feed()`'s real diameter and velocity
figures are exactly the inputs `orbits.kinetic_energy_j()` needs to turn a
live tracked object into a real impact-energy estimate; see
[`larzscript-impactwatch`](https://github.com/larz-scripter/larzscript-impactwatch)
for the full worked example.
