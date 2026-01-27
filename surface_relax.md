% 表面緩和

[SQUID](squid.html)にインストールした[GPAW](gpaw.html)で表面緩和を実行する方法を説明する。

## 計算条件
- Ag(111)-(1x1)単位胞
- 3層スラブのうち最下層を固定
- LCAOのdzp基底
- PBE交換相関汎関数
- 24x24x1k点サンプリング

## サンプルスクリプト

次のスクリプトを`input.py`に保存する。

~~~{.prettyprint}
from ase.io import write
from ase.build import fcc111
from ase.constraints import FixAtoms
from ase.optimize import QuasiNewton
from ase.parallel import paropen as open
from gpaw import GPAW, MethfesselPaxton, Mixer
from gpaw import extra_parameters
extra_parameters['blacs'] = True
import numpy as np

a = 4.14498365156092
vac = 3.0
ag = fcc111('Ag', size=(1,1,3), a=a, vacuum=vac)
ag.cell[2,2] += 4.0
ag.set_constraint(FixAtoms(range(0,1)))
system = ag
system.calc = GPAW(mode=PW(400), xc='PBE',
                   occupations=MethfesselPaxton(0.05, order=1), maxiter=999,
                   mixer=Mixer(nmaxold=5, beta=0.05, weight=75),
                   nbands='110%', kpts=(24, 24, 1), txt = 'eval.txt',
                   parallel={'sl_auto':True})

fd = open('optimization.txt', 'w')

print(system.get_positions(), file=fd)
print(system.get_potential_energy(), file=fd)

relax = QuasiNewton(system, logfile='qn.log')
relax.run(fmax=0.02)
print(file=fd)
print(system.get_positions(), file=fd)
print(system.get_potential_energy(), file=fd)
fd.close()

write('ag.traj', system)
~~~

ジョブスクリプトは[格子定数](lattice_const.html)を計算したときと同じファイルを使う。

## 表面構造の変化

表面緩和前後の原子座標とエネルギーは`optimization.txt`に書かれている。

~~~{.console}
$ cat optimization.txt
[[1.46547302 0.84609124 3.        ]
 [0.         1.69218249 5.39310743]
 [0.         0.         7.78621485]]
-6.83449250941794

[[ 1.46547302e+00  8.46091245e-01  3.00000000e+00]
 [-9.07751113e-22  1.69218249e+00  5.39600102e+00]
 [ 6.00195386e-19  0.00000000e+00  7.79266054e+00]]
-6.834636822900305

~~~
