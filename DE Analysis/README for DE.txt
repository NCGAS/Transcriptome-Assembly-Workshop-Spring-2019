#This is an example script for running differential expression analyses using the output from evigenes.
#For more information on selecting software to use in DE, see our lection on differential gene expression - ppt here: https://iu.app.box.com/s/3xo07ugnl7d3q2p2hy4qy0ow1n7blhtn/file/322723696063

#load modules

#fix to R
export R_LIBS=/N/soft/rhel7/r/3.4.4/lib64/R/library/

module load bioconductor 
module load trinityrnaseq/2.6.6 
module load kallisto


#make an index of the transcriptome with kallisto
kallisto index -i combined_kal_index ../okayset/combined.okay.fa

#quantify transcripts - in this case we are using individual samples that were parked in the input_files folder before being combined into left.fa and right.fa
#generally a good idea to end output files in "_kallisto" for use with ./adjust_matrix.sh script
kallisto quant -i combined_kal_index -o Sample2_control_kallisto ../../input_files/Sample2Cont_1.fq ../../input_files/Sample2Cont_2.fq 
kallisto quant -i combined_kal_index -o Sample14_control_kallisto ../../input_files/Sample14Cont_1.fq ../../input_files/Sample14Cont_2.fq 
kallisto quant -i combined_kal_index -o Sample5_treat_kallisto ../../input_files/Sample5Treat_1.fq ../../input_files/Sample5Treat_2.fq 
kallisto quant -i combined_kal_index -o Sample12_treat_kallisto ../../input_files/Sample12Treat_1.fq ../../input_files/Sample12Treat_2.fq

#make a table using custom scripts to combine above count files
#make required changes to the adjust_matrix.sh file - namely the header desired!
./adjust_matrix.sh

#annotate your sample design according to trinity's documentation
nano samples_described

#run DE, in this case using edgeR (due to lack of three replicates)
/N/soft/rhel7/trinityrnaseq/2.4.0/Analysis/DifferentialExpression/run_DE_analysis.pl --matrix matrix --method edgeR --samples_file samples_described

#run DE analyses (in this case using p=.5 due to the fact this was based on a toy data set with no DE
cd edgeR.*/ 
/N/soft/rhel7/trinityrnaseq/2.4.0/Analysis/DifferentialExpression/analyze_diff_expr.pl --matrix ../matrix -P 5e-1 -C 2 --sample ../samples_described
