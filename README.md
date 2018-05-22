# GetContacts

Application for efficiently computing non-covalent contact networks in molecular structures and MD simulations. Following example computes all salt bridges, pi cation, aromatic, and hydrogen bond interactions in a trajectory:
```bash
get_dynamic_contacts.py --topology my_top.psf \
                        --trajectory my_traj.dcd \
                        --itypes sb pc ps ts hb \
                        --output my_contacts.tsv
```
The output, `my_contacts.tsv`, is a tab-separated file where each line (except the first two) records frame, type, and atoms involved in an interaction:
```
# total_frames:20000 interaction_types:sb,pc,ps,ts,hb
# Columns: frame, interaction_type, atom_1, atom_2[, atom_3[, atom_4]]
0   sb     C:GLU:21:OE2    C:ARG:86:NH2
0   ps     C:TYR:36:CG     C:TRP:108:CG
0   ts     A:TYR:36:CG     A:TRP:108:CG
0   hbss   A:GLN:53:NE2    A:GLN:69:OE1
1   hbss   A:GLU:60:OE2    A:SER:56:OG
1   hbbb   C:LEU:87:N      C:LEU:83:O
1   hbbb   C:ARG:88:N      C:LEU:87:N
2   hbsb   A:LYS:28:N      A:HIS:27:ND1
2   hbsb   A:ASP:52:OD2    A:PHE:48:O
2   wb2    C:ASN:110:O     B:ARG:73:NH1    W:TIP3:1524:OH2    W:TIP3:506:OH2
2   wb2    C:ASN:110:O     C:SER:111:OG    W:TIP3:1524:OH2    W:TIP3:2626:OH2
3   wb     A:ASP:100:OD1   A:ILE:67:O      W:TIP3:6762:OH2
3   wb     A:ASP:100:OD1   B:ASN:105:ND2   W:TIP3:9239:OH2
4   sb     A:GLU:47:OE2    A:LYS:33:NZ
4   pc     A:LYS:9:NZ      A:TYR:21:CG
4   hbbb   A:ILE:12:N      A:THR:20:O
...
```
Interactions that involve more than two atoms (i.e. water bridges and extended water bridges) have extra columns to denote the identities of the water molecules. For simplicity, all stacking and pi-cation interactions involving an aromatic ring will be denoted by the CG atom. 

Interaction types are denoted by the following abbreviations:
* **sb** - salt bridges 
* **pc** - pi-cation 
* **ps** - pi-stacking
* **ts** - T-stacking
* **vdw** - van der Waals
* Hydrogen bond subtypes
  * **hbbb** - Backbone-backbone hydrogen bonds
  * **hbsb** - Backbone-sidechain hydrogen bonds
  * **hbss** - Sidechain-sidechain hydrogen bonds
  * **wb** - Water-mediated hydrogen bond
  * **wb2** - Extended water-mediated hydrogen bond
* Ligand-hydrogen bond subtypes 
  * **hlb** - Ligand-backbone hydrogen bonds
  * **hls** - Ligand-sidechain hydrogen bonds
  * **lwb** - Ligand water-mediated hydrogen bond
  * **lwb2** - Ligand extended water-mediated hydrogen bond

