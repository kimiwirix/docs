## Pasos 16 S analysis 

#### 1. Secuenciar 16S extracciones DNA
#### 2. Subir secuencias (Illumina) al cluster 
#### 3. Descomprimir folders 
1. Quitar .gz de todas las secuencias
    ```
    gunzip -r NS*
    ```
#### 4. Preparación files 
1. Hacer manifest en excel ods con \t separando
    ```console
        sample-id       forward-absolute-filepath       reverse-absolute-filepath
        N500001A        /mnt/atgc-d3/sur/users/nsaid/2024-06.novogene_16S_NS1_NS2/Batch_1/result_X202SC23054171-Z01-F001/01.RawData/N500001A/N500001A.raw_1.fastq       /mnt/atgc-	d3/sur/users/nsaid/2024-06.novogene_16S_NS1_NS2/Batch_1/result_X202SC23054171-Z01-F001/01.RawData/N500001A/N500001A.raw_2.fastq
        N500002A        /mnt/atgc-d3/sur/users/nsaid/2024-06.novogene_16S_NS1_NS2/Batch_1/result_X202SC23054171-Z01-F001/01.RawData/N500002A/N500002A.raw_1.fastq       /mnt/atgc-	d3/sur/users/nsaid/2024-06.novogene_16S_NS1_NS2/Batch_1/result_X202SC23054171-Z01-F001/01.RawData/N500002A/N500002A.raw_2.fastq
    ```
    Con echo, hacer un nuevo file en cluster y pegar manifest con \t, guardar como tsv. Cada renglón es una muestra diferente.
    ```
    echo -e "paste \t manifest in parts" > manifest.tsv
    echo -e "paste \t manifest in parts" >> manifest.tsv
    ```

2. Referencias (Sanger)
    1. Meter abi files en programa R `abi_references.R` para convertir a fasta  (output archivos **_contig.fasta** son los que nos sirven)
    2. Quitar espacios \n de cada fasta en Notepad++
    3. Unir todos los fasta en 1 documento en powershell
            ```
            cat *_contig.fa > reference_seqs_sangercontigR
            ```
    4. Trimmear primers de referencias con programa en R `matrices_open_reference.R` parte 4 porque son primers degenerados que pueden account for differences in vsearch
    5. Subir a cluster <br>
            **CONTROL:** Buscar en https://img.jgi.doe.gov/ con el IMG genome ID y hacer un blast con secuencia del id y secuencia de la referencia para comprobar si son iguales

#### 5. Correr qiime en cluster 
1. Correr `qsub_16Sanalysis.sh` para open reference
    **Outputs:** 
    - Tabla frecuencias **.tsv** 
    - Unmatched sequences **.fasta** 

#### 6. Dividir excel Tabla frecuencias .tsv en matched y unmatched 

#### 7. If unmatched sequences in excel have big frequencies we need to incorporate them to the final frequency table 
1. R program `matrices_open_reference.R` **parte 1** para cortar la tabla a solo los valores del dia 0 (valores iniciales)
2. R program `matrices_open_reference.R` **parte 2** para ver que unmatched ids concuerdan con ausencia y presencia de datos NS syncoms 
3. R program `matrices_open_reference.R` **parte 3** para calcular mahattan distance entre unmatched y cada vector de cepa en NS syncoms, solo selecciona los que sean idénticos <br>
    **CONTROL:** Checar taxonomías en NCBI de ref vs matched manhattan

    **Outputs:** total abundance table con las abundancias totales finales ya para hacer gráficas

#### 8. Graphs 
1. For filtered frequency table with `graphs.R` and `graphs2.R`
2. For unfiltered frequency table with `graphs_without_filtering.R` and `graphs2.R` <br>
    **Check** if all are same replica (A or B) if not, manually declare which oneas are A and which ones B  

#### 9. Árbol
1. Hacer un fasta con `merge_file.py`  con seqs de referencia trimmed y secuencias sacadas a partir de ids de unmatched en archivo `matching_unmatctched_manhattan.txt` con `extract_from_fasta.py`
2. Meter ese fasta a muscle
3. Meter fasta de muscle a mega evolution para hacer árbol 

        Maximum likelihood Tree 
        Bootstrap 500 replications
        General Time Reversible Model
        Gamma distributed 5 cateories
        NNI
        NJ/BioNJ
        Threads:1 para que no crashee



