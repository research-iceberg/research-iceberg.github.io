# The mixed layer of MiMA
When I sat down and started the endevour of building the [Model of an idealized Moist Atmosphere](https://github.com/mjucker/MiMA),
all my focus was on radiation. After all, MiMA was all about replacing the grey radiation from [Dargan Frierson's model](http://journals.ametsoc.org/doi/abs/10.1175/JAS3753.1),
and not about ocean dynamics.
Now, I soon discovered that if you want to force a dynamic atmosphere with radiation, you need a surface to absorb the incoming solar radiation.
The simplest version of this is a **mixed layer** or **slab ocean**, where the ocean is assumed to be static, and only absorbs or releases energy
based on its assumed depth (which sets its heat capacity). If there is no land, and the surface is assumed to be water everywhere,
this is called an **aquaplanet**.

Of course, I didn't invent this and it was already there in the grey radiation model. But it turned out that the mixed layer was going to
be the most important way of designing numerical experiments for my idealised model studies.
In fact, once I have plugged in [RRTM](http://rtweb.aer.com/rrtm_frame.html), there were only a few ways of influencing the behaviour
(climatology, variability, eddy forcing, etc.) of the atmosphere:
- Ozone: prescribed, can be used to change the base state of the stratosphere
- CO2: just a scalar, warms the troposphere, cools the stratosphere. No localised forcings.
- Mixed layer: only way to add localised forcing, other than ad-hoc diabatic heating (which at the time I didn't add but is available now).

I wanted to get a reasonable climatology, while keeping the number of "tuned" parameters to a minimum.

Here is the problem: **What should the mixed layer depth be?**

One would think it should be what it is in reality. But this is a-priori an aquaplanet, so what do we do with the parts where normally there is no ocean? Also, I don't have actual oceans anyway, so the mixed layer depth really isn't the "real" mixed layer depth.

So, let's just test things:
- deep mixed layer (say 100m):
  - **Good**: Stable climate, possibility to run seasonal cycle.
  - **Bad**: ITCZ moves too slowly as the tropical SSTs cannot react to the moving short wave forcing with the seasonal cycle. The result is a lag of the ITCZ of about three months in its seasonal cycle. Also, the amplitude of the meridional ITCZ motion is almost zero. As a result, a rather strong westerly jet develops in the lower tropical stratosphere (superrotation).
- shallow mixed layer (say 10m):
 - **Good**: ITCZ moves now in phase with the solar forcing.
 - **Bad**: ITCZ moves much too far into the subtropics. Also, the poles now heat a lot in summer (could be fixed with increased albedo simulating ice) and cool a lot in winter (cannot be fixed).This makes it difficult to run a seasonal cycle and impossible to run perpetual simulations.
- intermediate mixed layer (say 20-50m):
 - **Good**: More stable, ITCZ not too lagged, but also not too far poleward during solstice.
 - **Bad**: Poles still rather hot in summer and cold in winter, ITCZ still lagged (less so, but still...).

 That's what I feel everyone should be aware of: **_Depending on what you want to do with an aquaplanet model such as MiMA, it is important to chose a reasonable mixed layer depth. In particular, if one adds land, the much lower heat capacity of land versus water means that whatever works for an aquaplanet might not work for configurations with land_**.

 One solution is implemented in MiMA: Make the mixed layer depth a function of latitude. That way, the tropics are allowed to adjust quickly to the moving solar forcing, whereas the poles are not allowed to cool or heat too fast during the solstice seasons.
