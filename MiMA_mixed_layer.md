# The surface of an aquaplanet GCM

## Abstract
This article describes some of the challenges and culprits of working with a mixed layer ocean and trying to arrive at an acceptable climatology with a minimum number of parameters.
We discuss why even the simplest version of the Model of an idealized Moist Atmosphere [MiMA](https://github.com/mjucker/MiMA) [Jucker and Gerber (2017)](http://journals.ametsoc.org/doi/10.1175/JCLI-D-17-0127.1) includes meridional heat flux ("Q-flux"), and a meridionally inhomogeneous mixed layer depth as well as options to change surface albedo.

## Introduction

Studying the effects of the stratosphere and its coupling to the troposphere in idealized general circulation models (GCMs) has traditionally been restricted to dry models, where the temperature is simply forced to a prescribed relaxation temperature profile (e.g. [Held and Suarez (1994)](http://journals.ametsoc.org/doi/abs/10.1175/1520-0477(1994)075%3C1825:APFTIO%3E2.0.CO;2), [Polvani and Kushner (2002)](http://www.agu.org/pubs/crossref/2002/2001GL014284.shtml), [Jucker _et al._ (2013)](http://journals.ametsoc.org/doi/abs/10.1175/JAS-D-12-0305.1)). Although idealised moist models have existed for some time (e.g. [Frierson _et al._ (2006)](http://journals.ametsoc.org/doi/abs/10.1175/JAS3753.1), [tk find Tapio's model Schneider et al ????]()), the stratosphere adds some complexity in terms of vertical and meridional structures which are non-trivial to add to such models.

There was an obvious gap to close in the model hierarchy between the gray radation schemes and full GCMs, which required the inclusion of the radiative effects of stratospheric ozone. Therefore, we decided to build MiMA, which represents one step up from the Frierson gray radation model [[Frierson _et al._ (2006)](http://journals.ametsoc.org/doi/abs/10.1175/JAS3753.1), [Frierson _et al._ (2007)](http://journals.ametsoc.org/doi/abs/10.1175/JAS3935.1)] by replacing the gray radiation with the full radiative transfer code RRTM (Rapid Radiative transfer Model, [RRTM, Mlawer _et al._ (1997)](http://doi.wiley.com/10.1029/97JD00237)). A short introductory movie for MiMA can be found in Fig. 1.

[![Figure 1](http://img.youtube.com/vi/8UfaFnGtCrk/0.jpg)](http://www.youtube.com/watch?v=8UfaFnGtCrk "Watch on YouTube")
<p style="font-size:x-small;">Figure 1: A 30-second introduction to the Model of an idealized Moist Atmosphere</p>

Naively, this means simply starting from Frierson's model and exchanging the call to radiation. Everything else could be left as is, as the model has been used very successfully for many studies involving moist dynamics [tk Frierson model applications].

In particular, the existing moist model had what is called a boundary layer scheme based on Monin-Obukhov similarity theory, a simplified Betts-Miller convection scheme as well as a mixed layer ocean where shortwave solar radiation gets absorbed and re-emitted as longwave. All of these simplified parameterisations of real processes have been carefully built, tuned and tested by the original authors, and it was not our intent to modify them.


## Mixed layer ocean

As might be expected with a model which explicitly solves the full radiative transfer including the shortwave, the role of the mixed layer is rather important: Instead of simply acting as lower boundary for the dynamic atmosphere (plus source of water vapor), it has to properly absorb incoming shortwave radiation, convert the absorbed energy into surface temperature and then re-radiate in the longwave according to its temperature. As a result, the parameter settings which worked for the gray radiation scheme might not work in MiMA.
Indeed, it soon turned out that by far most of the development time of MiMA went into the mixed layer rather than the new radiation scheme.

### The seasonal cycle at the poles

[Frierson _et al._ (2006)](http://journals.ametsoc.org/doi/abs/10.1175/JAS3753.1) use a mixed layer depth of 10m. If MiMA is run with the same depth, the poles will heat a lot during summer and cool a lot during winter. Setting the radiation up as per Earth's astronomical parameters (obliquity, solar constant, rotation rate, radius), the model will crash because the winter pole will become too cold for the moist dynamics (Fig. 2).

[tk Figure 2 shows polar temperature timeseries until crash]

Thus, the mixed layer should be increased at least until the poles do not cause the simulation to fail. For realistic polar temperatures in both winter and summer it should be increased even more, and we found a value of about 100 meters to be about right.

### The seasonal cycle in the tropics

With such a deep mixed layer, the model starts being very stable. Unfortunately, it causes the tropics to be too stable: The ITCZ [tk acronym?] does not show any real seasonal cycle anymore but simply sticks to the equator. The little that it does move is lagged by about three months with respect to the solar forcing, such that such important things as the tropical monsoons around the globe are delayed by weeks to months (Fig. 3).

[tk Figure 3 shows ITCZ (not) moving during the seasonal cycle for deep tropics, and much more moving for very shallow tropics. Maybe add panels showing the mixed layer depth.]

Therefore, depending on which phenomena are important for a given research project, the mixed layer should either be deep or shallow. The most realistic cases are obtained if the mixed layer is shallow in the tropics (10-20m), and deep at the poles (~100m). Therefore, the heat capacity of the mixed layer can be adjusted to be constant or to have different values in the tropics and the poles, with a linear transition region in-between (Fig. 3).

### Effect of land

MiMA has the option of adding land in the sense of reduced mixed layer depth (the heat capacity of rock is about one thousand times lower than that of water). This can either be in the form of an external file (for real continents for instance), or by adding arbitrary rectangles or patches of land, and defining a different heat capacity for any grid points considered land. As a consequence, whenever adding land, the zonal mean heat capacity is lowered, and the effect of this new mixed layer depth distribution should first be tested before conducting research with the chosen setup. For instance, using a realistic land mask will add the Antarctic continent at the South Pole, which, depending on the setup, might cause problems at high southern latitudes.

### Ocean heat transport

As a consequence of neglecting any ocean dynamics is that the incoming solar energy is not transported to the poles as efficiently as often desired (note that the ocean dominates global meridional heat transport in the tropics). As a result, the ITCZ is very narrow and the tropical-extratropical temperature contrast is rather high. Therefore, as described by [ Merlis _et al._ (2013)](http://journals.ametsoc.org/doi/abs/10.1175/JCLI-D-11-00716.1), a meridional heat flux has to be introduced manually. In MiMA, this is done with a simple exponential and zonally symmetric profile following [Merlis _et al._ (2013)](http://journals.ametsoc.org/doi/abs/10.1175/JCLI-D-11-00716.1), and in the cousin model framework [Isca](https://execlim.github.com/Isca), either an external file can be read or a sophisticated algorithm deployed to adjust the heat flux such that the sea surface temperature (SST) matches a reference state [Vallis _et al._ (2018)](https://www.geosci-model-dev.net/11/843/2018/).

### Albedo

Another a-priori missing component of MiMA is sea ice. This is not a major shortcoming since ocean dynamics, biogeochemistry and others are willingly left out as well. The main effect of sea ice on atmospheric dynamics is arguably the albedo effect. Luckily, the albedo can be adjusted rather easily to account for this effect, at least in a static sense. However, we note that adding e.g. higher albedo south of 65S to mimic the effect of Antarctica does indeed change the strength and position of the Southern Hemisphere eddy driven jet, but the corrections are minimal.

## Conclusions



## Acknowledgments

---

## Comments





When I sat down and started the endeavour of building the [Model of an idealized Moist Atmosphere](https://github.com/mjucker/MiMA),
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

 ## Comments
