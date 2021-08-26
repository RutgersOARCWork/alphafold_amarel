# AlphaFold on Amarel

## Docker file

The Docker file was modifield from [Reaserch Computing at University of Virginia site](https://staging.rc.virginia.edu/userinfo/rivanna/software/alphafold/).  

## AlphaFold launch command and submission

The main job submission is done through:

```
sbatch alphafold_gpu.sh 
```

The submission script is located [here](https://github.com/RutgersOARCWork/alphafold_amarel/blob/main/alphafold_gpu.sh)

The main Singularity command to launch AlphaFold looks like this:


```
singularity run -B $ALPHAFOLD_DATA_PATH:/data -B .:/etc --pwd /app/alphafold --nv $CONTAINERDIR/alphafoldamarel_latest.sif \
    --data_dir=/data \
    --uniref90_database_path=/data/uniref90/uniref90.fasta \
    --uniclust30_database_path=/data/uniclust30/uniclust30_2018_08/uniclust30_2018_08 \
    --mgnify_database_path=/data/mgnify/mgy_clusters_2018_12.fa \
    --bfd_database_path=/data/bfd/bfd_metaclust_clu_complete_id30_c90_final_seq.sorted_opt  \
    --pdb70_database_path=/data/pdb70/pdb70 \
    --template_mmcif_dir=/data/pdb_mmcif/mmcif_files/ \
    --obsolete_pdbs_path=/data/pdb_mmcif/obsolete.dat \
    --preset=full_dbs \
    --fasta_paths=/scratch/pgarias/fastafiles/sarscovid2.fasta \
    --output_dir=/scratch/pgarias/outputdir \
    --model_names=model_1,model_2,model_3,model_4,model_5 \
    --max_template_date=2020-01-01 
```

## Explanation of Singularity flags

1. The database and models are stored in `$ALPHAFOLD_DATA_PATH` and loaded when you run `module load alphafold` after you load `module use /projects/community/modulefiles`.
2. A cache file `ld.so.cache` will be written to `/etc`, which is not allowed on Amarel. The workaround is to bind-mount e.g. the current working directory to `/etc` inside the container. `[-B .:/etc]`
3. You must launch AlphaFold from `/app/alphafold` inside the container due to this issue. `[--pwd /app/alphafold]`
4. The `--nv` flag enables GPU support.


## Explanation of AlphaFold flags 

1. The default command of the container is `/app/run_alphafold.sh`. All flags shown above are required and are passed to `/app/run_alphafold.sh`.
2. As a consequence of the Singularity `--pwd` flag, the fasta path and output directory must be full paths (e.g. `/scratch/$USER/mydir`, not relative paths (e.g. `./mydir`).
3. The `model_names` should be a comma-separated list of `model_*`. See `$ALPHAFOLD_DATA_PATH/params` for the complete set of model names. In [run_docker.py](https://github.com/deepmind/alphafold/blob/main/docker/run_docker.py) `model_1,model_2,model_3,model_4,model_5` is used.
4. The `max_template_date` is of the form `YYYY-MM-DD`.
5. For further explanations and additional options please see [run_alphafold.py](https://github.com/deepmind/alphafold/blob/main/run_alphafold.py).


It should be noted that a user need only to modify the following flags:
1. `--fasta_paths`  
2. `--output_dir`  
3. `--model_names`  
4. `--max_template_date`  


