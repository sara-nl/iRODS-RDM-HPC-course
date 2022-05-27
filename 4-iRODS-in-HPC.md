<img align="right" src="images/surf.jpg" width="100px">
<br><br>


# iRODS data handling and use in HPC processing pipelines

**Authors**
- Arthur Newton (SURF)

**License**
Copyright (c) 2020, SURFsara. All rights reserved.
This project is licensed under the GPLv3 license.
The full license text can be found in [LICENSE](LICENSE).


## Goal

Learning about the CLI tool icommands of iRODS and how to use it in High Performance Computing environments for data processing and keeping provenance on all your data. 
Here we focus on how to use these tools in a data processing pipeline at a SLURM based compute cluster with a shared scratch space near the worker nodes and getting data from iRODS to this environment. 
This is a followup section of [iRODS icommands hands-on](2-iRODS-icommands.md):

- searching for datasets of interest via `iquest`
- downloading the dataset to scratch space accessible by worker nodes on Lisa
- performing a simple processing pipeline
- adding results back to iRODS and add metadata to keep provenance
- adding everything in a jobscript
- submit the workflow to the scheduling system of the HPC cluster.


## Login to Lisa and iRODS

If you are not logged in, login to the Lisa compute cluster with your appropriate credentials as explained in your previous section:

```sh
ssh lcur##@lisa.surfsara.nl
```

Make sure you are connected to iRODS.
Try out a quick `ils` to see if you are:

```sh
ils

<optional> iinit
```

We will be uses the program `GNU parallel` to 'parrallelize' processes. 
You need to run it once and accept the terms of conditions.


## Data on an HPC cluster

On most HPC clusters you have access to different file systems which come with different advantages and disadvantages. 

- Your **home** folder is on a file system which is mounted to all worker nodes in the compute cluster, i.e. all data which is stored there is accessible and writable from all worker and data nodes.
However, the quota on that files system per user is normally restricted and the data access is slower than scratch space (on Lisa).
- If you need fast access to data during the computation you would use the **scratch** file system.
Each node in the compute cluster has its own **scratch space**.
It is thus not shared between nodes and you need to make sure that data is stored there before you access it from your compute workflow.
- Project space for Lisa/Cartesius.
This is a shared space which can be used by more than one researchers.
Especially useful if you require the same data to be at the compute resources.

You can get the path to your **home** folder with:

```sh
echo $HOME
```

and the **scratch** space with:

```sh
echo $TMPDIR
```


## Compute workflow - Wordcount

We will now prepare the compute workflow as need be executed on any worker node in the cluster.
The workflow will be a very simple word count of text files already existing inside the iRODS instance which are annotated with metadata.

We still do the workflow in interactive mode and on the user interface node (remember it behaves as any node in the cluster).


### Create data folders on **scratch**

For the compute job, we want to copy the data directly from iRODS to the scratch space, `$TMPDIR` and for this we will create both an input data folder and output data folder for the job to write to.

```sh
random_number=$RANDOM
inputdir="$TMPDIR/inputdat$random_number"
outputdir="$TMPDIR/outputdat$random_number"
mkdir $inputdir
mkdir $outputdir
```

Note that for now we will use `$RANDOM` to create an unique folder name. When we will create the jobscript we will use `$SLURM_JOBID`.


### Query files and download them to scratch space

We will query for the data objects based on the metadata we are interested in.
For now it is as simple as the name of an author.
This could be a more advanced query if you have decided on a standardized metadata scheme which is interoperable with your analysis program.

```sh
iquest "%s/%s" "select COLL_NAME, DATA_NAME where META_DATA_ATTR_NAME = 'author' and META_DATA_ATTR_VALUE = 'Lewis Carroll'"
```

If we are sure we found the correct file names, we can also download these files immediately by using `parallel` and store in the input folder:

```sh
iquest "%s/%s" "select COLL_NAME, DATA_NAME where META_DATA_ATTR_NAME = 'author' and META_DATA_ATTR_VALUE = 'Lewis Carroll'" | parallel iget {} $inputdir
```


