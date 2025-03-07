<div align="right">
<a href="https://github.com/PalaeoCarb/MyAMI/actions/workflows/test-myami.yml"><img src="https://github.com/PalaeoCarb/MyAMI/workflows/Check%20MyAMI%20Performance/badge.svg" height=18></a>
<a href="https://pypi.org/project/pymyami"><img src="https://badge.fury.io/py/pymyami.svg" height=18></a>
</div>

# MyAMI
The MyAMI Specific Ion Interaction Model for correcting stoichiometric equilibrium constants (*Ks*) for variations in seawater composition, made available available as the `pymyami` python package.

This package is a re-factor of the MyAMI model published by [Hain et al. (2015)](https://doi.org/10.1002/2014GB004986), which is available [here](https://github.com/MathisHain/MyAMI). The key differences between the original model and this package are:
- **Speed**: All calculations have been vectorised using NumPy, making MyAMI 2-3 orders of magnitude faster.
- **Direct Calculation**: `pymyami` directly calculates correction factors using the MyAMI model. This differs from [Hain et al. (2015)](https://doi.org/10.1002/2014GB004986), where the focus was on modifying parameters that could be input into the standard equations for calculating stoichiometric equilibrium products.
- **Correction Factor Focus**: `pymyami` produces *corrections factors* (F<sub>X,MyAMI</sub>) that can be applied to adjust stoichiometric equilibrium constants for variations in seawater composition, following K<sub>X,corr</sub> = K<sub>X,empirical</sub> * F<sub>X,MyAMI</sub>. For the direct calculation of Ks, including the corrections calculated by `pymyami`, please see the [Kgen](https://github.com/PalaeoCarb/Kgen) project.
- **Available Ions**: `pymyami` allows the modification of any ion in the model, rather than just Mg and Ca: Na<sup>+</sup>, K<sup>+</sup>, Mg<sup>2+</sup>, Ca<sup>2+</sup>, Sr<sup>2+</sup>, Cl<sup>-</sup>, B(OH)<sub>4</sub><sup>-</sup>, HCO<sub>3</sub><sup>-</sup>, CO<sub>3</sub><sup>2-</sup> and SO<sub>4</sub><sup>2-</sup>.
- **Parameter Transparrency**: Wherever possible, parameter tables are now constructed on-the-fly from raw tables in the Appendix of [Millero & Pierrot, 1998](https://doi.org/10.1023/A:1009656023546), making the origin of parameters explicit.
- **Pure Python**: There is no longer interface code for interacting with other languages (i.e. MATLAB). This caused a substantial performance bottleneck, and is discouraged. The [Kgen](https://github.com/PalaeoCarb/Kgen) project provides a convenient interface to use `pymyami` in R and MATLAB.
- **Approximation Method**: Where very fast calculations are required (e.g. Monte Carlo methods), `pymyami` uses a high-dimensional polynomial to approximate F<sub>X,MyAMI</sub> as a function of temperature, salinity, Mg and Ca. This is a very fast approximation, but is only accurate to within ~0.25%.

## Kgen
`pymyami` only calculations *correction factors* that can be applied to stoichiometric equilibrium constants (Ks). If you are looking for a convenient way to adjust Ks for variations in seawater composition, please see the [Kgen](https://github.com/PalaeoCarb/Kgen) project.

## Consistency with Hain et al. (2015)
The K correction factors calculated by `pymyami` are similar to those calculated by the code of [Hain et al. (2015)](https://doi.org/10.1002/2014GB004986), although there are some notable deviations of up to 4%. A summary of maximum and average differences compared to Hain et al. (2015) follows:
```
  K0: 0.00% max, 0.00% avg
  K1: 0.92% max, 0.05% avg
  K2: 3.77% max, -0.07% avg
  KW: 2.35% max, -0.42% avg
  KB: 0.92% max, 0.05% avg
  KspA: 1.87% max, 0.04% avg
  KspC: 1.87% max, 0.04% avg
  KS: 1.83% max, 0.10% avg
```
Note that maximum deviations are seen when the change in Mg and Ca correlates, meaning that these deviations shouldn't be too important for palaeo-seawater calculations because the concentration of Mg and Ca tend to be anti-correlated through geological history.

These differences arise from typo corrections in the original code, and pymyami should be closer to the original MIAMI model of Millerot and Pierrot (1998).

## Installation

The model is available as a PyPI package, which can be installed by:

```python
pip install pymyami
```

## Example Usage
```python
from pymyami import calc_Fcorr, approximate_Fcorr

# run the model to calculate correction factors
calc_Fcorr(TempC=35, Sal=36.2, Mg=0.03, Ca=0.012)

>>> {'KspC': 0.7843309390178521,
     'KspA': 0.7843309390178521,
     'K1': 1.002405617170862,
     'K2': 0.7885093392132683,
     'KW': 0.7459757009835559,
     'KB': 0.9382494946753764,
     'K0': 1.0056418412233974,
     'KS': 0.9573891319238595}

# use the polynomial approximation to calculate correction factors
approximate_Fcorr(TempC=35, Sal=36.2, Mg=0.03, Ca=0.012)

>>> UserWarning: WARNING: using approximate MyAMI K correction factors instead of calculated ones. These are only accurate to within ~0.25%. Please dont use them for anything critical.

>>> {'K0': array(1.00565919),
     'K1': array(1.00238861),
     'K2': array(0.78858314),
     'KB': array(0.93815884),
     'KW': array(0.74594823),
     'KspC': array(0.78442705),
     'KspA': array(0.78442705),
     'KS': array(0.95738293)}
```
