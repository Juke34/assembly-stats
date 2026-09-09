# Web page with assembly metric visualisations to facilitate rapid assessment and comparison of assembly quality.   

Demo:  https://juke34.github.io/assembly-stats-easy/

## Table of content

- [Short instructions for the impatients (quick start)](#short-instructions-for-the-impatients-quick-start)
- [Description](#description)
- [Plot description](#plot-descritption)
- [Generate statistics](#generate-statistics)
- [The json input format](#the-json-input-format)
- [Local Visualization](#local-visualization)
- [Automatic Deployment with GitHub Actions](#automatic-deployment-with-github-actions)
- [Acknowledgements](#acknowledgements)

## Short instructions for the impatients (quick start)

1) Fork this repository  
2) Install pixi
```bash
curl -fsSL https://pixi.sh/install.sh | bash
```
3) Install dependencies
```
pixi install
```
4) Compute statistics on your genome assembly file (e.g., `genome_assembly.fa`).
```bash
pixi run asm2stats genome_assembly.fa > json/output.assembly-stats.json
```
5) Put as many JSON output files as needed into the `json/` directory. Extension must follow the pattern `.assembly-stats.json`. Prefix with a custom name to distinguish between different assemblies.
6) Configure github pages (`Settings` > `Pages`) to work from `GitHub Actions`
7) Commit and push your changes to your forked repository.
8) Visit your GitHub Pages site to view the assembly metric visualisations at this URL: https://<your-github-username>.github.io/assembly-stats/

## Description

A _de novo_ genome assembly can be summarised by a number of metrics, including:
- Overall assembly length
- Number of scaffolds/contigs
- Length of longest scaffold/contig
- Scaffold/contig N50 and N90
- Assembly base composition, in particular percentage GC and percentage Ns
- CEGMA completeness
- Scaffold/contig length/count distribution

assembly-stats supports two widely used presentations of these values, tabular and cumulative length plots, and introduces an additional circular plot that summarises most commonly used assembly metrics in a single visualisation.  Each of these presentations is generated using javascript from a common (JSON) data structure, allowing toggling between alternative views, and each can be applied to a single or multiple assemblies to allow direct comparison of alternate assemblies.  

Tabular presentation allows direct comparison of exact values between assemblies, the limitations of this approach lie in the necessary omission of distributions and the challenge of interpreting ratios of values that may vary by several orders of magnitude.

![Screenshot](/screenshots/table.png "Table view")

Cumulative scaffold length plots are highly effective for comparison of two or more assemblies, plotting both on a single set of axes reveals differences in assembled size and the N50 count very clearly. However, other metrics must still be tabulated or annotated on the plot for example N50 length and the longest scaffold length can be particularly difficult to determine from the plot alone. The scale for the axes is usually chosen to accommodate the data for a single assembly or set of assemblies, meaning that it is usually necessary to replot the data or consider the relative axis scales carefully to compare assemblies that have been plotted separately. The cumulative distribution plots in assembly-stats address the problem of scaling by allowing any combination of assemblies to be plotted together and allowing rescaling of the axes to fit any one of the individual assemblies.

![Screenshot](/screenshots/cumulative.png "Cumulative view")

The circular plots have been introduced to overcome some of the shortcomings of tabular and cumulative distribution plots in a visualisation that allows rapid assessment of most common assembly metrics. The graphic is essentially scale independent so assemblies of any size with different strengths and weaknesses produce distinct patterns that can be recognised at a glance. While side by side presentation of a pair of assemblies on consistently scaled axes allows direct comparison, the standard presentation is designed to facilitate assessment of overall assembly quality by consideration of the keys features from the plot.

![Screenshot](/screenshots/assembly_stats.png "Circle view")


## plot descritption
- click on any colour tile in the legend to toggle visibility of that feature on/off
- The inner radius of the circular plot represents the length of the longest scaffold in the assembly
- The angle subtended by the first (red) segment within this plot indicates the percentage of the assembly that is in the longest scaffold
- The radial axis originates at the circumference and indicates scaffold length
- Subsequent (grey) segments are plotted from the circumference and the length of segment at a given percentage indicates the cumulative percentage of the assembly that is contained within scaffolds of at least that length
- The N50 and N90 scaffold lengths are indicated respectively by dark and light orange arcs that connect to the radial axis for ease of comparison
- The cumulative number of scaffolds within a given percentge of the genome is plotted in purple originating at the centre of the plot
- White scale lines are drawn at successive orders of magnitude from 10 scaffolds onwards
- The fill colour of the circumferential axis indicates the percentage base composition of the assembly: AT = light blue; GC = dark blue; N = grey
- Contig length (if available) is indicated by darker grey segments overlaying the scaffold length plot
- Contig count (if available) may be toggled on to be shown in place of the scaffold count plot
- Complete, fragmented and duplicated BUSCO genes (if available) are shown in mid, light and dark green, respectively in the smaller plot in the upper right corner
- Partial and complete CEGMA values (if available) are shown in light and dark green, respectively in the smaller plot in the upper right corner

## Generate statistics

