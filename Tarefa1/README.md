Nome: README.md
Tipo: Todos os arquivos (*.*)
Codificação: UTF-8
Local: dentro da pasta Tarefa 1

# Tarefa 1 — Do sequenciamento ao alinhamento

## Objetivo

Realizar o download de reads de sequenciamento, selecionar um subconjunto de reads, obter um genoma de referência, realizar o alinhamento, gerar e indexar um arquivo BAM e visualizar os alinhamentos no IGV.

## Organismo escolhido

O organismo escolhido foi *Escherichia coli* str. K-12 substr. MG1655.

Trata-se de uma bactéria cujo genoma de referência utilizado possui um cromossomo circular. O uso de um genoma bacteriano foi adequado para a atividade porque possui tamanho relativamente pequeno e é viável para execução em um computador pessoal com Windows.

## Reads utilizadas

- Origem: NCBI Sequence Read Archive (SRA)
- Run accession: `ERR008613`
- Plataforma: Illumina
- Tipo de biblioteca: paired-end
- Arquivos FASTQ completos gerados:
  - `ERR008613_1.fastq`
  - `ERR008613_2.fastq`
- Subconjunto utilizado:
  - `reads/subset_R1.fastq`
  - `reads/subset_R2.fastq`
- Tamanho aproximado do subset: 50.000 pares de reads, correspondendo a 100.000 reads individuais.

Os FASTQs completos não foram enviados ao repositório porque são muito grandes. Eles podem ser recuperados a partir do acesso `ERR008613` no NCBI SRA.

## Genoma de referência

- Fonte: Ensembl Bacteria
- Diretório principal:
  https://ftp.ensemblgenomes.ebi.ac.uk/pub/bacteria/current/
- Caminho utilizado no repositório:

```text
bacteria/
→ bacteria_0_collection/
→ escherichia_coli_k_12_mg1655_gca_000005845/
→ dna/
```

- Arquivo escolhido:

```text
Escherichia_coli_str_k_12_substr_mg1655_gca_000005845.ASM584v2.dna.toplevel.fa.gz
```

- Arquivo descompactado e renomeado localmente:

```text
reference/ecoli_k12_ref.fa
```

- Assembly: `GCA_000005845`
- Nome da assembly: `ASM584v2`

- Arquivo usado localmente:

```text
reference/ecoli_k12_ref.fa
```

- Link da referência:

https://ftp.ensemblgenomes.ebi.ac.uk/pub/bacteria/current/fasta/bacteria_0_collection/escherichia_coli_str_k_12_substr_mg1655_gca_000005845/dna/Escherichia_coli_str_k_12_substr_mg1655_gca_000005845.ASM584v2.dna.toplevel.fa.gz

### Escolha da referência

Foram disponibilizados arquivos `dna.chromosome`, `dna.toplevel`, `dna_rm.chromosome`, `dna_rm.toplevel`, `dna_sm.chromosome` e `dna_sm.toplevel`.

Foi escolhido o arquivo `dna.toplevel`, pois ele contém as sequências de referência de nível superior da montagem, sem mascaramento de regiões repetitivas. Esse formato é apropriado para um alinhamento inicial das reads contra o genoma de referência.

## Ferramentas

- Sistema operacional: Windows
- SRA Toolkit 3.4.1
- BWA-MEM2 2.3 para Windows
- Minimap2 2.31-r1302 para Windows
- SAMtools 1.23.1 para Windows
- IGV Desktop

## Comandos executados

### Download dos dados SRA

```bat
prefetch.exe ERR008613
```

### Conversão para FASTQ paired-end

```bat
fasterq-dump.exe ERR008613 --split-files --outdir "C:\BioinfoGeral\Tarefa1\reads" --threads 4 --progress
```

### Criação do subset de reads

Foram selecionados os primeiros 50.000 pares de reads. Como cada read FASTQ ocupa quatro linhas, foram extraídas 200.000 linhas de cada arquivo.

```bat
powershell -Command "Get-Content 'C:\BioinfoGeral\Tarefa1\reads\ERR008613_1.fastq' -TotalCount 200000 | Set-Content 'C:\BioinfoGeral\Tarefa1\reads\subset_R1.fastq'; Get-Content 'C:\BioinfoGeral\Tarefa1\reads\ERR008613_2.fastq' -TotalCount 200000 | Set-Content 'C:\BioinfoGeral\Tarefa1\reads\subset_R2.fastq'"
```

### Indexação inicial da referência com BWA-MEM2

```bat
"C:\BioinfoGeral\programas\bwa-mem2-2.3-windows-x86_64-msys\bwa-mem2\bwa-mem2.exe" index "C:\BioinfoGeral\Tarefa1\reference\ecoli_k12_ref.fa"
```

### Tentativa de alinhamento com BWA-MEM2

```bat
"C:\BioinfoGeral\programas\bwa-mem2-2.3-windows-x86_64-msys\bwa-mem2\bwa-mem2.exe" mem -t 4 -R "@RG\tID:ERR008613_subset\tSM:E_coli_K12_MG1655\tPL:ILLUMINA\tLB:ERR008613" "C:\BioinfoGeral\Tarefa1\reference\ecoli_k12_ref.fa" "C:\BioinfoGeral\Tarefa1\reads\subset_R1.fastq" "C:\BioinfoGeral\Tarefa1\reads\subset_R2.fastq" > "C:\BioinfoGeral\Tarefa1\bam\ecoli_ERR008613_subset.sam"
```

