# Projeto: Processamento de Contas Bancárias em COBOL

## Introdução
Este projeto foi desenvolvido em COBOL no ambiente Mainframe emulado (TK5/TN3270). O sistema tem como objetivo principal o processamento em lote (Batch) de arquivos de contas bancárias. Através de scripts JCL e lógica estruturada em COBOL, o sistema lê os dados, ordena os registros por agência, executa a validação das informações e gera um relatório consolidado com resumo geral.

## Funcionalidades
- **Concatenação de Arquivos:** O sistema unifica dados do arquivo principal (`CONTAS.TXT`) e de um arquivo extra (`CONTAS.NOVAS.TXT`) no momento da ordenação.
- **Ordenação via JCL (SORT):** Os registros são ordenados crescentemente por Agência e desempatados pelo Número da Conta, garantindo a organização antes do processamento no COBOL.
- **Uso de Copybook:** A estrutura de variáveis (54 posições) foi padronizada e isolada no arquivo externo `REGCONTA.cpy`, sendo injetada dinamicamente na compilação através do parâmetro `LIB`.
- **Validação de Dados:** O programa identifica e valida os tipos de conta ("C" para Corrente e "P" para Poupança). Contas com tipos não reconhecidos são classificadas no relatório como "INVÁLIDO".
- **Cálculo de Totais:** O processamento acumula e exibe no final do relatório o total exato de contas processadas e o saldo total financeiro do banco.

## Regras
Cada registro processado possui 54 posições divididas no seguinte formato da copybook `REGCONTA`:
* **05 NUM-CONTA:** Numérico (08 posições)
* **05 NOME-CLIENTE:** Alfabético (30 posições)
* **05 AGENCIA:** Numérico (04 posições)
* **05 TIPO-CONTA:** Alfabético (01 posição) -> `C = Conta Corrente` | `P = Conta Poupança`
* **05 SALDO:** Numérico Sinalizado Decimal (11 posições, sendo 2 decimais)

## Estrutura

```text
projeto4-contas-bancarias/
│
├── README.md                  # Documentação atual
│
├── evidencias/                # Evidências de execução com sucesso
│   ├── maxcc_0000.png         # Compilação sem erros (MAXCC 0000)
│   └── relatorio_tela.png     # Print do relatório processado na SYSOUT
│
├── src/                       
│   └── CONTAS.cbl             # Código-fonte principal do programa COBOL
│
├── copy/                      
│   └── REGCONTA.cpy           # Arquivo Copybook com o layout de 54 posições
│
├── jcl/                       
│   ├── JCOMPILA.jcl           # Script para compilação e Linkage Editor
│   └── JEXECUTA.jcl           # Script para concatenação, ordenação (SORT) e execução
│
├── data/                      
│   ├── CONTAS.txt             # Massa de dados original
│   └── CONTAS.NOVAS.txt       # Massa de dados extra 
│
└── output/                    
    └── RELATORIO_SYSOUT.txt   # Relatório final gerado
```

# Como Executar no Ambiente Mainframe (TK5)
- Alocação dos Arquivos: Crie no Mainframe os datasets sequenciais para os arquivos de dados (RECFM=FB, LRECL=54, BLKSIZE=540).
- Compilação: Acesse a biblioteca onde o JCL está salvo e submeta o job de compilação (SUB no arquivo JCOMPILA.jcl). O JCL deve conter o parâmetro PARM.COB='LIB,LOAD,SIZE=2048K,BUF=1024K' em linha única contínua para habilitar os Copybooks. Verifique a SYSOUT para garantir o MAXCC 0000.
- Execução: Submeta o arquivo JEXECUTA.jcl. Ele criará um arquivo temporário em memória (&&ORD), fará o SORT com a concatenação dos dados e entregará o arquivo limpo para o executável COBOL.
- Resultados: Acesse a área de Outlist (Opção =3.8 na SYSOUT) e visualize o relatório bancário impresso contendo as listagens agrupadas, validações e totais.
