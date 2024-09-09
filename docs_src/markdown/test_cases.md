# Test cases
## 1. Oxygen reduction reaction (ORR)
* The ORR is a key reaction in the cathode of H2-O2 fuel cells.
* The overall reaction for the ORR is:

  $$
    \begin{align*}
      & \ce{O2 + 4H+ + 4e-  -> 2H2O}  \ \text{(acidic)} \\
      & \ce{O2 + 2H2O + 4e- -> 4OH-} \ \text{(basic)}
    \end{align*}
  $$

* We will consider the acidic ORR in this test case.
* The acidic ORR consists of the following four elementary reacions;

  $$
    \begin{align*}
      & 1)\ \ce{O2(g) + surf + H+ + e- -> OOH-surf} \\
      & 2)\ \ce{OOH-surf + H+ + e- -> O-surf + H2O} \\
      & 3)\ \ce{O-surf + H+ + e- -> OH-surf} \\
      & 4)\ \ce{OH-surf + H+ + e- -> H2O + surf}
    \end{align*}
  $$

* Here we will evaluate the Gibbs free energy change ($\Delta G$)
  for each elemeantary reaction, and then take the maximum of these $\Delta G$s.
  This maximum $\Delta G$ is the overpotential ($\eta$) for the ORR.

  $$
    \eta = \max[\Delta G_1, \Delta G_2, \Delta G_3, \Delta G_4]
  $$

* $\eta$ is a key parameter to evaluate the catalytic activity, since it is
  the potential difference from the thermodynamically ideal potential (1.23 V).
* To peform the above procedure, we need to evaluate the $\Delta Gs$.
  for each elementary reactions. This is done by `calc_reaction_energy.`
* $\Delta Gs$ should be passed to `calc_overpotential_oer_orr` then $\eta$ is returned.

```python
import numpy as np
from microkinetics_toolkit.utils import make_surface_from_cif
from microkinetics_toolkit.utils import remove_layers
from microkinetics_toolkit.utils import replace_element
from microkinetics_toolkit.utils import fix_lower_surface
from microkinetics_toolkit.calc_reaction_energy import calc_reaction_energy
from microkinetics_toolkit.orr_and_oer import calc_overpotential_oer_orr 

cif_file = "LaMnO3.cif"
surface = make_surface_from_cif(cif_file, indices=(0, 0, 1), vacuum=10.0)

# for EMT
use_emt = False
if use_emt:
    surface = replace_element(surface, from_element="La", to_element="Al")
    surface = replace_element(surface, from_element="Mn", to_element="Pt")
    surface = remove_layers(surface, element="Al", n_layers=1)
else:
    surface = remove_layers(surface, element="La", n_layers=1)

surface = remove_layers(surface, element="O", n_layers=2)
surface = fix_lower_surface(surface)

# reaction_file = "orr_alkaline.txt"
reaction_file = "orr_alkaline2.txt"

deltaEs = calc_reaction_energy(reaction_file=reaction_file, surface=surface, calculator="vasp", verbose=True)
eta = calc_overpotential_oer_orr(reaction_file=reaction_file, deltaEs=deltaEs, reaction_type="orr", verbose=True)

print(f"overpotential = {eta:5.3f} eV")

