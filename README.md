# running_bitseq
[manual](http://bitseq.github.io/)

#install
git clone https://github.com/BitSeq/BitSeq.git

cd Bitseq

make

#Estimate expression and variance
for file in *sam

do

BitSeq/parseAlignment $file -o $(basename $file .sam).prob --trSeqFile Anopheles-coluzzii-Mali-NIH_TRANSCRIPTS_AcolM1.2.fa --trInfoFile $(basename $file .sam).tr --uniform --verbose

BitSeq/estimateExpression $(basename $file .sam).prob -o $(basename $file .sam) --outType RPKM -p parameters1.txt -t $(basename $file .sam).tr -P 2

BitSeq/getVariance -o $(basename $file .sam).mean $(basename $file .sam).rpkm

done


#differential expression
>compute overall mean and variance for each transcript

BitSeq/getVariance --log -o experiment_name.Lmean \*.rpkm 

#estimate expression dependent hyperparameters
BitSeq/estimateHyperPar --meanFile experiment\_name.Lmean -o experiment_name.param Mopti-C\*rpkm C Mopti-T\*rpkm

#estimate differential expression
BitSeq/estimateDE -o Mopti -p experiment_name.param Mopti-C\*rpkm C Mopti-T\*rpkm

#You can run the above commands sequentially with a __;__ like:

BitSeq/getVariance --log -o Mopti\_rmdup.Lmean 'Mopti\*rmdup.rpkm' ; BitSeq/estimateHyperPar --meanFile Mopti\_rmdup.Lmean -o Mopti\_rmdup.param Mopti-C\*rmdup.rpkm C Mopti-T\*rmdup.rpkm ; BitSeq/estimateDE -o Mopti\_rmdup -p Mopti_rmdup.param Mopti-C\*rmdup.rpkm C Mopti-T\*rmdup.rpkm


#Mopti Treatment versus cyp1 treatment
>compute overall mean and variance for each transcript

BitSeq/getVariance --log -o all.Lmean \*.rpkm 

#estimate expression dependent hyperparameters
BitSeq/estimateHyperPar --meanFile all.Lmean -o all.param Mopti-T\*rpkm C cyp1-T\*rpkm
>treatment effect

BitSeq/estimateHyperPar --meanFile all.Lmean -o all_Teffect.param \*-C\*rpkm C \*-T\*rpkm 


#estimate differential expression
BitSeq/estimateDE -o all -p all.param Mopti-T\*rpkm C cyp1-T\*rpkm
#treatment effect
BitSeq/estimateDE -o all\_Teffect -p all_Teffect.param \*-C\*rpkm C \*-T\*rpkm

BitSeq/getGeneExpression -o data1-1_Genes.rpkm -t ensemblGenes.tr -G geneFile.txt data1-1.rpkm
BitSeq/getGeneExpression -o Mopti-T3-YY\_S189\_genes.rpkm -t Mopti-T3-YY\_S189.tr -G Mopti-T3-YY\_S189\_genes.txt Mopti-T3-YY\_S189.rpkm


#Calc diff expression with "bad" samples removed

>compute overall mean and variance for each transcript

BitSeq/getVariance --log -o allgood.Lmean \*.rpkm 

#estimate expression dependent hyperparameters
BitSeq/estimateHyperPar --meanFile allgood.Lmean -o allgood.param C\*rpkm C T\*rpkm


#estimate differential expression
BitSeq/estimateDE -o treatment_E -p allgood.param C\*rpkm C T\*rpkm

#sequenctially
../BitSeq/getVariance --log -o allgood.Lmean \*.rpkm ; ../BitSeq/estimateHyperPar --meanFile allgood.Lmean -o allgood.param \*C\*rpkm C \*T\*rpkm ; ../BitSeq/estimateDE -o treatment_E -p allgood.param \*C\*rpkm C \*T\*rpkm


#Run bitseq with variational Bayes (VB) for gene expression only (faster)
>assumes we already ran BitSeq/parseAlignment

mkdir VB

>link in .prob files

for file in \*.prob

do

echo "BitSeq/estimateVBExpression -o $(basename $file .prob) $file"

done

\> runVB