Data to be plotted must be supplied as a JSON format object.   
The simplest way to generate this is using the ``asm2stats.pl`` perl script in the ``pl`` folder. 
The scripts depend on the Perl `JSON` and `List::Util` modules. If you have the required dependencies installed, you can run the script directly as shown below otherwise see the `PIXI` section:

```bash
perl asm2stats.pl genome_assembly.fa > output.json
```

By default each bin reports a single GC/N percentage. Pass the ``--minmaxgc``/``-m`` option to also report the min/max/mean GC and N content per bin (computed per-sequence within each bin):

```bash
perl asm2stats.pl --minmaxgc genome_assembly.fa > output.minmaxgc.json
```

Run ``perl asm2stats.pl --help`` for the full list of options.

### pixi

Instead of installing Perl and these modules manually, you can use [pixi](https://pixi.sh) to manage the environment. The repo ships a `pixi.toml` at the root that declares these dependencies.

- Install pixi (see [pixi.sh](https://pixi.sh) for platform instructions)
```bash
# Install pixi
curl -fsSL https://pixi.sh/install.sh | bash
# Install dependencies
pixi install
```
- From the repo root, run the script through pixi, which will download the required Perl environment on first use:

```bash
pixi run asm2stats json/genome_assembly.fa > json/output.assembly-stats.json
pixi run asm2stats --minmaxgc json/genome_assembly.fa > json/output.assembly-stats.minmaxgc.json
```

- Alternatively, drop into a shell with the environment activated and call the script directly:

```bash
pixi shell
perl pl/asm2stats.pl genome_assembly.fa > output.json
perl pl/asm2stats.pl --minmaxgc genome_assembly.fa > output.minmaxgc.json
```

## The json input format
 
The json object contains the following keys:
- ``assembly`` - the total assembly span
- ``ATGC`` - the assembly span without Ns (redundant if ``N`` is specified)
- ``GC`` - the GC percentage of the assembly
- ``N`` - the total number of Ns (redundant if ``ATGC`` is specified)
- ``scaffold_count`` - the total number of scaffolds in the assembly
- ``scaffolds`` - an array of scaffold lengths (only the longest scaffold is needed if ``binned_scaffold_lengths`` and ``binned_scaffold_counts`` are specified)
- ``binned_scaffold_lengths`` - an array of 1000 scaffold lengths representing the N0.1 to N100 scaffold lengths for the assembly
- ``binned_scaffold_counts`` - an array of 1000 scaffold counts representing the N0.1 to N100 scaffold numbers for the assembly
- ``contig_count`` - (optional) the total number of contigs in the assembly
- ``contigs`` - (optional) an array of contig lengths (only the longest contig is needed if ``binned_contig_lengths`` and ``binned_contig_counts`` are specified)
- ``binned_contig_lengths`` - (optional) an array of 1000 contig lengths representing the N0.1 to N100 contig lengths for the assembly
- ``binned_contig_counts`` - (optional) an array of 1000 contig counts representing the N0.1 to N100 contig numbers for the assembly
- ``binned_Ns`` - (optional) an array of 1000 values representing the N content of each bin based on size-sorted scaffold sequences
- ``binned_GCs`` - (optional) an array of 1000 values representing the GC content of each bin based on size-sorted scaffold sequences


Additional data will be plotted, if added to the stats object including:
- CEGMA scores

  ```json
  cegma_complete:    83.87,
  cegma_partial:    95.16
  ```

- BUSCO complete, duplicated, fragmented, missing and number of genes (will be plotted in place of CEGMA if both are present)

  ```json
  busco: { C:87.1,
           D:3.6,
           F:10.1,
           M:2.8,
           n:2675 }
  ```

While the plots were conceived as scale independent visualisations, there are occasions when it is useful to compare assemblies on the same radial (longest scaffold) or circumferential (assembly span) scales.  These scales may be modified on the plot by clicking the grey boxes under the scale heading.  Plots can also be drawn with an specific scale by supplying additional arguments to ``drawPlot()``.

For example to scale the radius to 10 Mb and the circumference to 400 Mb (values smaller than the default will be ignored):

```javascript
  asm.drawPlot('assembly_stats',10000000,400000000);
```

It is also possible to programmatically toggle the visibility of plot features by passing an array of classnames to ``toggleVisible()``:

```javascript
  asm.toggleVisible(['asm-longest_pie','asm-count']);
```
## Local Visualization

In order to visualize the assembly statistics locally, you can start a simple HTTP server:

```bash
python3 -m http.server 8000
```

and then access the index.html page in your web browser at `http://localhost:8000/index.html`.

## Automatic Deployment with GitHub Actions

The repository includes a [GitHub Actions workflow](/.github/workflows/deploy-pages.yml) that automatically deploys the site to GitHub Pages every time a change is pushed to the `master` branch (it can also be triggered manually from the `Actions` tab).

To enable it:
- Go to `Settings` > `Pages`
- Under **Source**, select `GitHub Actions`
- Push to `master` (or run the workflow manually) and the site will be built and published automatically


## Acknowledgements

@rjchallis for originally developing the assembly-stats visualization framework. (https://github.com/rjchallis/assembly-stats)  
@ammaraziz for contributions to the assembly-stats visualization framework. (https://github.com/ammaraziz/assembly-stats)