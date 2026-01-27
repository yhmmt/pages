% GOFEE

[SQUID](squid.html)にインストールした[GPAW](gpaw.html)でGOFEEによる構造探索を実行する方法を説明する。


## インストール

GOFEEのソースコードを```~/GPAW/src```にダウンロードして展開する。

~~~{.console}
$ cd
$ source ~/py38env/bin/activate
(...) $ cd GPAW/src
(...) $ wget http://grendel-www.cscaa.dk/mkb/_downloads/920d389b1e05cf17f747d8e1eb83f642/gofee.tar.gz
(...) $ tar xzvf gofee.tar.gz
~~~

GPFEEを```~/GPAW/src/gofee```でコンパイルする。

~~~{.console}
(...) $ cd gofee
(...) $ ./build_code
~~~

## テスト計算

ここではベンゼンのサンプルを用いてGOFEEが正しく動作するか確認する。


~~~{.console}
$ cd ~/GPAW/src/gofee/test_systems/C6H6
~~~

GOFEEのディレクトリ構造が変更されたので、`run_search.py`を以下のように修正する。

Line 8:

~~~{.prettyprint}
from gofee.surrogate.gpr import GPR
~~~

Line 11:

~~~{.prettyprint}
from gofee.candidates.candidate_generation import CandidateGenerator, StartGenerator
~~~

Line 12:

~~~{.prettyprint}
from gofee.candidates.basic_mutations import RattleMutation, RattleMutation2, PermutationMutation
~~~

Line 16:

~~~{.prettyprint}
from gofee.gofee import GOFEE
~~~

Lines 40-41:

