# Exercises [BC]^2 Tutorial


**Goal**: Assemble and annotate a bacterial genome using PacBio HiFi raw data. 

**Additional challenge**: Only use software in containers! 


## Cloud server

```
# extract key files
tar -xvf bc2-users.tar.gz
```
<br>

Login to your cloud instance with your private key:

```
chmod 400 private_keys/key_yourusername.pem ;
ssh -i private_keys/key_yourusername.pem yourusername@3.127.182.45
```

Basic information about containers

On the Linux system we are using today, we use Podman instead of Docker. Don’t worry, Podman has almost identical syntax as Docker. Basic syntax: 

```
# Enter a new container interactively
(host)$ podman run -it --rm -v .:/data staphb/flye /bin/bash
(container)$ flye --help
(container)$ exit
(host)$
```

**Explanation:**

```
--it    interactive and tty: forwards your input to the container 

--rm    automatically remove the container when it exits - today, we always want this to happen 

-v .:/data  the current working directory (.) of the host system will be available at /data in the container 

staphb/flye     the name of the image to be started, here staphb/flye, which contains the software Flye. To get more information about an image, search for it on hub.docker.com 

/bin/bash   Run the (default) bash shell in the container 
```


If you only want to run a command without entering the container, try this: 

```
# Run "flye --help" without entering the container
(host)$ podman run --rm -v $PWD:/data staphb/flye flye --help
```

<br>

**Explanation**

 Because we do not open a shell, -it is not necessary and instead of running /bin/bash, we run flye --help.  

<br>

### Copy data to your local computer

Since we cannot view image files or html reports, we need to synchronise the results back to your local computer.

```
# create a directory 'results'
mkdir results;

# use your private key to copy the results
scp -r -i test_bc2/private_keys/key_<user>.pem   <user>@3.127.182.45:/home/<user>/<sample> results/.

```


## Exersicse instructions

### Overview

1.  Download the reads: https://ngs-longreads-training.s3.eu-central-1.amazonaws.com/project3.tar.gz 

2. Assemble the reads using Flye from the container staphb/flye. 

3.  Assemble the reads using LJA from the container troder/lja. 

4. Compare the output

    - How many contigs? 
    - Could they be circularized?
    - Are they approximately the same size? 

5. Analyze the assemblies using BUSCO from ezlabgva/busco:v5.4.4_cv1
    - Compare the output from different assemblies. 
    - What BUSCO dataset was found and used? 

6. Choose one assembly to continue working with. 

7.    Structurally annotate the assemblies using Bakta from oschwengers/bakta. 
        - Use the arguments: --prefix <yourname>-1.1 and  --locus-tag <yourname>-1.1! 
        - We have already downloaded the “light” reference database for you. It is available at /databases/bakta-light-db (on the host system). 

8.   Annotate the protein FASTAs (.faa) using eggNOG from nanozoo/eggnog-mapper. 
    
        - This does not work today, unfortunately. We will do it for you. 


9. Organize your output as described on the next page! 

 


**Notes**

 Try not to use more than 4 CPUs. 


**Bonus tasks**


- Try out different assembler settings. 

- Use NanoPlot from staphb/nanoplot to perform QC on the raw reads. 

- Use the container troder/gfaviz to visualize the assembly graphs. 


## Organise your output as follows:

If you used flye:

```
/home/$USER/submission/
├── flye.log
├── assembly.fasta
├── assembly_graph.gfa
├── assembly_graph.gv
├── assembly_info.txt
├── gfaviz.svg (optional)
├── bakta
│  ├── yourname-1.1.embl
│  ├── yourname-1.1.faa
│  ├── yourname-1.1.ffn
│  ├── yourname-1.1.fna
│  ├── yourname-1.1.gbff
│  ├── yourname-1.1.gff3
│  ├── yourname-1.1.hypotheticals.faa
│  ├── yourname-1.1.hypotheticals.tsv
│  ├── yourname-1.1.json
│  ├── yourname-1.1.log
│  ├── yourname-1.1.png
│  ├── yourname-1.1.svg
│  ├── yourname-1.1.tsv
│  └── yourname-1.1.txt
├── busco
│  ├── short_summary.generic.bacteria_odb10.busco.json
│  ├── short_summary.generic.bacteria_odb10.busco.txt
│  ├── short_summary.?_odb10.busco.json
│  └── short_summary.?_odb10.busco.txt
├── eggnog
│  └── out.emapper.annotations
├── genome.json  -> {"assembly_tool": "Flye"}
└── organism.json -> {"taxid": 1}

```
<br>

