# TODO

## Goal
Rerun the GROMACS minimization from the existing minimized structure so the final maximum force is below `10 kJ mol^-1 nm^-1`.

## Inputs
- Use `finalsystem_min.gro` as the restart coordinate file.
- Reuse the original topology and include files from the existing setup.
- Reuse the original minimization nonbonded settings unless `grompp` forces a change.

## Stage 1: Strict Steepest-Descent Minimization
1. Create a new MDP file, `min_strict_steep.mdp`.
2. Set:
   - `integrator = steep`
   - `emtol = 10`
   - `emstep = 0.001`
   - `nsteps = 50000`
3. Keep the same cutoff and electrostatics settings as the original minimization.
4. Build the run input:
   - `gmx grompp -f min_strict_steep.mdp -c finalsystem_min.gro -p topology/topol.top -o finalsystem_min_strict.tpr`
5. Run the minimization:
   - `gmx mdrun -deffnm finalsystem_min_strict -v`

## Stage 2: Conjugate-Gradient Fallback
1. Check the stage-1 log.
2. If `Maximum force < 10`, stop.
3. If `Maximum force > 10`, create `min_strict_cg.mdp`.
4. Set:
   - `integrator = cg`
   - `emtol = 10`
   - `nsteps = 50000`
5. Keep the same cutoff and electrostatics settings as stage 1.
6. Build the fallback run input:
   - `gmx grompp -f min_strict_cg.mdp -c finalsystem_min_strict.gro -p topology/topol.top -o finalsystem_min_cg.tpr`
7. Run the fallback minimization:
   - `gmx mdrun -deffnm finalsystem_min_cg -v`

## Validation
- Confirm `grompp` succeeds without topology/include errors.
- Confirm the final minimization log reports `Maximum force < 10`.
- Confirm there are no fatal errors or signs of unstable energies.
- Confirm PFDA remains in a reasonable position in the bilayer.

## Notes
- The previous run already converged, but only to `Fmax < 100`.
- It stopped at about `93.79 kJ mol^-1 nm^-1` on atom 8 of `PF10C`.
- This looks like a loose minimization target, not an obvious catastrophic overlap.
- Do not rebuild the bilayer or manually move the sodium unless the stricter rerun still fails badly.
