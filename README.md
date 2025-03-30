# AGBT2025

[Bandage install](https://rrwick.github.io/Bandage/)

[SequnceTubeMap (online demo)](https://vgteam.github.io/sequenceTubeMap/)

[VG manual page](https://github.com/vgteam/vg/wiki/vg-manpage)

To build the container
```
singularity pull docker://docker.io/jonahcullen/agbt2025:latest
```
or download a pre-build singularity image
```
wget https://s3.msi.umn.edu/agbt2025/agbt2025_latest.sif
```

### vg toolkit
The two graph files we will use are vg and gfa formats. You can use `vg convert` to convert between formats.
```bash title="Converting files"
#Convert VG to GFA
vg convert --threads 8 --gfa-out --no-wline chr3.vg > chr3.gfa

#Convert GFA to VG
vg convert --threads 8 --gfa-in --vg-out chr3.gfa > chr3.vg
```

You can easily find coordinates of genes using NCBI's genome browser of the T2T Thoroughbred genome. For example, the MC1R gene is located on chromosome 3 at the base pair coordinates 36307397-36308540 in the new T2T Thoroughbred genome.  
https://www.ncbi.nlm.nih.gov/gdv/browser/genome/?id=GCF_041296265.1

Once you identity the region you want to extract, you can use `vg chunk` to make a subgraph of only that region. You can use any path within the graph, it does not have the be the "backbone" assembly. When you use path coordinates, the subgraph will retain variation within that region even if it is not present in the path you use.
```bash title="Extract vg subgraph"
# extract by coordinates
vg chunk --xg-name chr3.vg --threads 8 --path RefT2Tv5#0#chr3:36307397-36308540 > MC1R.coords.vg
```

Alternatively, you can extract regions by the node ids if known. For example, if you find an interesting variant in the VCF file where the nodes are listed.
```
# extract by node ids
vg chunk --xg-name chr3.vg --threads 8 --node-range 60441628:60441631 > MC1R.nodes.vg
```

Next, you can convert this subgraph back to a gfa file to view in bandage.
```
vg convert --threads 8 --gfa-out --no-wline MC1R.vg > MC1R.gfa
```

You can also use [SequnceTubeMap (online demo)](https://vgteam.github.io/sequenceTubeMap/) to visualize pangenome graph by importing the `vg` files. 

In addition to viewing the pangenome graph, you can also view alignments to the pangenome graph. In our example, we aligned whole genome short-read data to our pangenome graph using `vg giraffe` [(Manual)](https://github.com/vgteam/vg/wiki/vg-manpage#giraffe) to produce the gam (Graph Alignment/Map format, vg's BAM equivalent) files.

To visualize your gam alignment files, you must first prepare your gam files with the script provided by [SequenceTubeMap](https://github.com/vgteam/sequenceTubeMap) on their GitHub page. The script will output a .sorted.gam file in the same directory as your input file.
```bash title="Prepare GAM files for SequenceTubeMap"
./sequenceTubeMap/scripts/prepare_gam.sh sample1.giraffe.gam
```

Now you can extract regions from your graph (vg) file and sorted.gam files at the same time. Using `--context-steps 20`, we expanded the context of the chunk by 20 node steps to include the adjacent sequences on either side of our region.  
```bash title="Extract region from graph and gam files."
vg chunk --xg-name chr3.vg\
 --gam-name sample1.giraffe.sorted.gam\
 --gam-name sample2.giraffe.sorted.gam\
 --gam-name sample3.giraffe.sorted.gam\
 --gam-name sample4.giraffe.sorted.gam\
 --gam-name sample5.giraffe.sorted.gam\
 --gam-and-graph --context-steps 20 --threads 8\
 --path RefT2Tv5#0#chr3:36307397-36308540 > MC1R.vg
```

### odgi 1D visualizations
See the [odgi documentation](https://odgi.readthedocs.io/en/latest/rst/commands/odgi_viz.html) for more details and ways to customize figures.

Use the arugement `-M, --prefix-merges` to combine haplotypes or samples into one line in the output figure.



Merge lines by Sample or Haplotype
Use the arugement `-M, --prefix-merges` to combine haplotypes or samples into one line in the output figure for any 1D visual. 

```
odgi viz --idx file.og --out out.VIZ.id.png --color-by-prefix '#' --threads 2 --prefix-merges IDs.txt
```
```
odgi viz --idx file.og --out out.VIZ.hap.png --color-by-prefix '#' --threads 2 --prefix-merges Haplotype_IDs.txt
```

Color by mean depth
```
odgi viz --idx file.og --out out.depth.png --color-by-mean-depth --colorbrewer-palette Spectral:4 --threads 2
```

Color by strandedness (-z)
First, orient all sequences in direction of reference path, or any path you choose.

```
odgi flip --idx file.og --out file.flip.og --threads 4
odgi viz --idx file.flip.og --out out.strand.png --color-by-mean-inversion-rate --threads 2
```


Summarize Path Coverage
```
odgi viz --idx $ogi --out out.sum.png -path-x-padding 500 --compressed-mode -threads 2
```
