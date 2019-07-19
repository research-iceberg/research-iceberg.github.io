# The surface of an aquaplanet GCM

## Abstract
This article describes some of the challenges and culprits of working with a mixed layer ocean and trying to arrive at an acceptable climatology with a minimum number of parameters.
We discuss why even the simplest version of the Model of an idealized Moist Atmosphere [MiMA](https://github.com/mjucker/MiMA) [Jucker and Gerber (2017)](http://journals.ametsoc.org/doi/10.1175/JCLI-D-17-0127.1) includes meridional heat flux ("Q-flux"), a meridionally inhomogeneous mixed layer depth, options to reduce the mixed layer depth locally ("land"), as well as options to change surface albedo.

## Introduction

Studying the effects of the stratosphere and its coupling to the troposphere in idealized general circulation models (GCMs) has traditionally been restricted to dry models, where the temperature is simply forced to a prescribed relaxation temperature profile (e.g. [Held and Suarez (1994)](http://journals.ametsoc.org/doi/abs/10.1175/1520-0477(1994)075%3C1825:APFTIO%3E2.0.CO;2), [Polvani and Kushner (2002)](http://www.agu.org/pubs/crossref/2002/2001GL014284.shtml), [Jucker _et al._ (2013)](http://journals.ametsoc.org/doi/abs/10.1175/JAS-D-12-0305.1)). Although idealised moist models have existed for some time in the form of gray radiation models (e.g. [Frierson _et al._ (2006)](http://journals.ametsoc.org/doi/abs/10.1175/JAS3753.1), or [O'Gorman and Schneider (2008)](https://doi.org/10.1175/2007JCLI2065.1)), the stratosphere adds some complexity in terms of vertical and meridional structures which are non-trivial to add to such models.

There was an obvious gap to close in the model hierarchy between the gray radation schemes and full GCMs, which required the inclusion of the radiative effects of stratospheric ozone. Therefore, Ed Gerber and I decided to build the Model of an idealized Moist Atmosphere (MiMA), which represents one step up from the Frierson gray radation model [[Frierson _et al._ (2006)](http://journals.ametsoc.org/doi/abs/10.1175/JAS3753.1), [Frierson _et al._ (2007)](http://journals.ametsoc.org/doi/abs/10.1175/JAS3935.1)] by replacing the gray radiation with the full radiative transfer code RRTM (Rapid Radiative transfer Model, [RRTM, Mlawer _et al._ (1997)](http://doi.wiley.com/10.1029/97JD00237)). A short introductory movie for MiMA can be found in Fig. 1.

[![Figure 1](http://img.youtube.com/vi/8UfaFnGtCrk/0.jpg)](http://www.youtube.com/watch?v=8UfaFnGtCrk "Watch on YouTube")
<p style="font-size:x-small;"><span style="font-weight:bold;">Figure 1</span>: A 30-second introduction to the Model of an idealized Moist Atmosphere (link to YouTube).</p>

Naively, this means simply starting from Frierson's model and exchanging the call to radiation. Everything else could be left as is, as the model has been used very successfully for many studies involving moist dynamics.

In particular, the existing moist model had a boundary layer scheme based on Monin-Obukhov similarity theory, a simplified Betts-Miller convection scheme as well as a mixed layer ocean where shortwave solar radiation gets absorbed and re-emitted as in the longwave. All of these simplified parameterisations of real processes have been carefully built, tuned and tested by the original authors, and it was not our intent to modify them.


## Mixed layer ocean

As might be expected with a model which explicitly solves the full radiative transfer including the shortwave, the role of the mixed layer is rather important: Instead of simply acting as lower boundary for the dynamic atmosphere (plus source of water vapor), it has to properly absorb incoming shortwave radiation, convert the absorbed energy into surface temperature, and then re-radiate in the longwave according to its temperature. As a result, the parameter settings which worked for the gray radiation scheme might not work in MiMA.
Indeed, it soon turned out that by far most of the development time of MiMA went into the mixed layer rather than the new radiation scheme.

### The seasonal cycle at the poles

[Frierson _et al._ (2006)](http://journals.ametsoc.org/doi/abs/10.1175/JAS3753.1) use a mixed layer depth of 10m. If MiMA is run with the same depth, the poles will heat a lot during summer and cool a lot during winter. Setting the radiation up as per Earth's astronomical parameters (obliquity, solar constant, rotation rate, radius), the model will crash because the winter pole will become too cold for the moist dynamics in the model (Fig. 2).


<p style="font-size:x-small;"><span style="font-weight:bold;">Figure 2</span>: polar temperature timeseries until crash</p>


Thus, the mixed layer should be increased at least until the poles do not cause the simulation to fail. For realistic polar temperatures in both winter and summer it should be increased even more, and we found a value of about 100 meters to be about right.

### The seasonal cycle in the tropics

With such a deep mixed layer, the model starts being very stable. Unfortunately, it causes the tropics to be too stable: The InterTropical Convergence Zone (ITCZ) does not show any real seasonal cycle anymore but simply sticks to the equator. The little that it does move is lagged by about three months with respect to the solar forcing, such that such important things as the tropical monsoons around the globe are delayed by weeks to months (Fig. 3).

<p style="font-size:x-small;"><span style="font-weight:bold;">Figure 3</span>: ITCZ (not) moving during the seasonal cycle for deep tropics, and much more moving for very shallow tropics. Maybe add panels showing the mixed layer depth.</p>

Therefore, depending on which phenomena are important for a given research project, the mixed layer should either be deep or shallow. The most realistic cases are obtained if the mixed layer is shallow in the tropics (10-20m), and deep at the poles (100m). Therefore, the heat capacity of the mixed layer can be adjusted to be constant or to have different values in the tropics and the poles, with a linear transition region in-between (Fig. 4).

<p style="font-size:x-small;"><span style="font-weight:bold;">Figure 4</span>: With a meridionally varying mixed layer depth, the ITCZ can move and follow peak solar input more freely, while the poles do not become too cold nor too warm.</p>

### Effect of land

MiMA has the option of adding "land" in the sense of reduced mixed layer depth (the heat capacity of rock is about one thousand times lower than that of water). As a zeroth order approximation, land therefore is still an infinite source of water vapor to the atmosphere. This can either be in the form of an external file (containing a a land-sea mask as is often used in comprehensive models), slaving it to surface topography (by setting an altitude threshold above which any grid point is considered land), or by adding arbitrary rectangles or patches of land (Fig. 5). Then, one can define a different heat capacity for any grid points considered land. As a consequence, whenever adding land, the zonal mean heat capacity is lowered, and the effect of this new mixed layer depth distribution should first be tested before conducting research with the chosen setup. For instance, using a realistic land mask will add the Antarctic continent at the South Pole, which, depending on the setup, might cause problems at high southern latitudes because the South Pole now suddenly has a very low heat capacity (see above discussion).

<p style="font-size:x-small;"><span style="font-weight:bold;">Figure 5</span>: Different land configurations in MiMA.</p>

### Ocean heat transport

As a consequence of neglecting any ocean dynamics is that the incoming solar energy is not transported to the poles as efficiently as often desired (note that the ocean dominates global meridional heat transport in the tropics [Fasullo and Trenberth (2008)](https://doi.org/10.1175/2007JCLI1936.1)). As a result, the ITCZ is very narrow and the tropical-extratropical temperature contrast is rather high. Therefore, as described by [ Merlis _et al._ (2013)](http://journals.ametsoc.org/doi/abs/10.1175/JCLI-D-11-00716.1), a meridional heat flux ("Q-flux") has to be introduced manually. In MiMA, this is done with a simple exponential and zonally symmetric profile following the above paper, and in the cousin model framework [Isca](https://execlim.github.com/Isca), either an external file can be read or a sophisticated algorithm deployed to adjust the heat flux such that the sea surface temperature (SST) matches a reference state [Vallis _et al._ (2018)](https://www.geosci-model-dev.net/11/843/2018/).

One thing to keep in mind here is that the simple default Q-flux is zonally symmetric, and is also active over any regions considered "land" as defined above. Thus, if one strives for realism in the simulated climate and uses real topography and land-sea mask, one might also think about either not using any Q-flux at all or providing an input file with more a sophisticated structure. MiMAv2 again has seen some development in that direction and includes more realistic Q-fluxes which are zero over (realistic) land.

### Albedo

Another a-priori missing component of MiMA is sea ice. This is not a major shortcoming since ocean dynamics, biogeochemistry and others are willingly left out as well. The main effect of sea ice on atmospheric dynamics is arguably the albedo effect. Luckily, the albedo can be adjusted rather easily to account for this effect, at least in a static sense. However, we note that adding e.g. higher albedo south of 65&deg;S to mimic the effect of Antarctica does indeed change the strength and position of the Southern Hemisphere eddy driven jet, but the corrections are minimal. Of course, it could still be important if one's interest lies in the Southern Ocean for instance.

## Conclusions

Note that the default setup for MiMAv1 is constant mixed layer depth. This is to make it as simple as possible, not to make  it as realistic as possible. We will amend this for MiMAv2.

## Acknowledgments

---

## Comments