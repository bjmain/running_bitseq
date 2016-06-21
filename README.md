# running_bitseq
[manual](http://bitseq.github.io/)

#install
git clone https://github.com/BitSeq/BitSeq.git

cd Bitseq

make

#step 1
>high-throughput. I split up the for loop into 4 groups (Mopti Ccontrol, treatment, cyp1 control, treatment) to speed things up.
#Mopti
for file in Mopti-C*sam
do
BitSeq/parseAlignment $file -o $(basename $file .sam).prob --trSeqFile Anopheles-coluzzii-Mali-NIH_TRANSCRIPTS_AcolM1.2.fa --trInfoFile $(basename $file .sam).tr --uniform --verbose
BitSeq/estimateExpression $(basename $file .sam).prob -o $(basename $file .sam) --outType RPKM -p parameters1.txt -t $(basename $file .sam).tr -P 2
BitSeq/getVariance -o $(basename $file .sam).mean $(basename $file .sam).rpkm
done
for file in Mopti-T*sam
do
BitSeq/parseAlignment $file -o $(basename $file .sam).prob --trSeqFile Anopheles-coluzzii-Mali-NIH_TRANSCRIPTS_AcolM1.2.fa --trInfoFile $(basename $file .sam).tr --uniform --verbose
BitSeq/estimateExpression $(basename $file .sam).prob -o $(basename $file .sam) --outType RPKM -p parameters1.txt -t $(basename $file .sam).tr -P 2
BitSeq/getVariance -o $(basename $file .sam).mean $(basename $file .sam).rpkm
done

#cyp1
for file in cyp1-C*sam
do
BitSeq/parseAlignment $file -o $(basename $file .sam).prob --trSeqFile Anopheles-coluzzii-Mali-NIH_TRANSCRIPTS_AcolM1.2.fa --trInfoFile $(basename $file .sam).tr --uniform --verbose
BitSeq/estimateExpression $(basename $file .sam).prob -o $(basename $file .sam) --outType RPKM -p parameters1.txt -t $(basename $file .sam).tr -P 2
BitSeq/getVariance -o $(basename $file .sam).mean $(basename $file .sam).rpkm
done

for file in cyp1-T*sam
do
BitSeq/parseAlignment $file -o $(basename $file .sam).prob --trSeqFile Anopheles-coluzzii-Mali-NIH_TRANSCRIPTS_AcolM1.2.fa --trInfoFile $(basename $file .sam).tr --uniform --verbose
BitSeq/estimateExpression $(basename $file .sam).prob -o $(basename $file .sam) --outType RPKM -p parameters1.txt -t $(basename $file .sam).tr -P 2
BitSeq/getVariance -o $(basename $file .sam).mean $(basename $file .sam).rpkm
done

#redo with rmdup
for file in Mopti-C*sorted_rmdup.bam
do
BitSeq/parseAlignment $file -o $(basename $file .bam).prob --trSeqFile Anopheles-coluzzii-Mali-NIH_TRANSCRIPTS_AcolM1.2.fa --trInfoFile $(basename $file .bam).tr --uniform --verbose
BitSeq/estimateExpression $(basename $file .bam).prob -o $(basename $file .bam) --outType RPKM -p parameters1.txt -t $(basename $file .bam).tr -P 2
BitSeq/getVariance -o $(basename $file .bam).mean $(basename $file .bam).rpkm
done
for file in Mopti-T*sorted_rmdup.bam
do
BitSeq/parseAlignment $file -o $(basename $file .bam).prob --trSeqFile Anopheles-coluzzii-Mali-NIH_TRANSCRIPTS_AcolM1.2.fa --trInfoFile $(basename $file .bam).tr --uniform --verbose
BitSeq/estimateExpression $(basename $file .bam).prob -o $(basename $file .bam) --outType RPKM -p parameters1.txt -t $(basename $file .bam).tr -P 2
BitSeq/getVariance -o $(basename $file .bam).mean $(basename $file .bam).rpkm
done

#cyp1
for file in cyp1-C*sorted_rmdup.bam
do
BitSeq/parseAlignment $file -o $(basename $file .bam).prob --trSeqFile Anopheles-coluzzii-Mali-NIH_TRANSCRIPTS_AcolM1.2.fa --trInfoFile $(basename $file .bam).tr --uniform --verbose
BitSeq/estimateExpression $(basename $file .bam).prob -o $(basename $file .bam) --outType RPKM -p parameters1.txt -t $(basename $file .bam).tr -P 2
BitSeq/getVariance -o $(basename $file .bam).mean $(basename $file .bam).rpkm
done

for file in cyp1-T*sorted_rmdup.bam
do
BitSeq/parseAlignment $file -o $(basename $file .bam).prob --trSeqFile Anopheles-coluzzii-Mali-NIH_TRANSCRIPTS_AcolM1.2.fa --trInfoFile $(basename $file .bam).tr --uniform --verbose
BitSeq/estimateExpression $(basename $file .bam).prob -o $(basename $file .bam) --outType RPKM -p parameters1.txt -t $(basename $file .bam).tr -P 2
BitSeq/getVariance -o $(basename $file .bam).mean $(basename $file .bam).rpkm
done


--------
#differential expression

#compute overall mean and variance for each transcript
BitSeq/getVariance --log -o Mopti.Lmean Mopti*.rpkm 
BitSeq/getVariance --log -o cyp1.Lmean cyp1*.rpkm 

BitSeq/getVariance --log -o Mopti_rmdup.Lmean Mopti*rmdup.rpkm 
BitSeq/getVariance --log -o cyp1_rmdup.Lmean cyp1*rmdup.rpkm 

#estimate expression dependent hyperparameters
BitSeq/estimateHyperPar --meanFile Mopti.Lmean -o Mopti.param Mopti-C*rpkm C Mopti-T*rpkm
BitSeq/estimateHyperPar --meanFile cyp1.Lmean -o cyp1.param cyp1-C*rpkm C cyp1-T*rpkm

BitSeq/estimateHyperPar --meanFile Mopti_rmdup.Lmean -o Mopti_rmdup.param Mopti-C*rmdup.rpkm C Mopti-T*rmdup.rpkm
BitSeq/estimateHyperPar --meanFile cyp1_rmdup.Lmean -o cyp1_rmdup.param cyp1-C*rmdup.rpkm C cyp1-T*rmdup.rpkm

#estimate differential expression
BitSeq/estimateDE -o Mopti -p Mopti.param Mopti-C*rpkm C Mopti-T*rpkm
BitSeq/estimateDE -o cyp1 -p cyp1.param cyp1-C*rpkm C cyp1-T*rpkm

BitSeq/estimateDE -o Mopti_rmdup -p Mopti_rmdup.param Mopti-C*rmdup.rpkm C Mopti-T*rmdup.rpkm
BitSeq/estimateDE -o cyp1_rmdup -p cyp1_rmdup.param cyp1-C*rmdup.rpkm C cyp1-T*rmdup.rpkm

#run sequential commands in a screen:
BitSeq/getVariance --log -o Mopti_rmdup.Lmean Mopti*rmdup.rpkm ; BitSeq/estimateHyperPar --meanFile Mopti_rmdup.Lmean -o Mopti_rmdup.param Mopti-C*rmdup.rpkm C Mopti-T*rmdup.rpkm ; BitSeq/estimateDE -o Mopti_rmdup -p Mopti_rmdup.param Mopti-C*rmdup.rpkm C Mopti-T*rmdup.rpkm
BitSeq/getVariance --log -o cyp1_rmdup.Lmean cyp1*rmdup.rpkm ; BitSeq/estimateHyperPar --meanFile cyp1_rmdup.Lmean -o cyp1_rmdup.param cyp1-C*rmdup.rpkm C cyp1-T*rmdup.rpkm ; BitSeq/estimateDE -o cyp1_rmdup -p cyp1_rmdup.param cyp1-C*rmdup.rpkm C cyp1-T*rmdup.rpkm


------------

#Mopti Treatment versus cyp1 treatment
#compute overall mean and variance for each transcript
BitSeq/getVariance --log -o all.Lmean *.rpkm 
BitSeq/getVariance --log -o all_rmdup.Lmean *rmdup.rpkm 

#estimate expression dependent hyperparameters
BitSeq/estimateHyperPar --meanFile all.Lmean -o all.param Mopti-T*rpkm C cyp1-T*rpkm
BitSeq/estimateHyperPar --meanFile all_rmdup.Lmean -o all_rmdup.param Mopti-T*rmdup.rpkm C cyp1-T*rmdup.rpkm
  #treatment effect
  BitSeq/estimateHyperPar --meanFile all.Lmean -o all_Teffect.param *-C*rpkm C *-T*rpkm 


#estimate differential expression
BitSeq/estimateDE -o all -p all.param Mopti-T*rpkm C cyp1-T*rpkm
BitSeq/estimateDE -o all_rmdup -p all_rmdup.param Mopti-T*rmdup.rpkm C cyp1-T*rmdup.rpkm
  #treatment effect
  BitSeq/estimateDE -o all_Teffect -p all_Teffect.param *-C*rpkm C *-T*rpkm

BitSeq/getGeneExpression -o data1-1_Genes.rpkm -t ensemblGenes.tr -G geneFile.txt data1-1.rpkm
BitSeq/getGeneExpression -o Mopti-T3-YY_S189_genes.rpkm -t Mopti-T3-YY_S189.tr -G Mopti-T3-YY_S189_genes.txt Mopti-T3-YY_S189.rpkm

notebook:
terminal:
ssh -tL 9000:localhost:9543 bradmain@169.237.175.247 jupyter notebook --no-browser --port 9543 --port-retries=0
web:
localhost:9000/notebooks/gambiae/malphigs/Acol/cyp_expression.ipynb#


#Calc diff expression with odd samples removed
#compute overall mean and variance for each transcript
BitSeq/getVariance --log -o allgood.Lmean *.rpkm 

#estimate expression dependent hyperparameters
BitSeq/estimateHyperPar --meanFile allgood.Lmean -o allgood.param C*rpkm C T*rpkm


#estimate differential expression
BitSeq/estimateDE -o treatment_E -p allgood.param C*rpkm C T*rpkm

#sequenctially
../BitSeq/getVariance --log -o allgood.Lmean *.rpkm ; ../BitSeq/estimateHyperPar --meanFile allgood.Lmean -o allgood.param *C*rpkm C *T*rpkm ; ../BitSeq/estimateDE -o treatment_E -p allgood.param *C*rpkm C *T*rpkm


#Run bitseq with variational Bayes (VB) for gene expression only (faster)
#assumes we already ran BitSeq/parseAlignment
mkdir VB
link in .prob files
for file in *.prob
do
echo "BitSeq/estimateVBExpression -o $(basename $file .prob) $file"
done
> runVB

http://localhost:9000/notebooks/gambiae/malphigs/Acol/VB/VB_2.ipynb

