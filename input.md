# Preparing the input files
The hySAS ONEBEAD mapping (1 bead for 1 amino acid, 3 beads for 1 nucleodide) requires two PDBs, one for the MOLINFO and one for the TEMPLATE actions. This applies to both SAXS and SAS modules. The MOLINFO PDB may contain atoms, residues, chains, that are not considered for the SAS signal calculation, while the TEMPLATE PDB must contain only the residues that are used for the ONEBEAD conversion. Some general rules:

#### 1. Consistency between MOLINFO and TEMPLATE PDBs
The numbering of atoms and residues must be consistent between the MOLINFO and TEMPLATE PDBs. For example, if MOLINFO is provided with a PDB containing 100 residues, but the SAS calculation is performed on residues in the range 20-80, it is necessary to provide TEMPLATE with a PDB containing only these residues, while maintaining their numbering: if the 20th residue starts with atom number 319 and the 80th residue ends with atom number 1325, the TEMPLATE PDB must start with ATOM 319 and residue number 20, and end with ATOM 1325 and residue number 80. 
#### 2. Consistency between MOLINFO and the molfile
When using the plumed driver, the supplied molfile (PDB, XTC, ...) to be analysed must be consistent with the PDB provided in MOLINFO. Specifically, there must be the same number of atoms in the same order.
#### 3. MOLINFO PDB numbering
The MOLINFO PDB must start with ATOM number 1. The residue number can start with a number other than 1.
#### 4. Providing the same file to MOLINFO and TEMPLATE
It is possible to provide MOLINFO and TEMPLATE with the same PDB, taking into account all three points above.
#### 5. PDB format
The naming conventions for atoms and residues can vary depending on the forcefield or software used. This variability, particularly in the case of nucleic acids, is a major limitation in the hySAS mapping process. In addition to calculating the Solvent-Accessible Surface Area, which is necessary to introduce the solvent layer correction or the hydrogen-deuterium exchange, it is essential to detect the residue and analyse the atoms belonging to a bead in order to define its centre of mass. This is used to locate the centre of the bead and this position has a major impact on the final signal calculation. For this reason, as a preliminary step, each bead type and atom composition is verified. In order to perform this process, it is not feasible to consider all possible atom and residue names. For proteins, all existing names for histidine residues in the different protonation states are accepted: HIS, HIE/HID/HIP (AMBER), HSE/HSD/HSP (CHARMM). For nucleic acids, only the AMBER OL3 nomenclature for RNA and the AMBER OL15 nomenclature for DNA are accepted. The easiest way to convert a PDB to these formats is to use the pdb4amber script which is part of the free [AmberTools](https://ambermd.org/AmberTools.php) suite. 

Hydrogen must be removed, otherwise, doubled!

```
pdb4amber -i original_DNA.pdb -o DNA_amber.pdb -d -v --add-missing-atoms
```
Or LEaP. Here is a quick step-by-step guide:
* ##### RNA
create a `leapRNA.in` file:
```
vim leapRNA.in
```
copy the following instructions and save the file:
```
source leaprc.RNA.OL3

rna_molecule = loadpdb original_RNA.pdb

savepdb rna_molecule RNA_amber.pdb

quit
```
Command details:

`source leaprc.RNA.OL3`: Load the parameter set for RNA molecules, specifically the OL3 force field parameters. 
`rna_molecule = loadpdb original_RNA.pdb`: Load the RNA molecule from the specified PDB file (original_RNA.pdb) into the LEaP program and assign it to the variable rna_molecule. Replace original_RNA.pdb with the PDB name you wish to convert.
`savepdb rna_molecule RNA_amber.pdb`: Save the RNA molecule that was loaded into the LEaP environment into a new PDB file named RNA_amber.pdb. This new PDB file will be formatted according to the conventions used by the AMBER software suite. Replace "RNA_amber.pdb` with your preferred PDB name.

Run LEaP:
```
tleap -s -f leapDNA.in
```
* ##### DNA
create a `leapDNA.in` file:
```
vim leapDNA.in
```
copy the following instructions and save the file:
```
source leaprc.DNA.OL15

dna_molecule = loadpdb original_DNA.pdb

savepdb dna_molecule DNA_amber.pdb

quit
```
Run LEaP:
```
tleap -s -f leapDNA.in
```
#### 6. Special beads
##### DNA and RNA
Two additional bead types are available for DNA and RNA besides phosphate group, pentose sugar, and nucleobase:
- 5'-end pentose sugar capped with a hydroxyl moiety at C5'. To enable this, the residue name in the PDB must be followed by "5". For example, in the case of cytosine, the corresponding residue must be edited to DC5 or to C5 in DNA and RNA, respectively.
- 3'-end pentose sugar capped with an hydroxyl moiety at C3'. To enable this, the residue name in the PDB must be followed by "3". For example, in the case of cytosine, the corresponding residue must be edited to DC3 or to C3 in DNA and RNA, respectively.

##### Proteins
An additional bead type is available for proteins:
- Cysteine residue involved in disulfide bridge (the residue in the PDB must be named CYX).

#### 7. Atom names



pay attention to not cut residues in selecting atoms