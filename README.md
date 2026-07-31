<div align="center">

![ALL DOWN HILL](media/key-art.jpg)

# There is no lift back up.

You dropped in after last chair. The cats have gone in, the corduroy is setting up hard, and the light is going fast. Nobody is coming to sweep the mountain.

Point it down the fall line and see how long you last.

Then hand the mountain to someone else and watch where they fall.

**[▶ &nbsp;DROP IN](https://inkxel.github.io/all-down-hill/)**

<sub>Browser · No install · Desktop and phone</sub>

</div>

---

## The run

The mountain builds itself a few hundred metres ahead of you and forgets everything behind. Unless somebody sent you here, nobody has ever ridden it.

You get four minutes of daylight. Dawn comes up pink, burns off to hard midday glare, drops into gold, then dusk, then you're riding by moonlight with the fog closed in around you. Weather runs to its own schedule and doesn't care what the sky is doing — flurries, a whiteout that pulls the horizon in to nothing, wind that blows the snow sideways past your edge.

Everything on that mountain is a takeoff if you read it right. A rock is a launch. The crown of a pine will kick you higher than the jump you meant to take. A downed log is a rail if you can get onto it clean. String them together without touching snow and they stack — and it all cashes in the moment you stomp the landing.

Hit anything wrong and you're picking yourself out of the snow with three ways back on.

## Somebody else's mountain

Every mountain has a name — a short code, which is really just the number it grew from. Same code, same mountain: the same ridges, the same trees standing in the same places, the same storm arriving at the same point on the hill.

So you can give it away. **Share this mountain** copies a link, and everything the mountain knows travels inside the link itself — no account, no server, nothing stored anywhere. Whoever opens it drops into the same snow in the same weather, with the tracks of everyone who came before laid faintly over it. A gap in a line is where they were in the air. The end of a line is where they went down, and their gear is still lying there.

Ride it and your line joins the link. The five best runs survive each hop, plus yours whether it placed or not — so you can always pass it on, and a mountain that's been around a while gets hard to get onto.

They're traces, not rivals. There's no clock in them, so you can't race a ghost. You can only see how far they got.

## Riding it

|  | Desktop | Phone |
|---|---|---|
| **Carve** | `A` `D` or `←` `→` | Tilt |
| **Pop** | `SPACE` | Tap |
| **Spin** | Hold, release to set up for the landing | Hold |
| **Pause** | `ENTER` | Two fingers |
| **Sound** | `M` | The speaker, bottom right |

**Timing beats mashing.** There's no charge meter. Pop off the lip of a roller and your own speed throws you; pop on the flats and you'll barely clear the snow. The mountain decides how much air you get — you decide when.

**Let go early.** Hold to spin, release and the board eases round to the nearest full rotation, so a half-committed flip straightens itself out. Hold it all the way into the snow and you'll wear it.

**Carve hard into the takeoff and the spin goes over instead of round.** You're already banked, so it corkscrews. Leave it flat and you get the flat spin.

**Bail off the log before it runs out.** Riding off the end drops you back to the snow. Popping off the end is worth twice as much.

<div align="center">
<img src="media/shot-golden.jpg" width="49%"> <img src="media/shot-night.jpg" width="49%">
<img src="media/shot-dawn.jpg" width="49%"> <img src="media/shot-day.jpg" width="49%">
<br><sub>One run, four minutes apart</sub>
</div>

## Running it

It is one `index.html`. Three.js from a CDN, and then the file — no build step, nothing to install. Every ridge, pine, rock, sky colour and sound is generated at load rather than shipped as an asset, which is why the repo has no art in it.

```
git clone https://github.com/inkxel/all-down-hill.git
cd all-down-hill && python3 -m http.server
```

---

<div align="center">
<sub>Built by <a href="https://tvcker.com">Tucker</a> · MIT</sub>
</div>
