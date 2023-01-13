Never run with MERRA-2 emissions. It will overwrite your emissions choices.

Never use gcm_emip.setup. It will overwrite your restarts.

Check that the Hentry's law coefficients for CO2 are 0.

If CoDAS isn't assimilating observations, you're ssh key may be mangled. Try running
```
ssh discover
```
and make sure it works.
