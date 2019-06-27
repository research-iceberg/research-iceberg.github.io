# The mixed layer of MiMA
When I sat down and started the endevour of building the [Model of an idealized Moist Atmosphere](https://github.com/mjucker/MiMA),
all my focus was on radiation. After all, MiMA was all about replacing the grey radiation from [Dargan Frierson's model](http://journals.ametsoc.org/doi/abs/10.1175/JAS3753.1),
and not about ocean dynamics.
Now, I soon discovered that if you want to force a dynamic atmosphere with radiation, you need a surface to absorb the incoming solar radiation.
The simplest version of this is a **mixed layer** or **slab ocean**, where the ocean is assumed to be static, and only absorbs or releases energy
based on it's assumed depth (which sets it's heat capacity). If there is no land, and the surface is assumed to be water everywhere,
this is called an **aquaplanet**.

Of course, I didn't invent this and it was already there in the grey radiation model. But it turned out that the mixed layer was going to
be the most important way of designing numerical experiments for my idealised model studies.
In fact, once I have plugged in [RRTM](http://rtweb.aer.com/rrtm_frame.html), there were only a few ways of influencing the behaviour 
(climatology, variability, eddy forcing, etc.) of the atmosphere:
- Ozone: prescribed, can be used to change the base state of the stratosphere
- CO2: just a scalar, warms the troposphere, cools the stratosphere. No localised forcings.
- Mixed layer: only way to add localised forcing, other than ad-hoc diabatic heating (which at the time I didn't add but are available now).
Now there is a problem: The mixed layer sets up the climatology, but is also the main tool of changing the forcing of the model. But first, all I wanted 

I wanted to get a reasonable climatology, while keeping the number of "tuned" parameters to a minimum. 

Here is the problem: 
