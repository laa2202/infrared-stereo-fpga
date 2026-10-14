# FPGA Infrared Stereo Tracking & 3D Localization

**Embedded systems · FPGA / SystemVerilog · C · Stereo computer vision · Python**

A dual-camera infrared-beacon tracking system combining FPGA image processing with C-based stereo reconstruction and Python visualization.


## Architecture

```text
Camera A ── pixel clock/data ─┐
                             ├─ FPGA frame/pixel decoding → thresholding
Camera B ── pixel clock/data ─┘       → foreground image moments
                                         (area, weighted X, weighted Y)
                                                 │
                                            FPGA / HPS interface
                                                 │
                              ARM-side C: centroid extraction + calibration
                                                 │
                                  stereo triangulation → (X, Y, Z)
                                                 │
                                       Python 3D trajectory plot
```

## What the source implements

- **Pixel-stream decoding** and coordinates from camera sync signals (`hardware/pixel_coordinate_decoder.sv`).
- **Configurable per-frame thresholding and moment accumulation** for detected foreground pixels (`hardware/camera_moment_pipeline.sv`, `hardware/moment_accumulators.sv`).
- **SoC top-level integration and Platform Designer project metadata** (`hardware/soc_system_top.sv`, `hardware/soc_system.qsys`).
- **Camera configuration and capture, stereo calibration, and 3D triangulation** in C (`software/src/` and `software/include/`).
- **Live 3D plot** that reads `xyz=` coordinate reports from a serial log (`software/tools/live_xyz_plot.py`).

## Repository structure

```text
hardware/              FPGA SystemVerilog pipeline, SoC top-level, Quartus/Platform Designer files
software/include/      C headers for camera configuration, geometry, calibration
software/src/          ARM-side C application, image correspondence and reconstruction
software/tools/        Host visualization and serial tools
software/Makefile      C build rules
```

## Build notes

**Software:** From `software/`, run `make` with a suitable compiler/toolchain. For ARM target builds, set `CROSS_COMPILE` as appropriate for your toolchain; the Makefile also supports host compilation for portions of the code. The complete application expects platform-specific FPGA camera registers.

**Hardware:** The source targets Intel/Altera Cyclone V SoC tooling, requiring compatible Quartus/Platform Designer software and corresponding development hardware. The hardware build requires generated vendor IP components and a board-specific clock, pin, and camera configuration.

**Visualization:** Install `matplotlib`, produce a `screenlog.0` containing application output lines with `xyz=X,Y,Z`, then run:

```bash
python3 -m pip install matplotlib
python3 software/tools/live_xyz_plot.py
```