~~~{.prettyprint}
calc=GPAW(mode = 'lcao',
~~~

Line 57:

~~~{.prettyprint}
               candidate_generator=mutationSelector,
~~~

Line 58:

~~~{.prettyprint}
               max_steps=300,
~~~


Line 59:

~~~{.prettyprint}
               # dmax_cov=2.5,
~~~

To run GOFEE, save the following script as, e.g., `run.sh` in the current directory

~~~{.prettyprint}
#!/bin/sh
#------- qsub option -----------
#PBS -q SQUID
#PBS --group=グループ名
#PBS -m be
#PBS -M メールアドレス
#PBS -l cpunum_job=76
#PBS -l elapstim_req=1:00:00
#PBS -N C6H6
#------- Program execution -----------
module load BaseCPU/2022
export MYHOME=/sqfs2/cmc/0/home/ユーザ名
export PYTHONPATH=${PYTHONPATH}:${MYHOME}/GPAW/src/gofee
cd $PBS_O_WORKDIR

mpirun ${NQSV_MPIOPTS} -np 76 ${MYHOME}/py38env/bin/python run_search.py > search.log
~~~

and submit the job as

~~~{.console}
$ qsub run.csh
~~~

The progress of the calculation can be traced as, e.g.,

~~~{.console}
$ tail -f search.log | grep "energy \| STEP"
~~~

The evolution of the structure is registered in `structures.traj`, which can be visualized as

~~~{.console}
$ ~/myenv/bin/ase gui structures.traj
~~~

or just

~~~{.console}
(myenv) $ ase gui structures.traj
~~~

when the virtual environment is activated.

## Analyze multiple results

Since the results should be treated statistically, let us submit multiple jobs in directory `runs0`.
To submit, e.g., three jobs in `run0`, `run1`, and `run2` created in `runs0`, 
modify `run.sh` as

~~~{.prettyprint}
#!/bin/bash
#------- qsub option -----------
#PBS -q SQUID
#PBS --group=グループ名
#PBS -m be
#PBS -M hamamoto@c.oka-pu.ac.jp
#PBS -l cpunum_job=76
#PBS -l elapstim_req=1:00:00
#PBS -N C6H6
#PBS -T intmpi
#PBS -v OMP_NUM_THREADS=1
#------- Program execution -----------
module load BaseCPU/2022
export MYHOME=/sqfs2/cmc/0/home/ユーザ名
export PYTHONPATH=${PYTHONPATH}:$MYHOME/GPAW/src/gofee

cd $PBS_O_WORKDIR

dir=runs0/run${TASK_ID}
mkdir -p $dir
cp run_search.py ${dir}
cp slab.traj ${dir}
cd ${dir}

mpirun ${NQSV_MPIOPTS} -np 76 $MYHOME/py38env/bin/python run_search.py > search.log
~~~

and run the following command

~~~{.console}
$ for ((i=0; i<3; ++i)); do qsub -v TASK_ID=$i run.sh; done
~~~

where the environment variable TASK_ID is used in run.sh to distinguish the different jobs.
If the multiple jobs are submitted successfully, output files are generated in each directory as

~~~{.console}
$ ls -R runs0
runs0:
run0  run1  run2  run3

runs0/run0:
eval.txt  restart  run_search.py  search.log  slab.traj  structures.traj

runs0/run2:
eval.txt  restart  run_search.py  search.log  slab.traj  structures.traj

runs0/run3:
eval.txt  restart  run_search.py  search.log  slab.traj  structures.traj
~~~

Python scripts useful for analyzing these results are included in sibling directory `../statistics_tools` as

~~~{.console}
$ ls ../../statistics_tools
get_best_structures.py  plot_energy_evolution.py  survival_statistics
$ ls ../../statistics_tools/survival_statistics
__pycache__  collect_events.py  survival_stats.py
~~~

### Best structures

`get_best_structures.py` extracts the lowest energy from `structures.traj` in `run0`, `run1`, `run2`, ... and is used by specifying their parent directory, i.e. `runs0`, as

~~~{.console}
$ python3 ../../statistics_tools/get_best_structures.py runs0
0: runs3/run0/structures.traj
Ncalc: 610 Ebest: -68.35547274831681
1: runs3/run1/structures.traj
Ncalc: 610 Ebest: -63.94381148644545
2: runs3/run2/structures.traj
Ncalc: 610 Ebest: -73.32807093380796
.
.
.
[Errno 2] No such file or directory: 'runs0/run40/structures.traj'
~~~

Note that the script assumes that `runs0` includes 100 working directories from `run0` to `run99` as hard-coded in line 16 of the script:

~~~{.prettyprint}
for i in range(100):
~~~

This results are registered in `temp_best.traj` in the current directory, which will be used to calculate the cumulative success rate as seen below.

### Cumulative success rate

~~~{.prettyprint}
export PYTHONPATH=/home/<your-name>/GPAW/src/gofee/statistics_tools/survival_statistics
~~~

### Energy evolutions

`plot_energy_evolution.py` visualizes how the energy evolves in each structure search and is used by specfying the parent directory, the energy range (`dE`), and the number of the working directories (`Nruns2use`) as

~~~{.console}
$ python3 ../../statistics_tools/plot_energy_evolution.py runs0 80 8
progress: 1/8
Ebest=-68.35547274831681
progress: 2/8
Ebest=-63.94381148644545
progress: 3/8
Ebest=-73.32807093380796
.
.
.
len(E) 610
len(E) 610
len(E) 610
.
.
.
~~~
where the first eight results are shown. Note that `Nruns2use` is set to `12` if `Nruns2use` is unspecified, and in addition `dE` is set automatically if both the two arguments are unspecified. 

![](energyEvol_C6H6.png)

The figure is also saved as `energyEvol_C6H6_runs0.pdf` in the current directory.

#### Cumulative success rate

`collect_events.py` collects the numbers of iterations to achieve a structure close to the global minimum for the first time in respective structure searches, and `survival_stats.py` calculates the cumulative success rate from the data. To use these scripts, we add the following path to `$PYTHONPATH` in e.g. `~/.bashrc`

~~~{.prettyprint}
export PYTHONPATH=/home/<your-name>/GPAW/src/gofee/statistics_tools/survival_statistics
~~~

and 

~~~{.console}
$ source ~/.bashrc
~~~

Then, the functions defined in these scripts can be used as

~~~{.prettyprint}
from collect_events import collect_events_energy
from survival_stats import survival_stats
from ase.io.trajectory import Trajectory
from ase.io import read, write
import numpy as np

traj = Trajectory('temp_best.traj')
energies = [a.get_potential_energy() for a in traj]
a_gm = traj[np.argmin(energies)]

times, events = collect_events_energy(a_gm, 'runs0/', dE_max=0.2, N=40,
                                      traj_name='success_stat.traj',
                                      save_dir='success_curve')

survival_stats(times, events, labels=['curve 1'], save_dir='success_curve')
~~~

where `temp_best.traj` obtained above (see the description of `get_best_structures.py`) is used to identify the global minimum. Note also that `'runs0/'` and `'N=40'` denote directory `runs1` includes forty working directories. The above script can be copied from `/home/hamamoto/GPAW/src/gofee/test_systems/C6H6/success_rate.py`

~~~{.console}
$ python3 success_rate.py
run: 2   iter_found: 380         Energy: -73.17817593291628
run: 4   iter_found: 194         Energy: -73.26306007011353
run: 5   iter_found: 589         Energy: -73.230318440068
.
.
.
~~~

![](successRate_C6H6.png)


## Learn more about GOFEE

The information of GOFEE is available at [The documentation for GOFEE](http://grendel-www.cscaa.dk/mkb/).

