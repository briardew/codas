1. Find the line starting with `tropomi_ch4_s5p` in `analyze/etc/tgasinfo`.
a. Increase the error inflate entry to 5.0

2. Open a debug queue (`salloc --ntasks=486 --constraint="[hasw|sky]" --account=s1460 --qos=debug`)
a. Run `./codas_run.j` and wait until you start seeing some lines that look like:

```
    Observation Type           Nobs                        Jo        Jo/n
trace gases                   31398    1.0532275097286580E+05       3.354
                               Nobs                        Jo        Jo/n
           Jo Global          31398    1.0532275097286580E+05       3.354
```

You want the last number (`Jo/n`) in those lines to start at something a big bigger than 1, say 1.8 or 2.4 (3.3 seems ok), and finish at something less than 1, say 0.8 or so.

From this number alone, it's not entirely obvious if you should increase R (the obs error covariance) or B (the model error cov). From our experiments, it looked like we were trying too hard to fit the data and adding variability that wasn't in the original model run. So that's why I would suggest increasing that error inflation line from 3 to 5.

So you're going to run for a while and check if the run hits this metric. Try experimenting with the error inflation factor until you can get these numbers to go below 1 after the GSI runs: they are all always > 1 right now, for example.

You can always grep for "trace gases      " (with the spaces) to find these lines in slurm files.

The other number you could change (the model error, `B`) is set in the `analyze/codas.rc` file. The number there will determine how big the differences are in the initial Jo/n value and the final Jo/n value.
