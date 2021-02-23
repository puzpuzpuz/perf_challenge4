# Build

```bash
$ cd canny_baseline
$ mkdir build
$ cd build
# cmake also honors the following env variables:
# export CC=/usr/bin/clang
# export CXX=/usr/bin/clang++
$ cmake .. -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
$ make
```

# Analyze

Hot spots:
```bash
perf record -- ${DIR}/build/canny ${DIR}/build/221575.pgm 0.5 0.7 0.9
perf report -n --stdio
```

CPU cache stats:
```bash
perf stat -e L1-dcache-load-misses,L1-dcache-loads,LLC-load-misses,LLC-loads ${DIR}/build/canny ${DIR}/build/221575.pgm 0.5 0.7 0.9
```

Auto-vectorized loops (other helpful flags are `-Rpass-missed=loop-vectorize`, `-Rpass-analysis=loop-vectorize`):
```bash
clang -O3 -Rpass=loop-vectorize -S canny_source.c -o /dev/null
```

# Baseline

```bash
$ ../run-and-compare.sh
+ cp /home/puzpuzpuz/projects/perf_challenge4/canny_baseline/../221575.pgm /home/puzpuzpuz/projects/perf_challenge4/canny_baseline/build
+ /home/puzpuzpuz/projects/perf_challenge4/canny_baseline/build/canny /home/puzpuzpuz/projects/perf_challenge4/canny_baseline/build/221575.pgm 0.5 0.7 0.9

real	0m1.812s
user	0m1.608s
sys	0m0.156s
+ /home/puzpuzpuz/projects/perf_challenge4/canny_baseline/build/validate /home/puzpuzpuz/projects/perf_challenge4/canny_baseline/../221575-out-golden.pgm /home/puzpuzpuz/projects/perf_challenge4/canny_baseline/build/221575.pgm_s_0.50_l_0.70_h_0.90.pgm
Rows 4320, cols 7680, max_pixel = 255
Rows 4320, cols 7680, max_pixel = 255
Validation successful
```

# Hot spots

```bash
# Samples: 7K of event 'cycles'
# Event count (approx.): 7018229130
#
# Overhead       Samples  Command  Shared Object      Symbol                
# ........  ............  .......  .................  ......................
#
    40.27%          2854  canny    canny              [.] gaussian_smooth
    20.62%          1460  canny    canny              [.] derrivative_x_y
    16.57%          1174  canny    canny              [.] non_max_supp
    11.77%           832  canny    canny              [.] apply_hysteresis
     1.70%           120  canny    canny              [.] follow_edges
     1.62%           115  canny    canny              [.] canny
```
