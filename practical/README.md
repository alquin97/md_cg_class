# Protocol

Here we will be working with an experimentally determined dimeric structure of Apelin Receptor (**7W0N**), another GPCR.

[![](https://github.com/alquin97/md_cg_class/blob/main/practical/files/images/7w0n_pdb.png)](#)

## Building the system with CHARMM-GUI

CHARMM-GUI is a web-based platform to interactively build molecular biosystems ready to run in many different MD simulation engines (such as GROMACS). The building process is very streamlined and intuitive for new users but lacks some customization options to model very specific cases. Navigate to the `Input Generator > Martini Maker > Bilayer Builder` to start building a coarse-grained protein-membrane system. Select the latest Martini FF model (martini3.0.0). Here you can either fetch the PDB structure from RCSB or OPM but, for this specific case, it is better to download the preoriented PDB file from [OPM](https://opm.phar.umich.edu) and upload it from you computer.

[![](https://github.com/alquin97/md_cg_class/blob/main/practical/files/images/martini_ff.png)](#)

### Step 0: PDB manipulation

Select the chains/segments from the uploaded PDB file to be modelled. The uploaded PDB file contains not only the apelin receptor structures but also the co-crystallized endogenous apelin 18-32, the trimeric Gi protein and the scFv16 construct. Since we are only interested in the apelin receptors, deselect all the other objects.

[![](https://github.com/alquin97/md_cg_class/blob/main/practical/files/images/segment_ids.png)](#)

Then set `ph=7.0` and choose the N- and C-termini to be neutral (not charged). We are not gonna model any of the missing residues.

[![](https://github.com/alquin97/md_cg_class/blob/main/practical/files/images/neutral.png)](#)

### Step 1: Position and orientation options

Here we only want to select the PDB native orientation since we got the PDB from the preoriented OPM database.

[![](https://github.com/alquin97/md_cg_class/blob/main/practical/files/images/orient.png)](#)

### Step 2: Cell size and lipid packing

CHARMM-GUI encourages us to view the oriented PDB structure before proceding any further. Please do (you can do from the website viewer) and check that the selected chains and orientation of the two monomers is correct.

Once done, establish the cell size. We will do by setting a minimum `Water thickness 15` in the Z axis and an initial guess of `Length of X and Y: 120` for the XY plane. Next, choose the phospholipid composition. For the purpose of this tutorial, we will use the simplest membrane of POPC-only membrane. Finally, click on `Show the system info` to get the calculated size of the membrane and unit cell and continue the process.

[![](https://github.com/alquin97/md_cg_class/blob/main/practical/files/images/cell_membrane_size.png)](#)

### Step 3: Bilayer construction and solvation options

Here we can leave the default parameters as they are. Notice that we can adjust the lipid and ions placement method as well as the salt composition and concentration.

[![](https://github.com/alquin97/md_cg_class/blob/main/practical/files/images/solvate.png)](#)

### Step 4: Solvating

Check that there are no lipid penetration issues. If not, proceed with the building of the ions and water box.

[![](https://github.com/alquin97/md_cg_class/blob/main/practical/files/images/lipid_penetration.png)](#)

### Step 5: Assemble components and generate input options.

All components are built, now they will be combined in a single PDB file. Specify the conditions in which to run the simulation so CHARMM-GUI can write the proper molecular dynamics parameters (.mdp) files.

[![](https://github.com/alquin97/md_cg_class/blob/main/practical/files/images/assemble.png)](#)

Finally, download the compressed .tgz file with all the generated inputs.

## GROMACS simulation

Copy and extract the files downloaded from CHARMM-GUI to your working directory.

```
tar -xvf charmm-gui.tar
```

CHARMM-GUI does the whole preparation process for us and provides us with all the intermediate files used from steps 1-5. It provides as well standard molecular dynamics parameters (.mdp) files to get the simulations started right out of the box. For the purposes of this tutorial we won't tweak any of the parameters set by default but keep in mind that you can freely modify the .mdp files according to your needs.

Navigate to the `charmm-gui-XXXXXXXXXX/gromacs/` directory (where XXXXXXXXXX is your unique session ID number from CHARMM-GUI). Here you have the coordinates (step5_charmm2gmx.pdb) and topology file (system.top) required to produce the portable binary run (.tpr) for any GROMACS job, as well as the .mdp instructions for each minimization, equilibration and production step. Each consecutive step is executed from the `README` script found in the same folder. Examine the script and try to figure out what commands will be executed.

> Question: How many minimization, equilibration and production steps are executed?

Once you are ready, execute the README shell script.

```
chmod +x README
csh README
```

Pay attention to the output printed on the screen. It is likely that the process didn't run successfully.

> Question: Can you spot the ERROR? In which step did the procedure failed?

There are different ways in which the specific error that we get can be addressed. However, for the sake of simplicity, we will do a quick and dirty fix that consists on editing the `README` file with the `sed` shell command.

```
sed -i '' -e 's/        gmx grompp -f step6.${cnt}_equilibration.mdp -o step6.${cnt}_equilibration.tpr -c step6.${pcnt}_minimization.gro -r step5_charmm2gmx.pdb -p system.top -n index.ndx/        gmx grompp -f step6.${cnt}_equilibration.mdp -o step6.${cnt}_equilibration.tpr -c step6.${pcnt}_minimization.gro -r step5_charmm2gmx.pdb -p system.top -n index.ndx -maxwarn 1/g' README
sed -i '' -e 's/        gmx grompp -f step6.${cnt}_equilibration.mdp -o step6.${cnt}_equilibration.tpr -c step6.${pcnt}_equilibration.gro -r step5_charmm2gmx.pdb -p system.top -n index.ndx/        gmx grompp -f step6.${cnt}_equilibration.mdp -o step6.${cnt}_equilibration.tpr -c step6.${pcnt}_equilibration.gro -r step5_charmm2gmx.pdb -p system.top -n index.ndx -maxwarn 1/g' README
sed -i '' -e 's/gmx grompp -f step7_production.mdp -o step7_production.tpr -c step6.6_equilibration.gro -p system.top -n index.ndx/gmx grompp -f step7_production.mdp -o step7_production.tpr -c step6.6_equilibration.gro -p system.top -n index.ndx -maxwarn 1/g' README

```

Run `README` again. 

> Note: If you encounter a *'There is no domain decomposition for n ranks that is compatible with the given box and a minimum cell size of x nm'* ERROR, it means that GROMACS can't parallelize the calculations (not enough processing units for the specified input). This can be fixed by adding a `-ntmpi 1` flag in the `mdrun` command line in `README` with a text editor. Consider that there are multiple `mdrun` lines in the script. Add the option where necessary.

This time, the number of 'atoms' to simulate is much lower than in the ```md_membrane_class``` (as we are running a coarse-grained system of similar size) so it should not take as much time to complete. However, without GPU support, it still takes too long to finish in time for the class. The same as we did for the previous class, we will use some precomputed sample trajectories to run the analysis.

## Analysis

Get the sample trajectory provided in the shared folder. Visualize the trajectory in VMD or any other visualization software. 

For this project we will carry out a **contacts analysis** between the two monomers. By 'contacts' we refer to 'atomic' interactions ocurring during the course of the MD simulation. Protein activity and mechanism of action at the molecular level are fundamentally governed by networks of atom-atom interactions, so it is extremely valuable to know which of these are the most significant and to understand how they change for different states of the system (e.g. activation).

### Number of contacts

### Contact frequency
