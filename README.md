ansible-slurm-scratchspace 
=========

This role augments the [ansible-slurm](https://github.com/galaxyproject/ansible-slurm) role by installing prolog and epilog scripts used to manage locally-attached scratch space.

It assumes that the variable that is defined under `slurm_config.TmpFS` represents a local mount that exists on each node in the cluster. This mount should use the XFS filesystem and be mounted with the `uquota` option.

After this role has been layered on top of `ansible-slurm`, a `slurmd`-invoked prolog script will dynamically adjust the XFS quota on a node after the first job step for a job that has been scheduled based off a value given to the `--tmp` flag when running `sbatch` or `srun`.  A task prolog script will then create a temporary directory based off the username of the job owner and the job id and save the path to this directory within a bash variable (`SLURM_SCRATCH`), which can then be used within job scripts.

When a job has finished (or been requeued) a task epilog script removes all files within the scratch space.  The `slurmd`-based epilog script will then set the XFS quota for the local drive to the value of `slurm_default_scratch_quota` (see below).

Array jobs are currently not supported by this quota-adjustment mechanism.

A mechanism to deal with the case where cluster users write files outside of the directory represented by the `SLURM_SCRATCH` also is absent.

Requirements
------------

`ansible-slurm` must be run before this role.

Role Variables
--------------

`slurm_prolog_dir`: A directory that contains task prolog scripts, which will be executed sequentially (similar to _/etc/profile.d_).

`slurm_epilog_dir`: A directory that contains task epilog scripts, which will be executed sequentially (similar to _/etc/profile.d_).

`slurm_config`: Identical to the `slurm_config` variable in `ansible-slurm`.  The keys `TmpFS`, `TaskProlog`, `TaskEpilog`, `Prolog`, and `Epilog` must be defined.

`slurm_default_scratch_quota`: The default quota that all users should be given (in KB).

Dependencies
------------

- [ansible-slurm](https://github.com/galaxyproject/ansible-slurm)

Example Playbook
----------------

    - hosts: servers
      roles:
         - ansible-slurm-scratchspace

License
-------

BSD

Author Information
------------------

- [John Pellman](https://github.com/jpellman)
