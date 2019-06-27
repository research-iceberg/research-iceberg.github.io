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

Why?

