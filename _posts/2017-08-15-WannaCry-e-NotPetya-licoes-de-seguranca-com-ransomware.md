---
layout: post
author: Wellington Pardim
title:  "WannaCry e NotPetya: lições de segurança com ransomware"
categories: "security"
position: 10
---

Maio de 2017 ficou marcado na história da segurança da informação. Em menos de um mês, o mundo foi atacado por duas variantes de ransomware que causaram bilhões em danos: **WannaCry** e **NotPetya**. Vamos entender como funcionaram e o que podemos aprender.

## WannaCry: 12 de maio de 2017

Em questão de horas, o WannaCry infectou mais de **230.000 computadores** em **150 países**. Hospitais no Reino Unido tiveram que recusar pacientes, fábricas da Renault pararam, e a FedEx foi severamente impactada.

### Como ele se espalhava

O WannaCry explorava uma vulnerabilidade no protocolo **SMBv1** (Server Message Block) do Windows, chamada **EternalBlue** (CVE-2017-0144). Essa exploit foi desenvolvida pela NSA e vazou por um grupo chamado **Shadow Brokers** em abril de 2017.

A propagação era automatizada:

1. O WannaCry escaneava a rede local e a internet procurando portas **445** (SMB) abertas
2. Ao encontrar um sistema vulnerável, enviava o payload EternalBlue
3. O exploit executava código remoto no sistema alvo
4. O ransomware se instalava e repetia o processo

```
Scan porta 445 → EternalBlue → Instalação → Criptografia → Scan novamente
         ↑                                                        |
         └────────────────────────────────────────────────────────┘
```

Isso fazia do WannaCry um **worm** — ele se propagava automaticamente sem interação humana.

### A criptografia

Uma vez instalado, o WannaCry:

- Criptografava arquivos com extensões comuns (`.doc`, `.pdf`, `.jpg`, `.xlsx`, etc.) usando **AES-128-CBC**
- A chave AES era então criptografada com **RSA-2048**
- Exigia pagamento de **$300-$600 em Bitcoin** para descriptografar
- Ameaçava dobrar o valor após 3 dias e deletar os arquivos após 7 dias

### O kill switch

O pesquisador de segurança britânico **Marcus Hutchins** (MalwareTech) descobriu que o WannaCry verificava se um domínio específico estava registrado antes de executar:

```
iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com
```

Hutchins registrou esse domínio por $10.69. Isso ativou um "kill switch" no malware que impedia a execução. Foi uma medida temporária — variantes subsequentes removeram essa verificação.

### Lição técnica

A vulnerabilidade já tinha patch disponível desde **março de 2017** (MS17-010). Os sistemas infectados eram aqueles que não tinham sido atualizados. Muitos eram Windows XP e Windows 7 em ambientes corporativos.

## NotPetya: 27 de junho de 2017

Um mês depois, outro ataque devastador. O NotPetya parecia ransomware, mas na verdade era um **wiper** — seus dados eram destruídos, não apenas criptografados.

### Vetor de ataque diferente

O NotPetya se espalhou inicialmente através de um software de contabilidade ucraniano chamado **M.E.Doc**. Uma atualização comprometida distribuiu o malware para milhares de empresas na Ucrânia.

Depois da infecção inicial, o NotPetya usava **três métodos** de propagação:

1. **EternalBlue** (mesmo exploit do WannaCry)
2. **EternalRomance** (outra exploit SMB vazada pela NSA)
3. **Mimikatz** — extraía credenciais da memória e usava **PsExec** e **WMI** para se mover lateralmente na rede

```python
# Pseudocódigo da propagação lateral
def propagar(rede):
    for host in rede.escanear():
        if host.tem_smb_vulneravel():
            eternal_blue(host)
        elif host.tem_credenciais_cacheadas():
            creds = mimikatz_extrair(host)
            psexec_executar(host, creds)
```

### Por que era um wiper, não ransomware

O NotPetya criptografava o **Master Boot Record (MBR)** e a **Master File Table (MFT)** do sistema. A chave de descriptografia gerada era baseada em valores aleatórios que não eram enviados a nenhum servidor. Mesmo que a vítima pagasse, não havia como recuperar os dados.

### Danos

- **Maersk** (maior empresa de shipping do mundo): perdeu quase toda sua infraestrutura de TI, reconstruiu 45.000 PCs e 4.000 servidores em 10 dias
- **Merck** (farmacêutica): $870 milhões em perdas
- **FedEx/TNT Express**: $400 milhões
- **Mondelez**: $188 milhões
- **Custo total estimado**: **$10+ bilhões**

## Lições de segurança

### 1. Patching é crítico

O WannaCry explorou uma vulnerabilidade com patch disponível há 2 meses. Manter sistemas atualizados não é opcional.

### 2. Segmente sua rede

Se o WannaCry ou NotPetya não conseguissem alcançar toda a rede, o dano seria menor. Segmentação de rede limita a propagação lateral.

### 3. Backups offline

Ransomware só funciona se você não tiver backups. Backups offline (não conectados à rede) são imunes a criptografia.

### 4. Princípio do menor privilégio

O Mimikatz extrai credenciais de administradores logados. Se cada sistema tiver apenas as permissões necessárias, a propagação lateral é limitada.

### 5. Desabilite serviços desnecessários

Se o SMBv1 não fosse necessário em muitos sistemas, o EternalBlue não teria funcionado. Desabilitar serviços não utilizados reduz a superfície de ataque.

## Conexão com o artigo anterior

Em um [artigo anterior](/2020/02/09/Hashing-e-Encripitacao.html) expliquei como hashing e criptografia funcionam. O WannaCry usa exatamente esses conceitos:

- **AES** (criptografia simétrica) para criptografar os arquivos rapidamente
- **RSA** (criptografia assimétrica) para proteger a chave AES
- A vítima só consegue a chave RSA se pagar — sem ela, os arquivos são inacessíveis

Entender os fundamentos de criptografia ajuda a entender por que ransomware é tão eficaz e por que backups são a melhor defesa.

## Conclusão

WannaCry e NotPetya mostraram que segurança não é apenas um problema técnico — é um risco de negócio. Ambos os ataques poderiam ter sido evitados ou mitigados com práticas básicas: patching, segmentação, backups e princípio do menor privilégio.

Se você desenvolve software, pense em segurança desde o início. O custo de prevenir é sempre menor do que o custo de remediar.

Caso eu tenha falado alguma besteira, por favor, agradecerei correções e sugestões.
