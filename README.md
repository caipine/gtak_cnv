# gtak_cnv


gatk PreprocessIntervals \
    -L targets_C.interval_list \
    -R /home/qcai1/Downloads/gatk_CNV/Homo_sapiens_assembly38.fasta \
    --bin-length 0 \
    --interval-merging-rule OVERLAPPING_ONLY \
    -O sandbox/targets_C.preprocessed.interval_list

    #replace CollectFragmentCounts with CollectReadCounts
gatk CollectReadCounts \
    -I tumor.bam \
    -L sandbox/targets_C.preprocessed.interval_list \
    --interval-merging-rule OVERLAPPING_ONLY \
    -O sandbox/tumor.counts.hdf5
    
gatk --java-options "-Xmx6500m" CreateReadCountPanelOfNormals \
    -I HG00133.alt_bwamem_GRCh38DH.20150826.GBR.exome.counts.hdf5 \
    -I HG00733.alt_bwamem_GRCh38DH.20150826.PUR.exome.counts.hdf5 \
    -I NA19654.alt_bwamem_GRCh38DH.20150826.MXL.exome.counts.hdf5 \
    --minimum-interval-median-percentile 5.0 \
    -O sandbox/cnvponC.pon.hdf5
    
gatk --java-options "-Xmx12g" DenoiseReadCounts \
    -I hcc1143_T_clean.counts.hdf5 \
    --count-panel-of-normals cnvponC.pon.hdf5 \
    --standardized-copy-ratios sandbox/hcc1143_T_clean.standardizedCR.tsv \
    --denoised-copy-ratios sandbox/hcc1143_T_clean.denoisedCR.tsv    
    
gatk PlotDenoisedCopyRatios \
    --standardized-copy-ratios hcc1143_T_clean.standardizedCR.tsv \
    --denoised-copy-ratios hcc1143_T_clean.denoisedCR.tsv \
    --sequence-dictionary Homo_sapiens_assembly38.dict \
    --minimum-contig-length 46709983 \
    --output sandbox/plots \
    --output-prefix hcc1143_T_clean



######################################################################

gatk --java-options "-Xmx3g" CollectAllelicCounts \
    -L chr17_theta_snps.interval_list \
    -I normal.bam \
    -R /home/qcai1/Downloads/gatk_CNV/Homo_sapiens_assembly38.fasta  \
    -O sandbox/hcc1143_N_clean.allelicCounts.tsv


gatk --java-options "-Xmx3g" CollectAllelicCounts \
    -L chr17_theta_snps.interval_list \
    -I tumor.bam \
    -R /home/qcai1/Downloads/gatk_CNV/Homo_sapiens_assembly38.fasta  \
    -O sandbox/hcc1143_T_clean.allelicCounts.tsv



gatk --java-options "-Xmx4g" ModelSegments \
    --denoised-copy-ratios hcc1143_T_clean.denoisedCR.tsv \
    --allelic-counts sandbox/hcc1143_T_clean.allelicCounts.tsv \
    --normal-allelic-counts sandbox/hcc1143_N_clean.allelicCounts.tsv \
    --output sandbox \
    --output-prefix hcc1143_T_clean


gatk CallCopyRatioSegments \
    --input hcc1143_T_clean.cr.seg \
    --output sandbox/hcc1143_T_clean.called.seg
    
    
