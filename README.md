# YFISH analysis example

<!-- MarkdownTOC -->

- [Introduction](#introduction)
- [Requirements](#requirements)
- [Input](#input)
- [Tutorial](#tutorial)

<!-- /MarkdownTOC -->

## Introduction

Originally analyzed with the Python package `pygpseq` ([here](https://github.com/ggirelli/pygpseq)), nowadays we process YFISH images using the package `radiankit` ([here](https://github.com/ggirelli/radiantkit), still under development).

## Requirements

The tutorial relies on `radiantkit v0.0.1.20210618.1`. The recommended installation procedure utilizes `pipx`. Follow [these instructions](https://github.com/pypa/pipx#install-pipx) to install `pipx`. Once `pipx` is available on your system, install the proper version of `radiantkit` with:

`pipx install git+https://github.com/ggirelli/radiantkit.git@v0.0.1.20210618.1 --force`

## Input

Create a folder for the tutorial.

```bash
mkdir $HOME/yfish-tutorial
cd $HOME/yfish-tutorial
```

Starting data are available for download at [this link](http://bicroserver-2.scilifelab.se:5000/sharing/J2vF8UBhc) (accessible only from within SciLifeLab network). Place the downloaded nd2 file in the tutorial folder.

## Tutorial

1. Convert `nd2` file to `tiff` images.

```bash
cd $HOME/yfish-tutorial
radiant nd2_to_tiff iTK286_040519_001.nd2
```

2. Identify any out of focus fields of view.

```bash
cd $HOME/yfish-tutorial/iTK286_040519_001
radiant tiff_findoof . --threads 5 --rename --inreg '^dapi.*\.tiff$'
```

The script automatically renames all DNA staining channels (`dapi` in this tutorial) that are out-of-focus by adding the `.old` suffix. The user needs to add the same suffix *manually* to the corresponding fields in other channels.

```bash
mv cy5_005.tiff cy5_005.tiff.old
mv cy5_009.tiff cy5_009.tiff.old
mv cy5_011.tiff cy5_011.tiff.old
mv cy5_019.tiff cy5_019.tiff.old
```

3. Segment the DNA staining to identify nuclei.

```bash
cd $HOME/yfish-tutorial/iTK286_040519_001
radiant tiff_segment . --TCZYX --threads 5 --inreg "^dapi.*\.tiff$" -y
```

4. Measure nuclei.

```bash
cd $HOME/yfish-tutorial/iTK286_040519_001
radiant measure_objects . dapi --threads 5 -y
```

5. Select G1 nuclei.

```bash
cd $HOME/yfish-tutorial/iTK286_040519_001
radiant select_nuclei --k-sigma 2 --threads 5 . dapi -y
rm *mask.tiff # remove original masks
```

6. Build radial profiles.

```bash
cd $HOME/yfish-tutorial/iTK286_040519_001
radiant radial_population --aspect 200 130 130 --mask-suffix mask_selected --threads 5 . dapi -y
```

*NOTE. The radial profile building step might take several minutes to complete.*

7. Generate html report.

```bash
cd $HOME/yfish-tutorial
radiant report .
```

Finally, inspect the generated html report with your favorite browser.