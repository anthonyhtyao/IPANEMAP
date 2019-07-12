# IPANEMAP
### Integrative Probing Analysis of Nucleic Acids Empowered by Multiple Accessibility Profiles.

IPANEMAP is a software for predicting stable RNA secondary structures compatible from multiple chemical probing (SHAPE, DMS...)  reactivities profiles. From one or several input sequences, along with several reactivity profiles, it computes and outputs one or several secondary structures, corresponding to the conformers best supported by experimental data and thermodynamics.

## Installing IPANEMAP

IPANEMAP consists in a set of Python 2.7+ scripts, and requires the prior installation, and accessibility from the command line, of the following dependencies:
1. `ViennaRNA` package, version posterior to 2.0, [downloadable from the TBI](https://www.tbi.univie.ac.at/RNA/#download "Download the Vienna package")
2. `scikit-learn`, [version 0.2 for Python 2.7+](https://scikit-learn.org/stable/install.html "Download scikit-learn")

## Executing IPANEMAP

Once all dependencies are satisfied, IPANEMAP can be invoked through: 

      python IPANEMAP.py [--RNA rnafile.fa] [--cond c1 c2 ...]

The method will run with a configuration specified within `IPANEMAP.cfg`, optionnally overriding the RNA using the `--RNA` command-line option, and the  list of conditions with the `--cond` option.

## Input files

### Reactivity/soft constraints file format
IPANEMAP expects to find reactivities in  a file `{SoftConstraintsDir}/{RNA}{Cond}.txt`, where the meaning of the various variables is made clear below. The content of a reactivity file is simply a list of position/value pairs providing a reactivity for each position. 
Values are expected to be loosely normalized as fall in the [0,1] interval, with negative numbers indicating missing values.


Example:

      1	0.568309
      2	0.179692
      3	-999
      4	0.568389
           ....


### Hard constraints file format
Hard constraints allow to force predictions to be consistent with prior partial knowledge. They should be expressed in a file `{HardConstraintsDir}/{RNA}{Cond}.txt` in classic FASTA/DBN format (see example below), consisting of sequence/constraint mask in extended dot-bracket notation supported by the [Vienna package syntax](https://www.tbi.univie.ac.at/RNA/RNAfold.1.html).

Example: The following file content

      > Some RNA
      CCCAAAUGGG
      (x(....)x)
     
indicates that two base pairs, corresponding to matching parentheses `(` and `)`, should always be respected by the models. 
Positions associated with `x` will be forced to remain unpaired, but positions associated with a dot `.` are not constrained in the folding.
More complex constraints are available, as described in the [Vienna package documentation](https://www.tbi.univie.ac.at/RNA/RNAfold.1.html).

## Output

IPANEMAP typically produces many messages during execution, to keep the user informed of its progress.
However, only the final (Pareto) structural models are output to the standard output, meaning that, after running

      python IPANEMAP.py > finaloutput.fa

the `finaloutput.fa` file will only consist of the final models.

Example: Input:GGGAAACCCAAAGGGAAACCC



## Configuration
Most configuration options are set by modifying the content of the `IPANEMAP.cfg` file.

### Main options
 - `RNA`: Specifies a path (relative to the working directory) to a FASTA file where the nucleotide sequence of the main RNA of interest can be found. Note that the filename is important, as it will be used as a base name for the other input files. Example: `RNA: fasta_files/didymium.fa` will process the sequence found in the file, and `didymium` will be used as the *base name* of reactivities/hard contraints files (see `Conditions` option)
 - `SoftConstraintsDir` and `HardConstraintsDir`: Sets the *directories* used by IPANEMAP to locate soft (reactivities) and hard constraints files (if available)
 - `Conditions`: Can be used to specify the list of probing conditions used for the prediction. Should be set to a comma-separated list of conditions, i.e. the names of reactivity profiles/experiments to be considered for structure prediction
 
For an RNA having base name `{RNA}`, and a condition name `{Cond}`, IPANEMAP will attempt to locate files named `{SoftConstraintsDir}/{RNA}{Cond}.txt` and `{HardConstraintsDir}/{RNA}{Cond}.txt`. If none of these files is found, the method will rely on a purely thermodynamic sampling.

Example: Given a configuration
 
      [Input] 
      RNA: fasta_files/5sRNA.fa
      SoftConstraintsDir: soft
      HardConstraintsDir: hard
      Conditions: DMSMG,NMIA
      ...
   
the method will attempt to locate, and use for the sampling phase of the method, two files `5sRNADMSMG.txt` and `5sRNANMIA.txt` in each of the `soft` and `hard` directories.

### Sampling options
 - `DoSampling`: If set to `true`, IPANEMAP will always re-generate a representative structural sample (even if one can already be found)
 - `NumStructures`: Number of structures per condition, generated to approximate the pseudo-Boltzmann ensemble
 - `Temperature`: Temperature (in Celsius) used for the sampling
 - `m` and `b`: Slope and intercept used in the *reactivity to pseudo-energy* conversion (see Deigan et al, PNAS 2009)

### Misc options
 - `WorkingDir`: Main output directory for temp files, and final results of the analysis. Directory  will be created if non-existent.
 - `LogFile`: Name of file gathering the accumulated log. File will be created if non-existent.


## How to...
 - How do I perform a *pure thermodynamic*/constraints-free prediction? 
 Simply make sure that no constraint file named `{RNA}{Cond}.txt` is found in either `{SoftConstraintsDir}` or `{HardConstraintsDir}`, and IPANEMAP will default to a purely thermodynamic sampling (you may safely ignore the warning).
 
Example: Execute the following command with *no* file named `rnathermo.txt` in either of the constraints directories

      IPANEMPA.py --RNA rna.fa --cond thermo
 
 - How do I specify a different sequence for some specific condition? This need arises when minor variants of the original sequence have been probed (eg Mutate-and-Map protocols), and must be used for the sampling.
    - When available, hard constraint files already specify a sequence, which is used instead of the main FASTA file for the sampling.
 Example: For an RNA file `myRNA.fa` and a condition name of `SHAPE`, the sequence found in a `{HardConstraintsDir}/myRNASHAPE.txt` file, will be used for the sampling instead of the one found in `myRNA.fa`.
    - For reactivity/SHAPE data files, if a FASTA file named `{RNA}{Cond}.fa` is found in either of the condition directories, then its sequence will be used instead of the main FASTA file. Example: For an RNA  `rib.fa` and a condition name `1M7`, the sequence found at `{SoftConstraintsDir}/rib1M7.fa` will be used for the sampling instead of the one found in `rib.fa`.
 
