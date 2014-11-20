moira
=====

Quality-filter raw sequence reads using the Poisson binomial filtering algorithm.

REQUIREMENTS:

- Expects quality scores in Sanger/Illumina1.8+ format.
- Expects that input sequences (single or paired) and qualities are in the same order.
- Expects that sequences and qualities are stored only in one line (i.e. >header\\nsequence\\n>header2\\nsequence2)
- OPTIONAL: Requires numpy if --qmode is set to "bootstrap".
- OPTIONAL: Requires the bernoulli library, which includes the C implementation of the Poisson binomial filtering algorithm.
  To obtain it, the file bernoullimodule.c must be compiled as a shared library and placed in the same folder as 
  this script (or into the python library folder). If not available, the script will automaticaly switch to the pure
  python implementation.
- OPTIONAL: Requires the nw_align library, which includes the C implementation of the Needleman-Wunsch alignment algorithm.
  The nw_align library is writen in Cython. Use Cython to translate the code to C, and then compile it as a shared library.
  If not present, the script will automatically switch to the pure python implementation.


USAGE:

  - Make contigs from paired reads without quality-filtering:

        moira.py --forward_fasta=<FILE> --forward_qual=<FILE> --reverse_fasta=<FILE> --reverse_qual=<FILE> --paired --only_contig

  - Make contigs from paired reads and perform quality-filtering:

        moira.py --forward_fasta=<FILE> --forward_qual=<FILE> --reverse_fasta=<FILE> --reverse_qual=<FILE> --paired

  - Quality-filter already assembled contigs or single reads:

        moira.py --forward_fasta=<FILE> --forward_qual=<FILE>

OUTPUT:

  - If quality control is beeing performed, files will be generated with both the sequences that passed the QC and the ones that didn't. A small report will be included on the headers of the contigs that didn't pass the QC.

        <INPUT_NAME>.qc.good.fasta
        <INPUT_NAME>.qc.good.qual
        <INPUT_NAME>.qc.bad.fasta
        <INPUT_NAME>.qc.bad.qual

  - Else, only two files will be generated.

        <INPUT_NAME>.contigs.fasta
        <INPUT_NAME>.contigs.qual

  - If identical sequences are being collapsed, mothur-formatted name files will also be generated.


PARAMETERS:

  - Needleman-Wunsch aligner parameters:
    --match (default 1): match score
--gap (default -2): gap penalty
--mismatch (default -1): mismatch penalty

  - Contig constructor parameters:
 
      --insert (default 20): quality above which a base will be used for filling a complementary gap or ambiguity.
      --deltaq (default 6): minimum quality difference allowed between two mismatched bases for not including an N in the consensus sequence.
      --consensus_qscore (default 'best')
        best: use the best quality on each position of the alignment as the consensus quality score (Unless an ambiguity is introduced in that position by the contig constructor. In that case, quality score will be always 2).
        sum: in matching bases, consensus quality score will be the sum of the qualities of both reads in that position of the alignment.

  - Quality-filtering parameters:

                --collapse (default True): if True, identical sequences will be collapsed before quality control, and the one with
                        the best quality will be used as a representative of the whole group.

                --error_calc (default 'poisson_binomial'): algorithm used for error calculation.
                        poisson_binomial: calculate the Poisson binomial distribution (sum of bernoulli random variables).
                        poisson: approximating sum of bernoulli random variables to a poisson distribution.
                        bootstrap: numerical generation of an error distribution (deprecated).

                --ambigs (default treat_as_error): handling of ambiguous positions during quality checking.
                        treat_as_error: will consider than ambiguities always result in a misread base.
                        disallow: will discard sequences with ambiguities.
                        ignore: will ignore ambiguities.

                --uncert (default 0.01): Maximum divergence of the observed sequence from the original one due to sequencing errors.

                --maxerrors (no default value): Maximum errors allowed in the sequence.
                        Will be override --uncert if specified as a parameter.

                --alpha (default 0.005): Probability of underestimating the actual errors of a sequence.

                --bootstrap (default 100): Number of replicates per position used for error calculation by the bootstrap method.
        
        - Other:

                --paired: input files are paired end files and will be assembled into contigs.
                --only_contigs: assemble contigs, don\'t do quality control.
                --relabel (default False): if a prefix string is introduced, sequential labels will be generated for the sequences,
                                           with the format <prefix>N, where N=1,2,3 etc.
                --output_format (default fasta):
                        fasta: output files in fasta + qual format.
                        fastq: output files in fastq format.
                --pipeline (default mothur):
                        mothur: output for collapsed sequences will be in mothur\'s fasta + names format.
                        USEARCH: output for collapsed sequences will be in a single fasta file, with abundance information stored in the sequence header.
                --processors (default 1): number of processes to use.

COMMENTS:

        - Alignment parameters are set to replicate mothur's default implementation of the Needleman-Wunsch algorithm.

        - The 'insert' and 'deltaq' parameters from mothur make.contig are also reproduced. They are set at their default values.
          More details can be found at www.mothur.org/wiki/Make.contigs

        - Approximating the sum of bernoulli random variables to a poisson distribution is quicker than calculating 
          their exact sum (Poisson binomial distribution). That said, the Poisson binomial filtering algorithm is also implemented
          in C and even the python implementation is quick enough for processing large datasets. The bootstrap method
          (--error_calc bootstrap) is a numerical algorithm for performing the sum of bernoulli random variables.
          It is only included for testing purposes.

        - Quality-filtering will discard the contigs expected to have more than 'alpha' chances of diverging from the original 
          sequence more than the value specified by the 'uncert' param. That means that, during distance calculation between two
          given sequences, the observed distance will be at most 'dist + 2*uncert', where 'dist' is the original distance between
          those sequences without sequencing errors. Thus, a good rule of thumb would be considering the effective OTU clustering 
          distance to be actually 'OTUdist - 2*uncert', where OTUdist is the distance used for clustering the observed sequences.


Distributed under the GNU General Public License.
