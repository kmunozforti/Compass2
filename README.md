# Compass

## In-Silico Modeling of Metabolic Heterogeneity using Single-Cell Transcriptomes
Metabolism is a major regulator of immune cell function, but it remains difficult to study the metabolic status of individual cells. This motivated the development of Compass, an algorithm to characterize cellular metabolic states based on single-cell RNA-Seq and flux balance analysis (FBA).

For detailed instructions on how to install and use Compass, visit the [documentation][link-docs]. For an in-depth description of the algorithm, refer to Wagner et al., <i>Cell</i> 2021 ([link][link-manuscript]).

## Installation

### Requirements
```
 - python >= 3.10
 - gurobipy >= 11.0.0
 - numpy >= 1.12
 - pandas >= 0.20
 - scikit-learn >= 0.19
 - scipy >= 1.0
```

### Install Compass
If Numpy is not installed, you can install it with

```
python -m pip install numpy
```
   
This needs to be installed before the other requirements because a C extension needs the location of numpy headers to compile.

> [!NOTE]
> Accessing pip through python -m pip is done to emphasize that on systems with multiple python installations (e.g. python  2.7 and python 3.6) Compass and Cplex must be installed to the same version of python. It is otherwise identical to just using pip. Using sudo can also invoke a different version of python depending on your environment.

Then simplest way to install Compass is using pip to install from the github repository. This can be done with the following command in the terminal:

```
python -m pip install git+https://github.com/wagnerlab-berkeley/Compass.git --upgrade
```

This command will also update Compass to the newest version. Alternatively, you can clone the Compass repository and run setup.py.

Now to test if everything is installed, simply run:

```
compass -h
```

You should see the help text print out if installation was succesful. For more details on how to use Compass you can visit our tutorial.

### Obtain a Gurobi WLS license

At the heart of the Compass algorithm is linear programming. Compass computes a penalty score for each reaction in 
each cell by solving a constrained optimization problem using the Gurobi linear solver. 
In order to use Compass, you need to obtain a Gurobi WLS license, which is free for academic use.

Please refer to [this link][link-gurobi] to obtain a Gurobi WLS license. If you follow the instructions correctly, you should be able to obtain a ```gurobi.lic``` file that contains the access ID, secret, and license ID of your WLS license. 
Please store this file as it is required to set up Compass.

### Set up your Gurobi WLS license

In order to use Compass, you must provide your Gurobi WLS license to the Gurobi API. To do this, you can run the command

```
compass --set-license <PATH_TO_LICENSE>
```

where ```<PATH_TO_LICENSE>``` is the path to the ```gurobi.lic``` file you obtained in the previous step. 
This stores your license information in a default location within Compass. 
After you have successfully done so, you can proceed to run Compass without the ```--set-license``` parameter 
as Compass will directly read your license information from the default location.
Refer to the [next section][link-quickstart] of the documentation to learn how to run Compass.


## Quickstart

Broadly speaking, Compass takes in a gene expression matrix scaled for library depth (e.g., CPM) 
and outputs a reaction score matrix, where higher scores correspond to a reaction being **less** likely.

### Input Data

The input gene expression matrix can be a tab-delimited text file (tsv) or a matrix market format (mtx) 
containing gene expression estimates (CPM, TPM, or similar scaled units) with one row per gene, one column per sample.
We also support AnnData objects as input.

Tab-delimited files need row and column labels corresponding to genes and sample names. 
Market matrix formats need a separate tab delimited file of gene names and optionally a tab delimited file of cell names.
AnnData objects should contain **normalized counts** in the ``adata.X`` slot.

### Example Input

You can find example inputs in tab-delimited format (tsv) and market matrix format (mtx) 
on this github repo under [compass/Resources/Test-Data][link-testdata].

These files will exist locally as well under the Compass install directory which can be found by running:

```
compass --example-inputs --species homo_sapiens
```

Human or mouse species makes no difference for this command.

### Running Compass

After opening a command line in a directory with an input file ``expression.tsv``, 
you can run Compass on the data with the following command, which will limit the number of processes used to 10:

```
compass --data expression.tsv --num-processes 10 --species homo_sapiens
```


To run Compass on mtx formatted data, use the following command:

```
compass --data-mtx expression.mtx genes.tsv sample_names.tsv --num-processes 10 --species homo_sapiens
```

To run Compass on AnnData objects, use the following command:

```
compass --data anndata_object.h5ad --num-processes 10 --species homo_sapiens
```

Though the sample names file can be omitted, in which case the samples will be labelled by index.

Below is an example of the formatting for gene expression (we only show a small portion of the matrix):

.. image:: images/input_ex.png

For the first run of Compass on a given model and media there will be overhead building up the Compass cache. 
Compass will automatically build up the cache if it is empty, but you can also manually build up the cache 
before running Compass with:

```
compass --precache --species homo_sapiens
```

> [!NOTE]
> For every individual sample, Compass takes roughly 30 minutes to calculate the reaction penalties (varying by machine). This can be expedited by running more than one process at once. In addition, Compass saves the results of all samples that it has already processed in the _tmp directory. Therefore, Compass can also be stopped and restarted after it is done processing a subset of samples so long as the _tmp directory is still there.

For an in-depth explanation of the various Compass parameters, see [here][link-settings].

### Output

When Compass has completed, the outputs for all samples are stored in a tab delimited file reactions.tsv 
in the specified output directory (. directory when running Compass by default).

Below is an example of the output matrix:

![Compass Output](docs/images/output_ex.png)

To get more context on what the RECON2 reaction identifiers are, you can visit [virtual metabolic human][link-virtualmetabolichuman] or the [resources directory][link-resourcesdir] of Compass where there are several .csv files which include information on the reactions in Recon2.

If you are using Human1 or Mouse1, you can visit [Metabolic Atlas][link-metabatlas] to view the metabolic network.

> [!NOTE]
> While Compass is running, it will store partial results for each sample in the _tmp directory (or the directory following --temp-dir)

### Postprocessing

Once Compass has finished running, we apply several steps of postprocessing to the data. 
More specifically, postprocessing converts reaction penalties (where high values correspond to low likelihood reactions) 
to reaction scores (where high values correspond to likely reactions). 
Refer to [this page][link-postprocessing] of the documentation for an example notebook.



[link-docs]: https://compass-sc.readthedocs.io/en/latest/
[link-manuscript]: https://doi.org/10.1016/j.cell.2021.05.045
[link-gurobi]: https://support.gurobi.com/hc/en-us/articles/13232844297489-How-do-I-set-up-a-Web-License-Service-WLS-license
[link-quickstart]: https://compass-sc.readthedocs.io/en/latest/quickstart.html
[link-testdata]: https://github.com/wagnerlab-berkeley/Compass/tree/master/compass/Resources/Test-Data
[link-settings]: https://compass-sc.readthedocs.io/en/latest/settings.html
[link-virtualmetabolichuman]: https://www.vmh.life/#home
[link-resourcesdir]: https://github.com/wagnerlab-berkeley/Compass/tree/master/compass/Resources/Recon2_export
[link-metabatlas]: https://metabolicatlas.org/
[link-postprocessing]: https://compass-sc.readthedocs.io/en/latest/notebooks/postprocessing.html