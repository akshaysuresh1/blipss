# BLIPSS
The Breakthrough Listen Investigation for Periodic Spectral Signals (BLIPSS) targets the detection of narrowband periodic radar transmissions from potential technologically advanced alien life forms residing in the Universe. See this [link](http://www.mobileradar.org/radar_descptn_3.html) for examples of historic terrestrial radar operating at different radio frequencies.

BLIPSS utilizes the Fast Folding Algorithm (FFA) in [`riptide-ffa`](https://github.com/v-morello/riptide) to search for channel-wide periodic signals in radio dynamic spectra.

---

## Table of Contents
- [Package dependencies](#dependencies)
- [Installation](#installation)
- [Repository Organization](#organization)
- [Functionalities and Usage](#usage)
    - [blipss.py](#blipss_exec)
    - [compare_cands.py](#comparecands)
    - [plot_cands.py](#plotcands)
    - [phaseresolved_ds.py](#phaseds)
    - [inject_signal.py](#injectsignal)
    - [simulate_data.py](#simulatedata)
- [Troubleshooting](#troubleshooting)

## Package Dependencies <a name="dependencies"></a>
```blipss``` is written in Python 3.8.5, and has the following package dependencies.
- astropy >= 4.0
- [`blimpy`](https://github.com/UCBerkeleySETI/blimpy) >= 2.0.0
- matplotlib >= 3.1.0
- mpi4py >= 3.1.1
- numpy >= 1.18.1
- pandas >= 1.3.4
- [`riptide-ffa`](https://github.com/v-morello/riptide) >= 0.2.4
- scipy >= 1.6.0
- tqdm >= 4.32.1

## Installation <a name="installation"></a>
1. Clone this repository to your local machine. To do so, execute the following at the command line.
```
git clone git@github.com:UCBerkeleySETI/blipss.git
```
2. Verify that your local Python 3 installation satisfies all dependencies of ```blipss```. If not, either manually install the missing dependencies or run the below calls.
```
cd blipss
pip install pybind11
python setup.py install
python setup.py clean
```
Note: `pybind11` is a prerequisite for installing [`riptide-ffa`](https://github.com/v-morello/riptide).

## Repository Organization <a name="organization"></a>
The repository is organized as two major folders, which are: <br>
1. `config`: A folder containing sample input configuration scripts for various use cases <br>
2. `executables`: Primary executable files for different tasks. Unless absolutely required, avoid editing executable scripts to ensure smooth operation. <br> <br>

For every functionality (say `inject_signal.py`), you will find relevant configuration scripts and executable files under the `config` and `executables` folders respectively. <br> <br>

To run an executable file, use the `-i` flag to supply its companion configuration script in the command line. For example, if you are running a command line terminal from the ``blipss`` repository, initiate an instance of ``inject_signal.py`` using:
```
python executables/inject_signal.py -i config/inject_signal.cfg
```
Comments at the top of every executable file provide program execution syntax.

## Functionalities and Usage <a name="usage"></a>
The BLIPSS package currently contains six chief executable files, which are:
1. ``blipss.py`` <a name="blipss_exec"></a> <br>
Executes channel-wise FFA on input data files (filterbank or hdf5), identifies harmonics of detected periods, and outputs a .csv file of candidates. Here is a schematic of the `blipss.py` workflow. <br>

![BLIPSS workflow (Jan 27, 2022)](https://github.com/akshaysuresh1/blipss/blob/main/images/blipss_design_2022Jan27.png?raw=True)

Columns in the .csv file output by ``blipss.py`` include 'Channel', 'Radio frequency (MHz)', 'Bins', 'Best width', 'Period (s)', 'S/N', and 'Harmonic flag'. <br>

The current implementation takes about 35 min. to run on a single mid-resolution filterbank product (1.07 s sampling, 2.86 kHz, 1703936 channels). For processing multiple input files in parallel, enable MPI via the following syntax.
```
mpiexec -n <nproc> python -m mpi4py executables/blipss.py -i config/blipss.cfg | tee <Log file>
```
The above syntax assumes a Python call from the repo base directory. Alter paths as required to supply executable and config scripts located in different directories.

---
2. ``compare_cands.py``: <a name="comparecands"></a>
Compare periodicity detections across a set of <em>N</em> .csv files generated by ``blipss.py``. For every unique candidate period, an <em>N</em>-digit binary code is generated, wherein ones and zeros represent detections and non-detections respectively.<br>

Note that the order of input .csv files passed to ``compare_cands.py`` matters. When read from left to right, the <em>i</em>-th place of the <em>N</em>-digit binary code refers to the <em>i</em>-th .csv file in the input list.<br>

The output from ``compare_cands.py`` is a single .csv file containing the following columns.<br>
'Channel', 'Radio frequency (MHz)', 'Bins', 'Best width', 'Period (s)', 'S/N', 'Code' <br>

Execution syntax from repo base folder:
```
python executables/compare_cands.py -i config/compare_cands.cfg | tee <Log file>
```

---
3. ``plot_cands.py``: <a name="plotcands"></a>
Produce verification plots for a chosen subset of candidates. <br>

Here's a sample plot of a candidate with period 21.5105 s and code 001010. Each row represents a different data file. The left column shows periodograms derived from different data files. We indicate the candidate period by red dashed vertical lines in the left panels. The right column illustrates average pulse profiles and pulse stacks in the phase-time plane. <br>

![B04 candidate](https://github.com/akshaysuresh1/blipss/blob/main/images/sim_cand.png?raw=True)

Clearly, we see significant spikes at the expected candidate period in the periodograms on the 3rd and 5th rows. <br>

Execution syntax from repo base folder:
```
python executables/plot_cands.py -i config/plot_cands.cfg | tee <Log file>
```

---
4. ``phaseresolved_ds.py``: <a name="phaseds"></a>
Compute and plot the phase-resolved spectrum for a given folding period.

![B04 spectrum](https://github.com/akshaysuresh1/blipss/blob/main/images/guppi_58737_81548_PSR_B2021+51_0023_period0.52920.png?raw=True)

Execution syntax from repo base folder:
```
python executables/phaseresolved_ds.py -i config/phaseresolved_ds.cfg | tee <Log file>
```

---
5. ``inject_signal.py``: <a name="injectsignal"></a>
Inject one or more channel-wide periodic signals into a real-world data set. Fake periodic signals are assumed to have a boxcar single pulse shape with uniform pulse amplitude distribution.<br>

Execution syntax from repo base folder:
```
python executables/inject_signal.py -i config/inject_signal.cfg | tee <Log file>
```

---
6. ``simulate_data.py``: <a name="simulatedata"></a>
Build an artificial data set with one or more channel-wide periodic signals superposed on normally distributed, white noise background. Again, the injected fake periodic signals have boxcar single pulse shapes and uniform pulse amplitude distributions.

Execution syntax from repo base folder:
```
python executables/simulate_data.py -i config/simulate_data.cfg | tee <Log file>
```

## Troubleshooting <a name="troubleshooting"></a>
Please submit an issue to voice any problems or requests.

Improvements to the code are always welcome.