### Alinhamento final com minimap2

```bat
minimap2.exe -ax sr -t 4 -R "@RG\tID:ERR008613_subset\tSM:E_coli_K12_MG1655\tPL:ILLUMINA\tLB:ERR008613" "C:\BioinfoGeral\Tarefa1\reference\ecoli_k12_ref.fa" "C:\BioinfoGeral\Tarefa1\reads\subset_R1.fastq" "C:\BioinfoGeral\Tarefa1\reads\subset_R2.fastq" > "C:\BioinfoGeral\Tarefa1\bam\ecoli_ERR008613_subset.sam"
```

### Conversão e ordenação para BAM

```bat
"C:\BioinfoGeral\programas\samtools-1.23.1-windows-ucrt64\samtools.exe" sort -@ 4 -o "C:\BioinfoGeral\Tarefa1\bam\ecoli_ERR008613_subset.sorted.bam" "C:\BioinfoGeral\Tarefa1\bam\ecoli_ERR008613_subset.sam"
```

### Indexação do BAM

```bat
"C:\BioinfoGeral\programas\samtools-1.23.1-windows-ucrt64\samtools.exe" index "C:\BioinfoGeral\Tarefa1\bam\ecoli_ERR008613_subset.sorted.bam"
```

### Estatísticas de alinhamento

```bat
"C:\BioinfoGeral\programas\samtools-1.23.1-windows-ucrt64\samtools.exe" flagstat "C:\BioinfoGeral\Tarefa1\bam\ecoli_ERR008613_subset.sorted.bam" > "C:\BioinfoGeral\Tarefa1\bam\ecoli_ERR008613_subset.flagstat.txt"
```

## Parâmetros utilizados

- `-a`: gera a saída em formato SAM.
- `-x sr`: preset do minimap2 para reads genômicas curtas e precisas.
- `-t 4`: utiliza quatro threads de processamento.
- `-R`: adiciona informações de read group ao alinhamento.
- `-@ 4`: utiliza quatro threads no SAMtools durante a ordenação.
- `--split-files`: separa reads paired-end em arquivos R1 e R2.
- `--threads 4`: utiliza quatro threads durante a conversão de SRA para FASTQ.

## Erro encontrado e solução

O BWA-MEM2 2.3 para Windows conseguiu indexar corretamente a referência, mas falhou durante o alinhamento no estágio `worker_bwt`, gerando um arquivo SAM com 0 bytes.

O problema persistiu mesmo após reduzir o subset de 50.000 pares para 5.000 pares e reduzir o número de threads de quatro para uma. Como alternativa, foi utilizado o minimap2 2.31-r1302 para Windows com o preset `-x sr`, adequado para reads curtas de Illumina.

O minimap2 gerou corretamente um arquivo SAM de aproximadamente 35 MB, que foi convertido e ordenado em BAM com SAMtools.

## Resultados do alinhamento

Resultado de `samtools flagstat`:

```text
100010 + 0 in total (QC-passed reads + QC-failed reads)
100000 + 0 primary
0 + 0 secondary
10 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
99178 + 0 mapped (99.17% : N/A)
99168 + 0 primary mapped (99.17% : N/A)
100000 + 0 paired in sequencing
50000 + 0 read1
50000 + 0 read2
98526 + 0 properly paired (98.53% : N/A)
98564 + 0 with itself and mate mapped
604 + 0 singletons (0.60% : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```

A taxa de mapeamento foi de 99,17%, indicando que a maioria das reads apresentou correspondência com a referência de *E. coli* K-12 MG1655. Além disso, 98,53% dos pares estavam corretamente pareados, demonstrando consistência entre as reads R1 e R2.

## Visualização no IGV

O genoma `ecoli_k12_ref.fa` foi carregado no IGV, seguido do BAM ordenado e indexado.

Foram utilizadas duas formas de visualização:

1. Cobertura e alinhamentos individuais: a pista superior representa a profundidade de cobertura, enquanto a pista inferior exibe as reads alinhadas. Essa visualização permite identificar regiões com diferentes níveis de cobertura e possíveis diferenças entre reads e referência.

2. Pares de reads e orientação das fitas: foi utilizado o modo `View as pairs` e a opção `Color alignments by → Read strand`. Essa visualização permite verificar a organização paired-end, a orientação das reads e a consistência dos pares.

## Arquivos do repositório

- `reads/subset_R1.fastq`: subset de reads forward.
- `reads/subset_R2.fastq`: subset de reads reverse.
- `reference/ecoli_k12_ref.fa`: genoma de referência.
- `bam/ecoli_ERR008613_subset.sorted.bam`: alinhamentos ordenados em formato BAM.
- `bam/ecoli_ERR008613_subset.sorted.bam.bai`: índice do BAM.
- `bam/ecoli_ERR008613_subset.flagstat.txt`: estatísticas de alinhamento.
- `images/`: screenshots da escolha da referência e das visualizações no IGV.