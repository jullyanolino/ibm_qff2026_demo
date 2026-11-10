# Demonstrações quânticas e pós-quânticas em Qiskit 2.4

**Material de apoio da palestra** *Uma década quântica na nuvem e a contagem regressiva da criptografia*
IBM Qiskit Fall Fest Brasil 2026 · 10/11/2026 · Autor: **Jullyano Lino**

[![Abrir no Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/SEU_USUARIO/SEU_REPOSITORIO/blob/main/01_qber_bb84_eve_qiskit.ipynb)
![Qiskit](https://img.shields.io/badge/Qiskit-2.4.2-6929c4)
![Python](https://img.shields.io/badge/Python-3.13%20(Colab)-blue)
![Licença](https://img.shields.io/badge/licen%C3%A7a-a%20definir-lightgrey)

> **Nota para quem publica:** troque `SEU_USUARIO/SEU_REPOSITORIO` nos links do Colab pelo caminho real do repositório e defina a licença (seção 12) antes de divulgar.

---

## 1. Visão geral

Três notebooks Jupyter, prontos para o **Google Colab**, reproduzem as demonstrações da palestra. Cada afirmação numérica é calculada no próprio notebook e conferida por `assert`; cada notebook traz teoria em Markdown, fontes, exercícios com verificação automática e gabaritos.

| # | Notebook | O que demonstra | Onde foi executado | Conta IBM? |
|---|---|---|---|---|
| 1 | [`01_qber_bb84_eve_qiskit.ipynb`](01_qber_bb84_eve_qiskit.ipynb) | Emulação em circuito do **BB84 com espiã (intercept-resend)**: QBER ≈ 0% sem Eve e ≈ 25% com Eve | Simulador ideal, simulador com ruído parametrizável, backend congelado (FakeFez) **e hardware real `ibm_fez`** | Só no modo `hardware` |
| 2 | [`02_pqc_tamanhos_handshake_lwe.ipynb`](02_pqc_tamanhos_handshake_lwe.ipynb) | **Tamanhos** de ML-KEM, ML-DSA e SLH-DSA derivados dos FIPS 203/204/205; **modelo de bytes** do handshake TLS 1.3; **LWE de brinquedo** | CPU (Colab) | Não |
| 3 | [`03_rsa896_conferencia_pxq.ipynb`](03_rsa896_conferencia_pxq.ipynb) | **Conferência de p × q = N** do RSA-896; teste de resposta conhecida (RSA-100); RSA de brinquedo; custo de fatorar; parte clássica do algoritmo de Shor | CPU (Colab) | Não |

### Mapa palestra → notebook

| Slide da palestra | Tema | Notebook | Saída a usar |
|---|---|---|---|
| 7 | RSA-896 caiu por GNFS clássico | 3 | `rsa896_conferencia.png`; `p × q == N: True` |
| 10 e 11 | Medir perturba; a Eve deixa rastro de 25% | 1 | `qber_comparacao.png`; `bb84_qber_resultados.json` |
| 12 e 13 | ML-KEM, ML-DSA, SLH-DSA | 2 | Tabela de tamanhos (seção 2 do notebook) |
| 14 | O custo é banda | 2 | Tabelas das seções 3 e 4 (clássico × PQC; cenários de handshake) |
| 22 | Reproduza | 1, 2 e 3 | Este repositório |

### O que estes notebooks **não** são
- O notebook 1 é **emulação do protocolo em circuito**, **não QKD fotônico**: não há fótons, canal óptico, perdas, detectores, canal clássico autenticado, correção de erros nem amplificação de privacidade.
- O notebook 2 **não** usa computador quântico (PQC é criptografia clássica resistente a Shor). O LWE implementado é **inseguro de propósito**.
- O notebook 3 **não quebra RSA** e **não executa o algoritmo de Shor**. Ele verifica uma fatoração obtida por métodos **clássicos** (GNFS) e mostra a assimetria entre *conferir* e *encontrar* fatores. A fatoração de RSA-896 não afeta chaves RSA-2048 em uso.

---

## 2. Resultados de referência (execuções reais)

Todos os números abaixo vêm de execuções no Google Colab (Python 3.13, `qiskit==2.4.2`, `qiskit-aer==0.18.2`, `qiskit-ibm-runtime==0.52.0`). Intervalos são de **Wilson a 95%**.

### 2.1 Notebook 1 · QBER, quatro ambientes com a mesma amostragem (256 shots por circuito)

24 circuitos × 256 shots = 6.144 shots por ambiente → **1.024 bits peneirados sem Eve e 2.048 com Eve**. É a execução que inclui o hardware.

| Ambiente | QBER sem Eve | QBER com Eve | Eve acerta o bit |
|---|---|---|---|
| `ideal` (Aer sem ruído) | 0,00% (0/1.024; IC 0,00–0,37%) | 24,66% (505/2.048; IC 22,84–26,57%) | 75,2% (IC 73,3–77,1%) |
| `noisy_custom` (p1 = 0,001; p2 = 0,01; leitura = 0,02) | 1,37% (14/1.024; IC 0,82–2,28%) | 26,17% (536/2.048; IC 24,31–28,12%) | 74,2% (IC 72,3–76,1%) |
| `noisy_fake` (FakeFez) | 0,00% (0/1.024; IC 0,00–0,37%) | 25,29% (518/2.048; IC 23,46–27,22%) | 74,9% (IC 73,0–76,7%) |
| **`hardware` (`ibm_fez`)** | **0,68%** (7/1.024; IC 0,33–1,40%) | **26,42%** (541/2.048; IC 24,55–28,37%) | **73,2%** (IC 71,2–75,1%) |

**Procedência do dado de hardware**

| Campo | Valor |
|---|---|
| Backend | `ibm_fez` (escolhido por `least_busy`; disponíveis na conta: `ibm_fez`, `ibm_kingston`, `ibm_marrakesh`) |
| Job ID | `dbp7tbk4148s739u3ao0` |
| Data (UTC) | 2026-11-10 02:18 (= 09/11/2026 23:18 em Brasília) |
| Par de qubits | transmissão = 114, auxiliar da Eve = 115 (escolhido por `best_pair` a partir do `target` do backend) |
| Erros informados pelo backend no momento | porta `cz` = 0,0021; leitura do qubit 114 = 0,0010 (valores da calibração do `target`, não medidos aqui) |
| Execução | 1 job, 24 circuitos, 256 shots por circuito (6.144 shots), `SamplerV2` em *job mode*, `optimization_level=1` |

**Como ler**
- No hardware real, interceptar e reenviar leva o QBER a **≈ 26%**, bem acima do chão sem espiã (**< 1,5% no IC superior**) e do limiar assintótico de ≈ 11%. A Eve é **claramente detectável** neste modelo, com margem de dezenas de pontos percentuais.
- O QBER com Eve no hardware (26,42%) fica ≈ 1,4 ponto acima de 25%. A regra de bolso do notebook (25% + 0,5 × chão sem Eve) dá 25,34%; a diferença restante é compatível com o erro **extra** que o circuito da Eve traz (uma porta CNOT a mais) e com a incerteza estatística (IC 24,55–28,37%, ±1,9 ponto). O gráfico da seção 9 mostra que, com a Eve na base certa, o erro no hardware fica em poucos pontos percentuais, acima do chão sem Eve, e perto de 50% com a Eve na base errada, como previsto.
- **Poder estatístico:** com 256 shots, a incerteza do QBER com Eve é de ≈ ±1,9 ponto. Com os 2.000 shots padrão do notebook (16.000 bits peneirados com Eve), cai para ≈ ±0,7 ponto. A execução de hardware usou 256 shots para poupar cota do plano gratuito.
- **Uma execução, um dia, um par de qubits.** O chão do dispositivo varia com a calibração e com o par escolhido; não generalize para "o desempenho do `ibm_fez`".
- A amostragem em hardware **nunca é reprodutível**. O que torna o dado verificável é o JSON com contagens brutas e metadados (seção 11 do notebook) e o job ID.

### 2.2 Notebook 1 · simulações com 2.000 shots (referência de alta precisão)

Mesma configuração com `SHOTS = 2000` (8.000 bits peneirados sem Eve e 16.000 com Eve), `SEED = 20261110`. Duas execuções com as mesmas versões produzem resultados **idênticos**.

| Modo | QBER sem Eve | QBER com Eve | Eve acerta o bit |
|---|---|---|---|
| `ideal` | 0,00% (0/8.000; IC 0,00–0,05%) | 24,89% (3.983/16.000; IC 24,23–25,57%) | 74,9% |
| `noisy_custom` | 1,90% (IC 1,62–2,22%) | 26,21% (IC 25,54–26,90%) | 73,9% |
| `noisy_fake` (FakeFez) | 0,22% (IC 0,14–0,36%) | 25,21% (IC 24,55–25,89%) | 74,5% |

Teoria: 0% e 25% (ideal). Limiar assintótico de ≈ 11%: **recalculado** no notebook (1 − 2·h₂(Q) = 0 → Q = 0,1100).

### 2.3 Notebook 2 · tamanhos e modelo de handshake

Os 12 conjuntos de parâmetros (ML-KEM-512/768/1024, ML-DSA-44/65/87, SLH-DSA-128/192/256 nas variantes `s` e `f`) têm tamanhos **derivados das fórmulas dos padrões** e conferidos por `assert` contra as tabelas publicadas. Destaques:

| Esquema | Chave pública | Cifrado / assinatura |
|---|---|---|
| ML-KEM-768 | 1.184 B | 1.088 B |
| ML-DSA-65 | 1.952 B | 3.309 B |
| SLH-DSA-128s / 128f | 32 B | 7.856 B / 17.088 B |

Razões frente ao clássico: troca de chave **×37** (1.184 B ÷ 32 B do X25519); assinatura **×52** (3.309 B ÷ 64 B, ECDSA P-256 "crua") ou **×46** (3.309 B ÷ 72 B, ECDSA em DER). O notebook usa a codificação DER (72 B) na tabela da seção 3; escolha a convenção e **diga qual** ao citar a razão.

**Modelo de handshake** (suposições explícitas: só compartilhamentos de chave + chaves públicas + assinaturas; com `n` certificados na cadeia há `n` chaves públicas e `n + 1` assinaturas, incluindo o `CertificateVerify`; ignora cabeçalhos, extensões, campos X.509, SCT e OCSP):

| Cenário | 1 certificado | 2 certificados | Voo do servidor (2 cert.) |
|---|---|---|---|
| Hoje (X25519 + ECDSA P-256) | 0,30 kB | 0,46 kB | 0,43 kB |
| Híbrido (X25519MLKEM768 + ECDSA P-256) | 2,57 kB | 2,73 kB | 1,52 kB |
| ML-KEM-768 + ML-DSA-44 | 8,42 kB | 12,16 kB | 10,97 kB |
| ML-KEM-768 + ML-DSA-65 | 10,84 kB | 16,10 kB | 14,92 kB |
| KEX híbrido + ML-DSA-65 | 10,91 kB | 16,17 kB | 14,95 kB |
| ML-KEM-768 + SLH-DSA-128s | 18,02 kB | 25,90 kB | 24,72 kB |

X25519MLKEM768: 1.216 B (cliente) e 1.120 B (servidor), conforme o rascunho IETF (a versão pode mudar). **Os valores dependem das suposições** (nº de certificados, família de assinatura, KEX híbrido ou não); ao citar um número, cite o cenário. O handshake real é **maior** que o modelo; ele serve para **comparar cenários**, não para prever bytes na rede.

Com ML-DSA-65 e 2 certificados, o voo do servidor (≈ 14,9 kB) passa da janela inicial típica do TCP (10 × 1.460 B ≈ 14,6 kB; RFC 6928), o que tende a custar um RTT extra. São valores típicos de um modelo, não medições.

**LWE de brinquedo** (n = 32, m = 128, q = 3329, erro em [−2, 2]): 0 falhas em 2.000 decifrações; expansão ≈ 396× por bit. Sem ruído o segredo cai por eliminação de Gauss mod q; com ruído, a mesma eliminação falha (resíduo máximo 2 com o segredo verdadeiro contra mediana ≈ 500 com o palpite da eliminação).

### 2.4 Notebook 3 · RSA-896

| Verificação | Resultado (execução no Colab) |
|---|---|
| `p × q == N` | **True** (471,8 µs) |
| Tamanhos | N: 270 dígitos / 896 bits; p e q: 135 dígitos / 448 bits cada |
| p e q primos prováveis (Miller–Rabin, 40 bases; erro ≤ 4⁻⁴⁰) | True, True |
| Verificação completa (produto + primalidade) | True (55,2 ms) |
| Teste de resposta conhecida (RSA-100) e controles negativos | aceitos / rejeitados como esperado |

Os tempos dependem da máquina. A fatoração em si custou ≈ **30 anos-GPU** (GNFS, até 2.048 GPUs, ≈ 10 dias; anúncio de S. A. Weis, 19/09/2026): este número vem **da fonte**, não do notebook.

Extrapolação **heurística** do GNFS, calibrada em RSA-896 = 30 anos-GPU (o termo o(1) é ignorado; memória e álgebra linear não escalam como a fórmula; leia como **ordem de grandeza**): ≈ 30× para RSA-1024 e ≈ 3,5 × 10¹⁰ × para RSA-2048.

Parte clássica do Shor (busca de ordem por força bruta, só em N pequeno): fração das escolhas de `a` coprimas que fatoram: N = 21 → 0,545; N = 35 → 0,783; N = 91 → 0,761; N = 221 → 0,911 (todas ≥ ½, como previsto).

---

## 3. Início rápido (Google Colab)

1. Abra o notebook desejado pelo botão do Colab (ou *Arquivo → Fazer upload de notebook*).
2. Execute **Ambiente de execução → Executar tudo**. A primeira célula instala as dependências com versões fixadas; se o Colab pedir, reinicie a sessão e execute novamente.
3. Os modos **simulados** funcionam sem configuração. Para o **hardware IBM** (apenas notebook 1), siga a seção 5.

Tempos aproximados (CPU do Colab gratuito): notebook 1 ≈ 1 min nos modos simulados; notebook 2 ≈ segundos; notebook 3 ≈ menos de 2 min.

> **Avisos benignos no Colab:** a instalação pode imprimir `pip's dependency resolver ... python-dateutil` (conflito com pacotes pré-instalados do Colab) e a conexão ao serviço IBM pode imprimir `Loading account with the given token. A saved account will not be used.` Nenhum dos dois impediu a execução.

### Execução local (opcional)
```bash
python -m venv .venv && source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install "qiskit==2.4.2" "qiskit-aer==0.18.2" "qiskit-ibm-runtime==0.52.0" \
            matplotlib scipy pandas numpy pylatexenc jupyterlab
jupyter lab
```
Fora do Colab, os segredos são lidos de **variáveis de ambiente** de mesmo nome (ou digitados via `getpass`).

---

## 4. Versões e reprodutibilidade

| Componente | Versão |
|---|---|
| `qiskit` | **2.4.2** (fixada) |
| `qiskit-aer` | 0.18.2 (fixada) |
| `qiskit-ibm-runtime` | 0.52.0 (fixada) |
| Python | 3.13 (Colab, nov/2026); confira a compatibilidade do Qiskit 2.4 com a versão do seu ambiente |

- Os modos locais usam `SEED = 20261110` (transpilador **e** amostrador `SamplerV2`). Com as mesmas versões e o mesmo número de shots, os resultados são **idênticos**; outras versões podem mudar os números em detalhes.
- Os resultados das seções 2.1 e 2.2 diferem porque usam números de shots diferentes (256 e 2.000), não porque algo mudou.
- **No hardware real a amostragem nunca é reprodutível.** O notebook 1 salva contagens brutas e metadados (versões, par de qubits, `job_id`, data) em `bb84_qber_resultados.json` e refaz a análise a partir do arquivo.

---

## 5. Tokens como *Secrets* (somente notebook 1, modo `hardware`)

O notebook 1 lê as credenciais do **Colab Secrets**; elas **nunca** ficam gravadas no `.ipynb`.

| Segredo | Obrigatório | Conteúdo |
|---|---|---|
| `IBM_CLOUD_TOKEN` | Sim (para hardware) | API key da IBM Quantum Platform (exibida uma única vez na criação) |
| `IBM_CLOUD_INSTANCE` | Recomendado | CRN (ou nome) da instância, copiado da página *Instances* |

**Passo a passo**
1. Barra lateral do Colab → ícone de **chave** (*Secrets*) → **Adicionar novo segredo**.
2. Crie os dois segredos acima.
3. Ative **Acesso ao notebook** em cada um.
4. No notebook 1, mude `RUN_HARDWARE = False` para `True` (seção 1) e execute **somente** a célula da seção 8. Depois reexecute as seções 9, 10 e 11 (gráfico, informação da Eve e JSON). Mantenha a **mesma sessão**: os resultados dos modos simulados ficam em memória.

**Boas práticas**
- O notebook nunca imprime o token; apenas informa se o segredo existe.
- Se a chave vazar, revogue-a no painel da IBM e crie outra.
- Quem copiar o notebook precisa criar os próprios segredos.
- Os nomes `IBM_CLOUD_TOKEN`/`IBM_CLOUD_INSTANCE` são rótulos do Colab e precisam coincidir com os usados nas células das seções 2 e 8.

**Cota.** O plano gratuito *Open* oferece até **10 minutos de tempo de QPU por janela móvel de 28 dias**. O experimento envia **um único job** (24 circuitos); o padrão do notebook é 2.000 shots por circuito (48.000 shots), e a execução de referência usou 256 (6.144 shots). O tempo de QPU consumido **não foi registrado** nas saídas; confira em *Usage* no painel da IBM e rode os modos simulados antes. Se a fila for longa, o `job_id` aparece na página *Jobs* da plataforma.

---

## 6. Notebook 1 · QBER com a Eve (BB84)

**Ideia.** Em BB84, Alice envia bits em bases aleatórias (Z/X) e Bob mede em bases aleatórias; após o *sifting* (A = B) os bits deveriam coincidir. Uma espiã que mede e reenvia (*intercept-resend*) erra a base em metade das rodadas e produz **QBER = ½ · ½ = 25%** nos bits peneirados; ela acerta o bit em 75% das rodadas.

**Implementação**
- 24 circuitos de 1 qubit de transmissão + 1 auxiliar: 8 sem Eve e 16 com Eve (todas as combinações de bit, base de Alice, base de Bob e base da Eve).
- A medição da Eve usa **medição adiada** (CNOT para um qubit auxiliar, lido ao final): sem circuitos dinâmicos, compatível com qualquer backend.
- Um único `SamplerV2` roda tudo; **o mesmo código** serve para simulador ideal, ruído parametrizável, backend congelado e QPU real (só muda o objeto `backend`).
- Estatística: QBER agregado com **intervalo de Wilson a 95%**; escolha automática do par de qubits de menor erro (`best_pair`).

**Modos de execução**

| Flag | Modo | Descrição |
|---|---|---|
| `RUN_IDEAL` | `ideal` | Aer sem ruído |
| `RUN_NOISY_CUSTOM` | `noisy_custom` | Ruído depolarizante e de leitura parametrizáveis (`P1`, `P2`, `READOUT`) |
| `RUN_NOISY_FAKE` | `noisy_fake` | Ruído derivado de um backend IBM congelado (`FakeFez`) |
| `RUN_HARDWARE` | `hardware` | QPU IBM via Qiskit Runtime (padrão: **desligado**) |

**Saídas geradas:** `bb84_qber_resultados.json` (dados brutos + metadados) e `qber_comparacao.png` (o rodapé do gráfico traz backend, job ID, data e shots quando há hardware).

**Atenção ao `job=` das simulações.** Nos modos simulados, a linha `[modo] backend=... job=<UUID>` mostra um identificador **local**; ele **não** é um job de hardware. Só o `job_id` do modo `hardware` (como `dbp7tbk4148s739u3ao0`) identifica uma execução na IBM.

**Exercícios (7).** Shots vs. largura do IC · Monte Carlo do protocolo e ataque parcial (QBER = 0,25·f) · varredura de ruído de leitura · Eve com eixo de medição arbitrário (a informação máxima ocorre em θ = π/4; o QBER fica em 25% para qualquer θ) · decisão de detecção com margem estatística · hardware: efeito do par de qubits e da mitigação de leitura · questões conceituais.

---

## 7. Notebook 2 · PQC: tamanhos, handshake e LWE

- **Tamanhos:** derivados dos parâmetros dos FIPS 203, 204 e 205 e comparados, com `assert`, às tabelas publicadas pelo Open Quantum Safe (liboqs).
- **Handshake:** modelo da seção 2.3, com suposições explícitas e a relação do voo do servidor com a janela inicial do TCP.
- **LWE:** esquema de Regev para 1 bit; mostra por que o ruído é essencial e o efeito do ruído sobre a falha de decifração (0% com B = 2 e B = 50; 7,6% com B = 100; 43,9% com B = 200; 50,7% com B = 400, para n = 32, m = 128, q = 3329). O ML-KEM usa **Module-LWE** sobre anéis polinomiais, com chaves e cifrados muito menores.

**Exercícios (6).** Tamanho de assinatura SLH-DSA com outro `w` · voo do servidor × janela do TCP (inclui cenário com RSA-2048) · taxa de falha do LWE vs. ruído · cifrar texto bit a bit · teste de resíduos · questões conceituais (anéis polinomiais, híbrido, ML-DSA × SLH-DSA).

---

## 8. Notebook 3 · RSA-896

- **Dados:** N, p e q copiados do anúncio de Stephen A. Weis (19/09/2026). O notebook **não confia na cópia**: recalcula produto, tamanhos e primalidade.
- Teste de resposta conhecida com **RSA-100** e **controles negativos** (fator errado, ordem trocada, soma preservada com produto alterado e fatoração trivial são rejeitados).
- RSA de livro-texto (inseguro, sem *padding*) para ver a chave quebrar em tamanhos pequenos.
- Medição do **ρ de Pollard (Brent)**: o tempo dobra a cada ~4 bits do módulo.
- Extrapolação **heurística** do GNFS (seção 2.4) e parte **clássica** do Shor, em que só a busca da ordem é a parte quântica.

**Saída gerada:** `rsa896_conferencia.png`.

**Exercícios (6).** Divisão por tentativas e a escala para 2⁴⁴⁸ · regressão do custo vs. bits · método de Fermat e por que p e q não podem ser próximos · fração de escolhas de `a` que fatora em Shor clássico · verificação estrita · primalidade certificada (ECPP) e a leitura correta do resultado de RSA-896.

---

## 9. Como os exercícios funcionam

- Cada exercício traz um **esqueleto** e uma célula de verificação que imprime `pendente` até você implementar.
- Os **gabaritos** ficam no fim de cada notebook em células de formulário do Colab (recolhidas por padrão) e rodam em simulador ou CPU, sem gastar cota de hardware.
- Sugestão de uso em aula: resolver 1–2 exercícios por notebook e comparar com o gabarito.

---

## 10. Estrutura do repositório

```
.
├── README.md
├── 01_qber_bb84_eve_qiskit.ipynb
├── 02_pqc_tamanhos_handshake_lwe.ipynb
├── 03_rsa896_conferencia_pxq.ipynb
└── resultados/                                   (opcional, recomendado)
    ├── bb84_qber_resultados_hardware.json        # contagens brutas + metadados da execução em ibm_fez
    ├── qber_comparacao.png
    └── rsa896_conferencia.png
```
Publique o JSON do experimento de hardware (contagens brutas e metadados, incluindo o job ID): é ele que permite a terceiros refazerem a análise sem reexecutar nada. Os demais arquivos gerados na execução não precisam ser versionados.

---

## 11. Limitações e uso responsável

- **Detectabilidade ≠ segurança.** O notebook 1 mostra que o *intercept-resend* é detectável neste modelo; não prova a segurança de QKD na prática (faltam fonte real, autenticação do canal clássico, blocos finitos, correção de erros, amplificação de privacidade e defesa contra canais laterais).
- O limiar de ≈ 11% é **assintótico e idealizado**; com blocos finitos e dispositivos reais o limite efetivo é menor.
- O resultado de hardware é **uma execução** (1 job, 256 shots por circuito, 1 par de qubits, 1 dia); o intervalo de confiança do QBER com Eve é de ≈ ±1,9 ponto.
- Os *fake backends* são uma fotografia estática da calibração e não capturam *crosstalk* nem deriva temporal: são uma aproximação, não uma previsão do hardware de hoje.
- Os tamanhos de handshake são um **modelo**; o handshake real é maior. A codificação de chaves e assinaturas clássicas (DER) é de ordem de grandeza.
- O código criptográfico (LWE, RSA) é **didático e inseguro**. Não use em produção.
- Cada notebook declara, nas fontes, o que foi **verificado** e o que foi apenas citado.

---

## 12. Fontes principais

**Verificadas na produção deste material**
- S. A. Weis, *RSA-896* (anúncio da fatoração): https://saweis.net/posts/rsa-896.html
- PostQuantum.com, *RSA-896 Factored with Claude on GPUs*: https://postquantum.com/security-pqc/rsa-896-factored-claude-gpus/
- Open Quantum Safe (liboqs), tabelas de tamanhos: [ML-KEM](https://openquantumsafe.org/liboqs/algorithms/kem/ml-kem) · [ML-DSA](https://openquantumsafe.org/liboqs/algorithms/sig/ml-dsa) · [SLH-DSA](https://registry.projecteleven.com/algorithms/slh-dsa)
- IETF, rascunho *Post-quantum hybrid ECDHE-MLKEM Key Agreement for TLSv1.3* (X25519MLKEM768; rascunho, confira a versão vigente)
- IBM Quantum Documentation: [*Initialize your account*](https://quantum.cloud.ibm.com/docs/en/guides/initialize-account) · [*Plans overview*](https://quantum.cloud.ibm.com/docs/guides/plans-overview) · [*Manage instances*](https://quantum.cloud.ibm.com/docs/guides/instances)

**Referências clássicas citadas nos notebooks**
- NIST: [FIPS 203](https://csrc.nist.gov/pubs/fips/203/final), [FIPS 204](https://csrc.nist.gov/pubs/fips/204/final), [FIPS 205](https://csrc.nist.gov/pubs/fips/205/final) (13/08/2024). Os tamanhos foram conferidos por derivação das fórmulas e contra as tabelas do liboqs.
- C. H. Bennett, G. Brassard, *Quantum cryptography: Public key distribution and coin tossing* (BB84), Theor. Comput. Sci. 560 (2014), DOI 10.1016/j.tcs.2014.05.025.
- W. K. Wootters, W. H. Zurek, *A single quantum cannot be cloned*, Nature 299 (1982).
- V. Scarani et al., *The security of practical quantum key distribution*, Rev. Mod. Phys. 81 (2009).
- P. W. Shor, J. Preskill, *Simple proof of security of the BB84 quantum key distribution protocol*, Phys. Rev. Lett. 85 (2000).
- P. W. Shor, *Polynomial-time algorithms for prime factorization and discrete logarithms on a quantum computer*, SIAM J. Comput. 26 (1997).
- C. Gidney, *How to factor 2048 bit RSA integers with less than a million noisy qubits*, arXiv:2505.15917 (2025).
- O. Regev, *On lattices, learning with errors, random linear codes, and cryptography*, J. ACM 56 (2009).
- Pollard (1975) e Brent (1980), métodos ρ de Monte Carlo; Miller (1976) e Rabin (1980), testes de primalidade.
- RFC 6928 (janela inicial do TCP) e RFC 7748 (X25519).

---

## 13. Licença, citação e contato

**Licença:** *a definir pelo autor.* Sugestão usual para material didático: código sob MIT e texto/figuras sob CC BY 4.0. Inclua o arquivo `LICENSE` correspondente antes de publicar.

**Como citar o material**
```
LINO, J. Demonstrações quânticas e pós-quânticas em Qiskit 2.4 (material da palestra
"Uma década quântica na nuvem e a contagem regressiva da criptografia").
IBM Qiskit Fall Fest Brasil 2026, 10 nov. 2026.
```

**Como citar o resultado de hardware (modelo de frase)**
> Emulação do BB84 com espiã *intercept-resend* em circuito (não QKD fotônico), executada em `ibm_fez` (job `dbp7tbk4148s739u3ao0`, 10/11/2026 02:18 UTC; 24 circuitos × 256 shots; qubits 114 e 115): QBER de 0,68% (IC95 0,33–1,40%) sem Eve e 26,42% (IC95 24,55–28,37%) com Eve.

**Autor:** Jullyano Lino · Qiskit Advocate · consultor em cibersegurança.
Correções e sugestões são bem-vindas via *issues* do repositório.