### Performing the analysis

We are going to use the files just downloaded, as input files for the analysis and store the files in the output data folder.
First we are going to 'clean' the data, and use `awk` as 'analysis' tool.

```sh
cat $inputdir/* | tr '[:upper:]' '[:lower:]' | awk '{for(i=1;i<=NF;i++) count[$i]++} END {for(j in count) print j, count[j]}' > $outputdir/results.dat
```


### Uploading the results to iRODS

Now we will upload the files to iRODS (for now simply the home folder, however, it would be better if there is a structured directory tree.

```sh
iput $outputdir/results.dat
```


### Adding provenance data to the results

Final step to close the loop, is to add provenance data in the metadata of the results data object linking it to the raw data.

Remember that metadata items in iRODS are string triples: key,value, unit.
Try to always use standard formats to make your data interoperable with other datasets.
Of course, there is no such template for this small processing pipeline. 
 
```
imeta add -d results.dat 'somekey' 'somevalue'
imeta ls -d results.dat
```

**Exercise**
What metadata items would help to keep the provenance of your results? *i.e.* if a researcher in the future (someone else or yourself) sees `results.dat`, what information would that person need to know how this dataset came to be?


## Aggregating all steps and creating a jobscript

All steps that we did above, we can put into one jobscript and submit this job to the Lisa queue.
The job will be schedules to run according to the priority, queue and allocated resources.
See the userinfo pages of Lisa if you want to know more details.

A simple jobscript looks like:

```sh
#!/bin/bash
#Set job requirements
#We are going to use only one node (-N 1), and use the express queue (-p short). And we set the time to 4 minutes (-t 4:00)
#SBATCH -p short
#SBATCH -N 1
#SBATCH -t 4:00

#Make sure you have logged in to your iRODS zone prior to job submission. iRODS creates a irodsA file which is subsequently used by the worker nodes.

#move to your home directory and current git repository which is also mounted on your scratch space and might hold the processing script
cd $HOME/iRODS-RDM-HPC-course

rodscoll='/yoda/home/research-rdmcourse/YOUR OUTPUT COLLECTION'

inputdir="$TMPDIR/inputdat$SLURM_JOBID"
outputdir="$TMPDIR/outputdat$SLURM_JOBID"
mkdir $inputdir
mkdir $outputdir

#search for data objects with iquest
#get these files from iRODS and store them under scratch
iquest "%s/%s" "select COLL_NAME, DATA_NAME where META_DATA_ATTR_NAME = 'author' and META_DATA_ATTR_VALUE = 'Lewis Carroll'" | parallel iget {} $inputdir

#perform the word count analysis
resultsfile=results$SLURM_JOBID.dat
cat $inputdir/* | tr '[:upper:]' '[:lower:]' | awk '{for(i=1;i<=NF;i++) count[$i]++} END {for(j in count) print j, count[j]}' > $outputdir/$resultsfile

#put results back into iRODS
iput $outputdir/$resultsfile $rodscoll

#add metadata provenance
#possible add a metadata annotation script call either locally via scipt file, rule file or server side via installed rules, where last is preferred but also difficult to implement.
imeta add -d $rodscoll/$resultsfile 'somekey' 'somevalue'
imeta ls -d $rodscoll/$resultsfile
```

You can submit the job as follows:

```sh
sbatch jobscript
```

There are numerous tools to monitor your job but the simplest way is to check the queue for jobs under your name:

```sh
squeue -u $USER
```

When your job finishes, the output of the job (also if an error has occurred) will be put into the folder you submitted your job to. 


## What's Next?

You have performed a very simple data processing pipelines.
You can create a much more complicated pipeline with mostly the same tools.
By using tools like `parallel` you can optimize your job and analyze your data concurrently. 

Similar to every large scientific project where you need to write data management plans, you need to think about how your data processing pipelines can fit your data management processes.
Thinking about a proper interoperable metadata scheme is not trivial and you should allocate time to it.
You will gain a lot of time and effort in the long run, *e.g.* your MsC project, PhD or PostDoc. 



