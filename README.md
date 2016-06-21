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
BitSeq/getVariance --log -o experiment_name.Lmean *.rpkm 

#estimate expression dependent hyperparameters
BitSeq/estimateHyperPar --meanFile experiment_name.Lmean -o experiment_name.param Mopti-C*rpkm C Mopti-T*rpkm

#estimate differential expression
BitSeq/estimateDE -o Mopti -p experiment_name.param Mopti-C*rpkm C Mopti-T*rpkm

#You can run the above commands sequentially like:

BitSeq/getVariance --log -o Mopti_rmdup.Lmean Mopti*rmdup.rpkm ; BitSeq/estimateHyperPar --meanFile Mopti_rmdup.Lmean -o Mopti_rmdup.param Mopti-C*rmdup.rpkm C Mopti-T*rmdup.rpkm ; BitSeq/estimateDE -o Mopti_rmdup -p Mopti_rmdup.param Mopti-C*rmdup.rpkm C Mopti-T*rmdup.rpkm


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

