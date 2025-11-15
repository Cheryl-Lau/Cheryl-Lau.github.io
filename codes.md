---
permalink: /codes
---
[HOME](README.md) &emsp; [PROFILE](profile.md) &emsp; [RESEARCH](research.md) &emsp; [CODES](codes.md) &emsp; [PHOTOS](photos.md) &emsp; [CONTACT](contact.md)

## Simulation codes 

All simulation codes I used in my work are publicly available in [my github](https://github.com/Cheryl-Lau).


<br/>
### Phantom SPH (fork) 

The SPH code I mainly use is [Phantom](https://phantomsph.github.io/), developed by [Daniel Price and collaborators](https://doi.org/10.1017/pasa.2018.25). Documentations can be found [here](https://phantomsph.readthedocs.io/en/latest/). I work with my own [fork repository](https://github.com/Cheryl-Lau/phantom). Several new physics modules have been implemented for my research. Major ones include: 

**For the SPH-MCRT RHD scheme -**
* photoionize_cmi.F90
* kdtree_cmi.f90
* hnode_cmi.f90
* heating_cooling_cmi.f90
* utils_cmi.f90

**For cloud-scale supernovae injection -**
* inject_sne_sphng.f90
* cooling.F90

**For cluster potentials -** 
* extern_starcluster.f90


<br/>
### CMacIonize (fork) 

The grid-based Monte Carlo radiative transfer (MCRT) code adopted in our [RHD scheme](https://doi.org/10.1093/mnras/stab2178) is [CMacIonize](https://bwvdnbro.github.io/CMacIonize/about/), developed by [Bert Vandenbroucke and collaborators](https://doi.org/10.1016/j.ascom.2018.02.005). I work with my [fork repository](https://github.com/Cheryl-Lau/CMacIonize) though only the mapping routines have been modified. 

Instructions on how to couple Phantom to CMacIonize can be found [here](rhd_coupling.md). 


<br/>
### NbodyAcc 

[NbodyAcc](https://github.com/Cheryl-Lau/nbodyacc) is a recently-developed simple N-body code, dedicated for modelling massive binary accretion in clusters. Individual point masses can be treated as binaries; we track their accretion radii, internal angular momenta and separations. The code employs a 4th-order Runge-Kutta integrator, with King models for cluster potentials and incorporates turbulence. Code structure is similar to Phantom. 


<br/>
### SPHNG 

SPHNG is the precursor of Phantom. It was developed by Willy Benz, then passed to Ian Bonnell, then to Matthew Bate. 
I keep a copy of SPHNG from James Wurster and individual modules from Ian Bonnell. This code is yet to be adopted in my work but it was occationally used as a reference. 





