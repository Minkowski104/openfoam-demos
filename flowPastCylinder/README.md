welcome to the CFD bench. We're running a canonical 2D flow-past-a-cylinder case with `icoFoam`. Goal: get a working transient run, produce velocity/pressure snapshots, and the `forceCoeffs` time series. You'll hand me (a) a `checkMesh` report that's clean, (b) a screenshot or ParaView state showing vortex shedding, and (c) a short table of average drag coefficient for two mesh sizes.

I'm going to give you a clear checklist, the exact commands, and what I expect. Work through in order and report back with the artifacts listed. I'll comment on anything you post.

* * * * *

Task 0 --- environment (quick)
----------------------------

-   Are you on Linux? (I assume yes.)

-   You need OpenFOAM installed (any recent distro of OpenFOAM) and ParaView. If not installed, tell me the distro and I'll give install commands.\
    (If already installed, skip this step.)

* * * * *

Task 1 --- create case skeleton (30--60 min)
-----------------------------------------

1.  Make a folder `~/cases/flowPastCylinder/`.

2.  Create subfolders: `0/`, `constant/`, `system/`.

3.  Put a thin `cylinder.stl` in `constant/triSurface/`. If you don't have one, generate with OpenSCAD:

    `translate([0.5, 0.205, 0]) cylinder(h=0.01, r=0.05, $fn=96);`

    Export `cylinder.stl`, place at `constant/triSurface/cylinder.stl`.

4.  Create `system/blockMeshDict` (use the blockMeshDict I previously gave). Save it.

**Command to run next:**

`cd ~/cases/flowPastCylinder
blockMesh`

**Deliverable 1 (send to me):** `blockMesh` output and `checkMesh` after snappy (or at least blockMesh success). If `blockMesh` fails, paste the console output.

* * * * *

Task 2 --- snappyHexMesh & surface features (30--90 min)
-----------------------------------------------------

1.  From case root, run:

`surfaceFeatureExtract constant/triSurface/cylinder.stl   # optional, may require utilities
snappyHexMesh -overwrite
checkMesh`

1.  If `surfaceFeatureExtract` fails (not available), run `snappyHexMesh` without features --- that's fine for now.

**What I must see:**

-   `checkMesh` output. I expect `overall OK` and no `non-orthogonality > ~65` warnings. If warnings appear, note them.

**If checkMesh fails**: don't panic --- common fixes:

-   Increase base mesh resolution in `blockMeshDict` (increase x cells from 180 to 360).

-   Reduce snappy refinement level near geometry.

-   Turn off `addLayers` initially.

* * * * *

Task 3 --- set initial fields & transport properties (15 min)
-----------------------------------------------------------

Create `0/U`, `0/p`, `constant/transportProperties` using the values we discussed:

-   Use `U = (0.2 0 0)` as inlet velocity.

-   `nu = 1e-6` (we'll accept low Re; we just want the workflow).\
    Save them.

* * * * *

Task 4 --- control files (15--30 min)
----------------------------------

Put these in `system/`: `controlDict`, `fvSchemes`, `fvSolution`. Use the PISO settings I gave earlier. Important: set `endTime = 5` and `deltaT = 0.001` initially.

* * * * *

Task 5 --- run and monitor (1--2 hours depending on mesh)
------------------------------------------------------

Run:

`icoFoam > log &   # or just `icoFoam` if you want foreground
tail -f log`

What to watch:

-   Residuals for `U` and `p` should fall; they will oscillate but should not diverge.

-   If solver diverges: reduce `deltaT` by factor 2; change `divSchemes` to more stable options (e.g., `Gauss upwind`); check `checkMesh` again.

**Deliverable 2 (send to me):**

-   The `log` tail showing the solver running for at least a few seconds of simulation time.

-   A note: max Courant number (CFL). If you don't have a functionObject for courant, report `deltaT` and approximate smallest `dx` from checkMesh.

* * * * *

Task 6 --- postprocess & plots (30--60 min)
----------------------------------------

1.  Open ParaView:

`paraFoam`

1.  Make these views:

-   Slice at mid-plane showing velocity magnitude (animated).

-   Contour of vorticity or pressure to see wake/vortices.

-   Plot `forceCoeffs` time history (if you configured the functionObjects in `controlDict`).

**Deliverable 3 (send to me):**

-   Two screenshots: (a) velocity magnitude slice showing wake, (b) forceCoeffs time series plot showing oscillatory lift (vortex shedding). Export CSV of forceCoeffs if possible.

* * * * *

Task 7 --- grid independence (1--2 sessions)
-----------------------------------------

-   Repeat with a coarser mesh (reduce blockMesh cells to `(90 12 1)`) and a finer mesh `(360 48 1)`. Keep all other settings same.

-   Run until `forceCoeffs` stabilizes and compute time-averaged drag over last N seconds (pick at least 1s of steady oscillation).

-   Produce a small table: mesh resolution | mean C_D | Std. dev C_L.

**Deliverable 4 (send to me):**

-   The table and a one-paragraph conclusion: "drag changed by X% between coarse and fine --- grid independence achieved/not achieved."

* * * * *

What I will review when you hand things in
------------------------------------------

1.  Are the mesh and `checkMesh` sensible? (no degenerate cells)

2.  Did the solver run without divergence?

3.  Do the flow fields physically make sense? (wake behind cylinder, alternating vortices)

4.  Is there grid convergence for drag coefficient?

5.  Quality of your notes: what you changed and why (short changelog).

* * * * *

Hints & troubleshooting (fast)
------------------------------

-   If `snappyHexMesh` never finishes: reduce `maxLocalCells`, decrease refinement.

-   If `icoFoam` residuals plateau high and don't drop: switch to more diffusive `divSchemes` (linearUpwind -> upwind), reduce `deltaT`.

-   If forces are noisy: you may need more sampling frequency (`writeInterval`) and a finer mesh.