Generated contact-list files are useful as inputs to visualization and analysis tools that operate on interaction-networks:
 * [Flareplot](https://gpcrviz.github.io/flareplot) - Framework for analyzing interaction networks based on circular diagrams
 * [MDCompare](MDCompare) - Heatmap fingerprints revealing groups of similar interactions in multiple MD trajectories
 * [TICC](https://github.com/davidhallac/TICC) - Changepoint detection algorithm to identifying significant events in the dynamic contact network
 * [NetworkAnalysis](Applications) - Compute residue contact frequencies in a simulation, analyze contact network graphs, visualize in PyMol

## Dependencies

GetContacts has the following dependencies
* [vmd-python](https://github.com/Eigenstate/vmd-python) 
  * netcdf >= 4.3
  * tk = 8.5

The easiest way to install netcdf is using a package manager. On a Mac, use the [homebrew package manager](https://brew.sh/) and run:
```bash
brew install netcdf
```

To install vmd-python on LINUX, use the [anaconda platform](https://www.anaconda.com/download):
```bash
conda install -c https://conda.anaconda.org/rbetz vmd-python
```

To install vmd-python on MAC (or LINUX systems without `conda`), you'll need to compile and install from source:
```bash
git clone https://github.com/Eigenstate/vmd-python
cd vmd-python
python setup.py build 
python setup.py install
cd ..
python -c "import vmd"  # Should not throw error
```

## Installation

To install GetContacts locally, first set up dependencies (see above) and then run:
```bash
git clone https://github.com/getcontacts/getcontacts

# Add folder to PATH
echo "export PATH=\$PATH:`pwd`/getcontacts" >> ~/.bashrc
source ~/.bashrc
```

To test the installation, run:
```bash
cd getcontacts/example
get_dynamic_contacts.py --topology 5xnd_topology.pdb \
                        --trajectory 5xnd_trajectory.dcd \
                        --itypes hb \
                        --output 5xnd_hbonds.tsv
```
and verify that no error was thrown and that the `5xnd_hbonds.tsv` file contains 1874 lines of interactions.

## Simulation and structure file format

GetContacts is compatible with all topology and reimaged trajectory file formats readable by [VMD](https://www-s.ks.uiuc.edu/Research/vmd/).

## Running the Code

### Command line arguments

Required Arguments:

    --topology STRING 
        Path to topology
    --trajectory STRING
        Path to simulation trajectory fragment
    --output STRING
        Path to output file
    --itypes STRING [STRING ...]
        Specifies interaction types (see above)
          all, sb, pc, ps, ts, vdw, hb, lhb

Optional Arguments:

    --cores INT 
        Number of CPU cores for parallelization [default = 6]
    --ligand STRING [STRING ...]
        Resname of ligand molecule(s) [default = ""]
    --sele STRING 
        VMD selection query to compute contacts in specified region of protein 
        [default = "protein"]
    --solv STRING 
        Solvent identifier in simulation [default = "TIP3"]

Arguments for adjusting geometric criteria:

    --sb_cutoff_dist FLOAT
        Cutoff for distance between anion and cation atoms [default = 4.0 Angstroms]
    --pc_cutoff_dist FLOAT
        Cutoff for distance between cation and centroid of aromatic ring [default = 6.0 Angstroms]
    --pc_cutoff_ang FLOAT
        Cutoff for angle between normal vector projecting from aromatic plane and vector from 
        aromatic center to cation atom [default = 60 degrees]
    --ps_cutoff_dist FLOAT
        Cutoff for distance between centroids of two aromatic rings [default = 7.0 Angstroms]
    --ps_cutoff_ang FLOAT
        Cutoff for angle between the normal vectors projecting from each aromatic plane 
        [default = 30 degrees]
    --ps_psi_ang FLOAT
        Cutoff for angle between normal vector projecting from aromatic plane 1 and vector between 
        the two aromatic centroids [default = 45 degrees]
    --ts_cutoff_dist FLOAT
        Cutoff for distance between centroids of two aromatic rings [default = 5.0 Angstroms]
    --ts_cutoff_ang FLOAT
        Cutoff for angle between the normal vectors projecting from each aromatic plane minus 90 
        degrees [default = 30 degrees]
    --ts_psi_ang FLOAT
        Cutoff for angle between normal vector projecting from aromatic plane 1 and vector between 
        the two aromatic centroids [default = 45 degrees]
    --hbond_cutoff_dist FLOAT
        Cutoff for distance between donor and acceptor atoms [default = 3.5 Angstroms]
    --hbond_cutoff_ang FLOAT
        Cutoff for angle between donor hydrogen acceptor [default = 70 degrees]
    --vdw_epsilon FLOAT
        Amount of padding for calculating vanderwaals contacts [default = 0.5 Angstroms]
    --vdw_res_diff INT
        Minimum residue distance for which to consider computing vdw interactions [default = 2]

   
### Examples

Compute all pi-cation, pi-stacking, and van der Waals contacts throughout an entire simulation:

    get_dynamic_contacts.py --topology TOP.psf \
                            --trajectory TRAJ.dcd \
                            --output output_pc_ps_vdw.tsv \
                            --itypes pc ps vdw

Compute salt bridges and hydrogen bonds for residues 100 to 160, including those that involves a ligand:

    get_dynamic_contacts.py --topology TOP.pdb \
                            --trajectory TRAJ.nc \
                            --output loop-ligand_sb_hb.tsv \
                            --cores 12 \
                            --solv IP3 \
                            --sele "chain A and resid 100 to 160" \
                            --ligand EJ4 \
                            --itypes sb hb lhb

Locate salt bridges and hydrogen bonds using a modified distance cutoffs:

    get_dynamic_contacts.py --topology TOP.mae \
                            --trajectory TRAJ.dcd \
                            --output output_sb_hb.tsv \
                            --sb_cutoff_dist 5.0 \
                            --hbond_cutoff_dist 4.5 \
                            --itypes sb hb

For further examples, see the [GetContacts main page](https://getcontacts.github.io).


## For developers

Unit tests are located in `unittests` subfolders and use the python `unittest` module. To run just the unit-tests, type

    python -m unittest

Integration tests verifying the functionality of executables are in the `tests`-folder. To run all tests make sure pytest is installed (e.g. `pip install pytest`) and run the following from the root:

    pytest
    
Pull-requests won't be accepted before all tests pass, but if there are any problems we are happy to help work them out.

The code aims to be [PEP 8](http://pep8.org/) compliant, but pull-requests wont be rejected if they're not. 
