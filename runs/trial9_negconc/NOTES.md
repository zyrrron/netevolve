# Trial 9 (numtrials 10, master seed 1): "negative concentration" in the long-run check

Source: `netevolve evolve --config examples/default.cfg --seed 1 --numtrials 10`, trial 9
(trial seed 10511824513240686848), branch cvode-timeseries @ 1f8e5da.
Fitness 1 at generation 64 (also 1 under the tight re-check, rtol 1e-10 / atol 1e-14).

## What happens
- t = 0-150: S1/S2 oscillate (36 S2 peaks), so the fitness window t = 50-100 scores 1.
- After the last S2 peak (t ~ 151) S2 decays to ~0 and does not recover in practice.
  S2 is the only species that consumes S0 (J5) and S3 (J3) and drains S1 (J0, J5), so:
  S1 grows linearly (inflow J6 = k6*X0 ~ 25.5 / time unit),
  S3 grows exponentially (J2: S3 + X1 -> 2 S3, rate k2*X1 ~ 0.037),
  reaching > 1e6 (max_conc) at t ~ 441.
- With S2 ~ 0 its equation is dS2/dt = S2 * (k0*S1 - k3*S3 - k4 - k5*S0). Whenever this
  bracket is positive, S2 = 0 is unstable, and integrator error of either sign grows.
  The default-tolerance run (rtol 1e-6 / atol 1e-9) drifts below the -1e-8 guard at t = 135.5,
  which is the "negative concentration" message. Even Tellurium/CVODE at rtol 1e-10 /
  atol 1e-14 produces S2 = -1.49 at t ~ 164, so it is not a netevolve solver bug.

## Conclusion
The long-run verdict FAILED is correct: this is a transient oscillator whose concentrations
then run away. The "negative concentration" text is only the first guard tripped; with tighter
tolerances the run would instead fail on max_conc. The fitness = 1 is a false positive of the
t = 50-100 window (same issue as seed 1 on main).

Files: best_final.ant / .csv, log.csv, run.cfg from the trial; sim500.csv = `netevolve simulate
--t-end 500 --points 5001` (stops at t = 135.5).
