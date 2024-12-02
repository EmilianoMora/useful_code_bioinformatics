Install and use scikit-allel

```
conda activate emiliano3.6
conda install -c conda-forge scikit-allel
```

Go through a VCF file and do a Site Frequency Spectrum. Scikit-allel can only be run through python.
```
python
import allel #this is to open scikit-allel
s = allel.read_vcf("/home/emiliano/Desktop/4fold_sites_EMC1.vcf.gz", fields='*') #Reads the VCF file. It can be compressed (*.vcf.gz) or uncompressed (*.vcf).
# s = allel.read_vcf("/home/emiliano/Desktop/4fold_sites_EMC1.vcf.gz", fields='*', region='pve_haplotypeT_001') #Reads specific chromosome or genomic region. Can add coordinates in Tabix format. A Tabix file is needed.
genotypes = s['calldata/GT'] #This function grabs the genotypes of each individual from the VCF file.
gt_array = allel.GenotypeArray(genotypes) #Transforms the table above
allele_counts = gt_array.count_alleles(max_allele = 1) #Counts the alleles per site from the array. Note that 'max_allel = 1' option does not refer to the number of alleles but to the number of axis to be included from the array itself. This means that '0' includes the first allele, '1' includes the second allele (i.e., allele 1 and 2), and so forth.
biallelic_sites = allele_counts.is_biallelic()[:] #Creates a vector indicating whether each site has two alleles or not.
ac_biallelic_derived = allele_counts.compress(biallelic_sites, axis=0)[:, 1] #Filters out the non-biallelic sites from the 'allele_counts'. The resulting file will have an array with two columns. The first and second are the ancentral and dervied allele counts, respectively. The [:, 1] in the function tells to keep the second column, meaning we keep the derived allele counts. This is needed for the unfolded-SFS but not for the folded-SFS.
ac_biallelic_derived
sfs1 = allel.sfs(ac_biallelic_derived)
sfs1
```

## Plot SFS
```
import matplotlib as mpl
import matplotlib.pyplot as plt
import seaborn as sns
fig, ax = plt.subplots(figsize=(8, 5))
sns.despine(ax=ax, offset=10)
allel.plot_sfs(sfs1, ax=ax, yscale='linear', n = 20) #'yscale' options are 'linear', 'log', 'symlog', 'logit', 'function', 'functionlog'. and 'n' is the number of chromosomes (individuals times two if diploid)
ax.legend()
ax.set_title('Site frequency spectra')
ax.set_xlabel('minor allele frequency');
plt.show() #Need to run this to plot the figure.
```

```
allele_counts = gt_array.count_alleles(max_allele = 1)
allele_counts_2 = numpy.sum(allele_counts, axis = 1)
n_fully_missing = numpy.count_nonzero(allele_counts_2 == 0)

test = allele_counts.any(axis=1)
(~test).sum()
```
