# Python package for MEGAN CE using the blast2lca tool

![DOIzendoBadge](https://zenodo.org/badge/76442470.svg)

This is a collection of python classes for working with with MEGAN6 CE (v6.6.0). Now supports both GI and acc

## Usage

```
fullPipeline <rootDirectory> <sampleDirectory> <sampleName> <inputFile.m8> <taxOutput> <koOutput> --blast2lca <path to blast2lca tool> --gi2kegg <gi2kegg or acc2kegg> --gi2taxid <gi2taxid of acc2taxid>
```

`fullPipeline` processes meganised DAA files as input into a
tabular output where each query (DIAMOND) is given its constituent KEGG and NCBI taxonomy assignment.

#### required Directory structure

#### input

```
/root/directory
└── sampleDirectory/
     └── inputFile.daa / inputFile.m8
```

#### Output

```
/root/directory
└── sampleDirectory/
     ├── KOoutput
     ├── taxoutput
     └── inputFile.daa / inputFile.m8
```

## Others

### Docker

Running the image like how you would a binary/executable
to convert the m8 diamond outputs into KO and TAXID tables

The image comes installed with MEGAN CE

If you’re using the older NCBI NR database used the `nr_gi_0_1` tagged docker image.

```
docker pull etheleon/blast2lca:nr_gi_0_1
```

#### Example: Running the dockerised blast2lca executable

```
WORKDIR=/path/2/your/workdir
DIAMONDOUTPUT=/path/2/query.m8
GI2TAXID=/export2/home/uesu/simulation_fr_the_beginning/data/classifier/gi2taxid.refseq.map
GI2KEGG=/export2/home/uesu/github/MEGAN/tools/gi2kegg.map

docker run --rm \
   -v $WORKDIR:/data \
   -v $DIAMONDOUTPUT:/data/diamond.m8 \
   -v $GI2KEGG:/data/gi2kegg \
   -v $GI2TAXID:/data/gi2taxid \
    etheleon/blast2lca:nr_gi_0_1
```

for the new acc based NCBI NR database use the `nr_acc_0_1` tagged docker image

```
docker pull etheleon/blast2lca:acc_gi_0_1
```

`$GI2TAXID` will point to the `acc2taxid` mapping file same for `$GI2KEGG`

### parseMEGAN

If you already have the outputs from blast2lca, and you want to combine the annotations (ko and taxonomy for analysis)

```
usage: parseMEGAN [-h] [--verbose] root sampledir sample taxonomy kegg

Command line tool for processing blast2lca outputs

positional arguments:
  root        the root directory
  sampledir   relative path from root directory to sample directory
  sample      sample name, could be same name as sample directory
  taxonomy    blast2lca taxonomy output filename - has to be in taxIDs d__2
  kegg        blast2lca ko output filename

optional arguments:
  -h, --help  show this help message and exit
  --verbose   to switch on verbose mode
```

Accepts `.bz2` files

```bash
./parseMEGAN $PWD \ 
	tests/trimmed/NUSM01AD00_M01_1_Day0 \
	NUSM01AD00_M01_1_Day0 \
	taxoutput.bz2 \
	KOoutput.bz2
```


#### input

```
/root/directory
└── sampleDirectory/
     ├── KOoutput
     └── taxoutput
```

#### Output

```
/root/directory
└── sampleDirectory/
     ├── sampleName-combined.txt
     ├── KOoutput
     └── taxoutput
```

Each query may have multiple KEGG annotations (top 10% of subjects in a blastx query may be mapped to subject GIs mapping to multiple KOs). In this case the combination parser only takes we just the 1st KO

## Incorporating python class into your python script

### Initialize

```python
from MEGAN.process import Parser
lcaparser = Parser(
    "/export2/home/uesu/mouseData/",#rootPath
    "data/trimmed/NUSM01AD00_M01_1_Day0/", #sampledirectory
    "NUSM01AD00_M01_1_Day0", #sampleName
    "KOoutput",#kofile
    "taxoutput3"#taxfile
)
```

### Input files

__NOTE__: The readID itself should not include any semi-colon character, if there is please remove it before running the parser.

#### KEGG
```
HISEQ:327:HN35KBCXX:2:2216:17590:101139/2; ; [1] K16363: 100 # 1
HISEQ:327:HN35KBCXX:2:2216:17505:101147/2; ;
HISEQ:327:HN35KBCXX:2:2216:18101:101081/2; ; [1] K16053: 100 # 1
HISEQ:327:HN35KBCXX:2:2216:18469:101047/2; ;
HISEQ:327:HN35KBCXX:2:2216:18275:101110/2; ; [1] K02621: 100 # 1
HISEQ:327:HN35KBCXX:2:2216:19012:101012/2; ;
HISEQ:327:HN35KBCXX:2:2216:19208:101184/2; ;
HISEQ:327:HN35KBCXX:2:2216:20173:101054/2; ;
HISEQ:327:HN35KBCXX:2:2216:20417:101000/2; ; [1] K02837: 100 # 1
HISEQ:327:HN35KBCXX:2:2216:20656:101185/2; ;
```

#### Taxonomy

```
HISEQ:327:HN35KBCXX:2:1101:17405:2046/1; ;d__2; 100;p__976; 100;c__200643; 100;o__171549; 100;f__171552; 80;g__838; 80;s__1262917; 20;
HISEQ:327:HN35KBCXX:2:1101:20056:2050/1; ;d__2; 100;p__1239; 100;c__186801; 100;o__186802; 100;f__186803; 76;g__189330; 40;s__1263073; 4;
HISEQ:327:HN35KBCXX:2:1101:19475:2499/1; ;d__2; 100;p__1239; 100;c__91061; 92;o__186826; 92;f__81852; 92;g__1350; 92;s__1351; 92;
HISEQ:327:HN35KBCXX:2:1101:16910:2803/1; ;d__2; 100;p__976; 100;c__200643; 100;o__171549; 100;f__815; 33;g__816; 33;s__1410607; 33;
HISEQ:327:HN35KBCXX:2:1101:20670:2813/1; ;d__2; 100;p__1239; 100;c__186801; 100;o__186802; 100;f__186803; 75;g__572511; 25;s__1226324; 13;
HISEQ:327:HN35KBCXX:2:1101:5294:3036/1; ;d__2; 100;p__976; 100;c__200643; 100;o__171549; 100;f__171552; 100;g__838; 100;s__1263102; 50;
HISEQ:327:HN35KBCXX:2:1101:10938:3042/1; ;d__2; 100;p__976; 100;c__200643; 100;o__171549; 100;f__171552; 100;g__838; 67;s__52227; 67;
HISEQ:327:HN35KBCXX:2:1101:7569:5698/1; ;d__2; 59;p__1239; 33;c__186801; 33;o__186802; 33;f__541000; 22;g__1263; 19;s__40519; 7;
HISEQ:327:HN35KBCXX:2:1101:11823:5595/1; ;d__2; 100;p__1239; 92;c__186801; 68;o__186802; 68;f__31979; 20;g__1485; 20;s__1262806; 4;
HISEQ:327:HN35KBCXX:2:1101:7035:5916/1; ;d__2; 100;p__976; 100;c__200643; 100;o__171549; 100;f__171551; 100;g__283168; 100;s__1263090; 50;
```


### `.megan` summary format



```python
lcaparser.singleComparison()
```

### Combined format

We count reads which have the same taxons and KOs annotations

```python
lcaparser.combined()
```

```
phylum  67820   K00000  4
phylum  1224    K06937  1
phylum  1224    K00656  6
phylum  1224    K04564  2
phylum  1224    K06934  1
phylum  1224    K12524  24
phylum  1224    K00558  7
phylum  1224    K02674  1
phylum  1224    K06694  1
phylum  1224    K01785  12
...

...
species 1262910 K00033  1
species 1262911 K00000  525
species 35760   K00000  14
species 7462    K15421  1
species 7462    K00000  1
species 1262918 K00000  365
species 1262919 K02429  5
species 1262919 K07133  1
species 1262919 K03800  1
species 1262919 K00000  529
```

## Misc

Mapping tool generators, in the `tools` folder of the blast2lcaPlus repo, you’ll find the 3 folders:

1. binaries
2. gi2kegg
3. ref2kegg

The map generators are written in [golang](https://golang.org/), and the compiled versions are found in the `binaries` folder. 
The `gi2kegg` binary is meant for older NR databases before September 2016 when genebank identifiers (GI) are still supported by NCBI

The newer `ref2kegg` binary is meant for the newer NR databases after Sept 2016.

## gi2kegg 

In the example below, only `gi|489223532|ref|WP_003131952.1|` will be displayed as the subject when you carry out alignment against the NR database, but in the linker files from KEGG link kegg genes to the sub GIs

```
K00001|contig00001      gi|489223532|ref|WP_003131952.1|        81.8    390     71      0       1202    33      1       390     1.4e-185        656.8
```

> above record is not accurate

```
>gi|489223532|ref|WP_003131952.1| 30S ribosomal protein S18 [Lactococcus lactis]gi|15674171|ref|NP_268346.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis Il1403]gi|116513137|ref|YP_812044.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris SK11]gi|125625229|ref|YP_001033712.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris MG1363]gi|281492845|ref|YP_003354825.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis KF147]gi|385831755|ref|YP_005869568.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis CV56]gi|385839508|ref|YP_005877138.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris A76]gi|389855617|ref|YP_006357861.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris NZ9000]gi|414075194|ref|YP_007000411.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris UC509.9]gi|459286377|ref|YP_007509482.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis IO-1]gi|544395586|ref|YP_008569870.1| ribosomal protein S18 RpsR [Lactococcus lactis subsp. cremoris KW2]gi|554464728|ref|YP_008703967.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis KLDS 4.0325]gi|13878750|sp|Q9CDN0.1|RS18_LACLA RecName: Full=30S ribosomal protein S18 [Lactococcus lactis subsp. lactis Il1403]gi|122939895|sp|Q02VU1.1|RS18_LACLS RecName: Full=30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris SK11]gi|166220956|sp|A2RNZ2.1|RS18_LACLM RecName: Full=30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris MG1363]gi|12725253|gb|AAK06287.1|AE006448_5 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis Il1403]gi|116108791|gb|ABJ73931.1| SSU ribosomal protein S18P [Lactococcus lactis subsp. cremoris SK11]gi|124494037|emb|CAL99037.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris MG1363]gi|281376497|gb|ADA65983.1| SSU ribosomal protein S18P [Lactococcus lactis subsp. lactis KF147]gi|300072039|gb|ADJ61439.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris NZ9000]gi|326407763|gb|ADZ64834.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis CV56]gi|354692797|gb|EHE92602.1| hypothetical protein LLCRE1631_01913 [Lactococcus lactis subsp. lactis CNCM I-1631]gi|358750736|gb|AEU41715.1| SSU ribosomal protein S18p [Lactococcus lactis subsp. cremoris A76]gi|374674265|dbj|BAL52156.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis IO-1]gi|413975114|gb|AFW92578.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris UC509.9]gi|525225771|emb|CDG05746.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis A12]gi|530789293|gb|EQC53187.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis bv. diacetylactis str. TIFN4]gi|530789525|gb|EQC53393.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis bv. diacetylactis str. TIFN2]gi|530791209|gb|EQC54683.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris TIFN6]gi|530793883|gb|EQC56744.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris TIFN5]gi|530859245|gb|EQC82878.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris TIFN7]gi|530868587|gb|EQC91162.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris TIFN1]gi|530872454|gb|EQC94448.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris TIFN3]gi|537110246|gb|ERE60813.1| 30S ribosomal protein S18 [Enterococcus gallinarum EGD-AAK12]gi|543873771|gb|AGV74185.1| ribosomal protein S18 RpsR [Lactococcus lactis subsp. cremoris KW2]gi|552065042|gb|AGY45032.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis KLDS 4.0325]gi|554892214|gb|ESK79551.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis bv. diacetylactis str. LD61]gi|666395058|gb|KEY61992.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. cremoris GE214]gi|669187288|gb|AII13743.1| 30S ribosomal protein S18 [Lactococcus lactis subsp. lactis NCDO 2118]
MAQQRRGGFKRRKKVDFIAANKIEVVDYKDTELLKRFISERGKILPRRVTGTSAKNQRKVVNAIKRARVMALLPFVAEDQ
```

## ref2kegg

### Usage

```
KEGG=/path/to/your/KEGG/FTP
./ref2kegg nr $KEGG/genes/links/genes_ncbi-proteinid.list $KEGG/genes/links/genes_ko.list > acc2ko.map
```
