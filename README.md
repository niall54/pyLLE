# pyLLE ![NIST logo](images/NISTlogo32x32.jpg)

[![](https://img.shields.io/static/v1.svg?label=docs&message=passing&color=success)](https://gregmoille.github.io/pyLLE/)
[![](https://img.shields.io/static/v1.svg?label=version&message=2.1.1&color=bue?style=flat)]()


pyLLE is a tool to solve the Lugiato Lefever Equations (LLE)<sup>[1](#ref1)</sup><sup>,</sup><sup>[2](#ref2)</sup><sup>,</sup><sup>[3](#ref3)</sup>in a fast and easy way. Thanks to a user-friendly front-end in python and an efficient back-end in Julia, solving this problem becomes easy and fast. 

For a complete documentation of the package, please visit the [github page](https://gregmoille.github.io/pyLLE/)

## Instalation

As pyLLE relies on a Julia back-end, please prior to installing this package be sure that Julia is installed on your machine or visit the julia [package download page](https://julialang.org/downloads/oldreleases.html) to install it by selecting &#9888; **v0.6.4** &#9888;. Due to issues handling the hdf5 file format, version 1.0.1 is not yet supported.

**Windows users**: Please, keep julia in the default directory during the installation (i.e. ~\AppData\Local\Julia-0.6.4\ for windows). 

If not, please go to the manual installation.
Once Julia installed, the different packages needed to run pyLLE, either python or julia related, will be automatically downloaded and installed. Just a heads up, the installation of the package can vary in time, especially because of Julia that might rebuild the cache.
For a automatic install, just pip it : 

```bash
pip install pyLLE
```

For a manual install, download the .zip of the repository or clone it and install with the setup.py script 

```bash
git clone https://github.com/gregmoille/pyLLE.git
cd pyLLE
python setup.py install
```

If the julia location is custom, please before installing change in the setup.py, line 18 to the correct location, as in pyLLE/llesolver.py line 430 to point to the correct location. Thanks

## Example

A complete example is available in the example directory [notebook](https://github.com/gregmoille/pyLLE/tree/master/example/NotebookExample.ipynb) with the corresponding file needed in the folder. Here we will only go through the main aspect of pyLLE

- First import the package:
```python
import pyLLE
```

- Define a resonator and a simulation dictionary such as (parameters from Li et _al._<sup>[4](#ref4)</sup>): 
```python
res = {'R': 23e-6, # ring radius
       'Qi': 1e6,  # intrinsic quality factor
       'Qc': 1e6,  # coupling quality factor
       'γ': 2, # non-linear coefficient
       'dispfile': 'TestDispersion.txt' # name of the dispersion file
       }
sim = {'Pin': 100e-3, #input power in W
       'Tscan': 1e6, # Total time for the simulation in unit of round trip
       'f_pmp': 191e12, # frequency of the pump in Hz
       'δω_init': -4, # start frequency of detuning ramp in Hz
       'δω_end': 10, # stop frequency of detuning ramp in Hz
       'μ_sim': [-70,170], # limit of the mode on the left and right side of the pump to simulate
       'μ_fit': [-60, 160], # limit of the mode on the left and right side of the pump to fit the dispersion with
        }
```

It is important to note the format of the dispersion file *TestDispersion.txt*. It must be formatted such that each line represents a resonance, with first the azimuthal mode order listed and then the frequency of the resonance, separated by a comma ',' 

- The simulation needs to be set up to create a .hdf5 

```python
   solver.Setup()
```

<br>

We can now set up the pyLLE class: 

```python 
solver = pyLLE.LLEsolver(sim=sim,
                       res=res,
                       debug=False)
```

The debug input allows the script to generate a log file in the working directory with useful information on the simulation. The authors highly encourage to keep this key to True, unless some loops are run, which could create an issue with the read/write access to the file. 

<br>

To analyze the dispersion just the *Analyze*

```python
solver.Analyze(plot=True,
               plottype='all')
```

To start the simulation, first we need to set up an hdf5 file which makes the bridge between python and julia 

```python
solver.Setup()
```

<br>

For the sake of simplicity and to retrieve the different parameters, all keys of the sim and res dictionary are translated with the previously described translator dictionary and cannot be accessed anymore with the greek alphabet. Hence, in an ipython console, one can retrieve the parameter with, for example: 

```python
IN [1]: solver.sim['mu_sim']
OUT [1]: [-70,170]
```

<br>

Then we can start the simulation 

```python 
solver.Solve()
```

To retrieve the data computed by julia, we call the *RetrieveData* method

```python
solver.RetrieveData()
```

We can finally start to plot the result of the simulation. One can start with a complete overview of the simulation, where a spectral and a temporal map vs the LLE step is displayed in addition to the comb power Vs the LLE step

```python
solver.PlotCombPower()
```

From there, we can find the step of the LLE where we want to see the spectra:

```python
ind = 600
solver.PlotCombSpectra(ind)
```


One can also quickly solve the LLE through a steady-state method to find its roots

```python
sim['δω'] =  -10e9,
Ering, Ewg, f, ax = solver.SolveSteadySteate()
```

## How to Cite Us?

You can cite our arXiv paper available [here](https://arxiv.org/abs/1903.10441): 

> G. Moille, Q. Li, X. Lu and K. Srinivasan, _"pyLLE: a Fast and User Friendly Lugiato-Lefever Equation Solver"_, arXiv, 1903.10441 , 2019


## References

<a name="ref1">1</a>: Luigi A. Lugiato and René Lefever. "Spatial dissipative structures in passive optical systems." Physical review letters 58, no. 21 (1987): 2209.

<a name="ref1">2</a>: Yanne K. Chembo and Curtis R. Menyuk. "Spatiotemporal Lugiato-Lefever formalism for Kerr-comb generation in whispering-gallery-mode resonators." Physical Review A 87, no. 5 (2013): 053852.

<a name="ref1">3</a>: Stéphane Coen, Hamish G. Randle, Thibaut Sylvestre, and Miro Erkintalo. "Modeling of octave-spanning Kerr frequency combs using a generalized mean-field Lugiato–Lefever model." Optics letters 38, no. 1 (2013): 37-39.

<a name="ref1">4</a>: Qing Li, Travis C. Briles, Daron A. Westly, Tara E. Drake, Jordan R. Stone, B. Robert Ilic, Scott A. Diddams, Scott B. Papp, and Kartik Srinivasan. "Stably accessing octave-spanning microresonator frequency combs in the soliton regime." Optica 4, no. 2 (2017): 193-203.