If you used LJA

```
/home/$USER/submission/
├── assembly.fasta
├── dbg.log
├── gfaviz.svg (optional)
├── bakta
│  ├── yourname-1.1.embl
│  ├── yourname-1.1.faa
│  ├── yourname-1.1.ffn
│  ├── yourname-1.1.fna
│  ├── yourname-1.1.gbff
│  ├── yourname-1.1.gff3
│  ├── yourname-1.1.hypotheticals.faa
│  ├── yourname-1.1.hypotheticals.tsv
│  ├── yourname-1.1.json
│  ├── yourname-1.1.log
│  ├── yourname-1.1.png
│  ├── yourname-1.1.svg
│  ├── yourname-1.1.tsv
│  └── yourname-1.1.txt
├── busco
│  ├── short_summary.generic.bacteria_odb10.busco.json
│  ├── short_summary.generic.bacteria_odb10.busco.txt
│  ├── short_summary.?_odb10.busco.json
│  └── short_summary.?_odb10.busco.txt
├── eggnog
│  └── out.emapper.annotations
├── genome.json  -> {"assembly_tool": "LJA"}
└── organism.json -> {"taxid": 1}
```

## Solutions


1. Download and extract data

```
wget https://ngs-longreads-training.s3.eu-central-1.amazonaws.com/project3.tar.gz
tar -xvf project3.tar.gz
rm project3.tar.gz
```

2. Assembly


**Flye**

```
podman run --rm -v .:/data staphb/flye flye -t 10 --plasmids --pacbio-hifi project3/sample_1.fastq.gz -o sample_1/flye
```

**Gfaviz for Flye**

```
podman run --rm -v .:/data troder/gfaviz gfaviz --no-gui --render --group-labels --output sample_1/flye/gfaviz.svg sample_1/flye/assembly_graph.gfa

```

**LJA**


```
 podman run --rm -v .:/data troder/lja lja -t 10 --reads project3/sample_1.fastq.gz -o sample_1/lja
 ```

**Gfaviz for LJA**
 ```
 podman run --rm -v .:/data troder/gfaviz gfaviz --no-gui --render --labels --output sample_1/lja/gfaviz.svg sample_1/lja/mdbg.gfa
 ```

3. Structural Annotation

**BUSCO**

```
 podman run --rm -v .:/busco_wd --user 0:0 -it ezlabgva/busco:v5.4.4_cv1 busco --cpu 10 -i sample_1/flye/assembly.fasta -o sample_1/flye/busco --mode genome --auto-lineage-prok
```


**Bakta**

```
 podman run -it --entrypoint='/bin/bash' --rm -v .:/data -v /databases:/databases --workdir /data oschwengers/bakta
(container)$ bakta --db /databases/db-light --threads 10 --prefix yourname-1.1 --locus-tag yourname-1.1 --output sample_1/flye/bakta sample_1/flye/assembly.fasta
```

OR:

```
podman run --rm -v .:/data -v /databases:/databases --workdir /data --entrypoint='/bin/bash' oschwengers/bakta -c "bakta --db /databases/db-light --threads 10 --prefix yourname-1.1 --locus-tag yourname-1.1 --output sample_1/flye/bakta sample_1/flye/assembly.fasta"
```

4. Functional Annotation

**eggNOG**


See instructions here: https://github.com/eggnogdb/eggnog-mapper/wiki/eggNOG-mapper-v2.1.5-to-v2.1.12#user-content-Basic_usage
Also possible: use online service at http://eggnog-mapper.embl.de/

5. NanoPlot (optional)

```
 podman run --rm -v .:/data staphb/nanoplot NanoPlot --threads 10 --fastq project3/sample_1.fastq.gz --outdir sample_1/NanoPlot
```