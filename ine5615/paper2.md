# Exercicio 2
___
## Disciplina: INE 5615 – Redes de Computadores
## Artur Silva Muniz Junior - 16101095
___

### 1 - Explique o funcionamento:
#### a) Do protocolo ARP
O protocolo ARP consiste em dois tipos de pacotes.

O primeiro, chamado ARP Query, consulta à rede o endereço fisico associado um endereço logico.

Funciona da seguinte forma, quando um nó da rede deseja se comunicar com outro, deve montar um quadro
contendo o MAC do nó de destino.
Para isso ele consulta uma tabela (o cache ARP) com a finalidade de averiguar se já conhece o MAC associado ao IP de destino.
Caso encontrado o MAC, monta o quadro e o envia.
Porem, caso contrario, inicia o protocolo ARP com a finalidade de descobrir o MAC destino.
É montado um quadro contendo, entre outras informações, o MAC de origem e o IP que se deseja consultar o MAC.
Anota-se como MAC de destino o endereço de broadcast (FF-FF-FF-FF-FF-FF), assim todos os adaptadores da sub-rede receberam o quadro.
Essa é a ARP Query.

Ao receber um ARP Query, o nó verifica se o IP solicitado é o mesmo que o seu IP.
Caso afirmativo, e somente nesse caso, é montado um pacote APR Reply que devolve para o remetente da ARP Query o seu endereço MAC

Agora então o nó que iniciou o protocolo pode montar seus quadros normalmente, com o MAC obtido.

#### b) Do cache ARP
O cache ARP consiste de um mapeamento em memoria presente em todos os equipamentos da camada de rede.
É anotado em formato de tabela e releciona endereços logicos aos repectivos endereços fisicos.
Importante notar que o ARP apenas guarda o MAC de endereços da sub-rede e que, em determinado momento, nem todos endereços de uma sub-rede se encontrarão na tabela.
Alem disso, registros no cache podem expirar e serem removidos.

### 2) Visualizar cache ARP
#### a) Saída do comando `arp -a`

```
_gateway (192.168.100.1) em 40:9b:cd:29:f2:5c [ether] em wlp3s0
```

#### b) Qual endereço IP está presente na tabela ARP: da sua máquina ou do gateway padrão?
Do gateway padrão `192.168.100.1`.

### 3) Capturando tráfego próprio para verificar o ARP em ação
Em anexo.

### 4) Usando o seu tráfego (arquivo seunome-ARP), responda as perguntas.
#### a) Indique a linha do seu tráfego usada para o ARP Request.
Linha 29.

```
29	22.385769856	24:0a:64:94:10:73	ff:ff:ff:ff:ff:ff	ARP	42	Who has 192.168.100.1? Tell 192.168.100.104
```

#### b) Quais são os valores hexadecimais para os endereços de fonte (Source) e destino (Destination) no quadro Ethernet II contendo a mensagem de ARP Request? Explique o valor do endereço de destino (Destination) no quadro Ethernet II: o que significa esse valor?

Fonte: `24:0a:64:94:10:73`
Destino: `ff:ff:ff:ff:ff:ff`

O valor de destino coresponde ao endereço de broadcast e indica que todos os nós da sub-rede são destinatarios do quadro.

#### c) Escreva os endereços: Sender MAC Address, Sender IP Address, Target MAC Address, Target IP Address.
Sender MAC address: `24:0a:64:94:10:73`
Sender IP address: `192.168.100.104`
Target MAC address: `00:00:00:00:00:00`
Target IP address: `192.168.100.1`

#### d) Qual endereço tem um valor zero? Explique o motivo.
O Target MAC address, pois é desconhecido e justamente o que queremos descobrir com o protocolo ARP.

#### e) Indique a linha do seu tráfego usada para o ARP Reply.
Linha 30.

```
30	22.386831547	40:9b:cd:29:f2:5c	24:0a:64:94:10:73	ARP	42	192.168.100.1 is at 40:9b:cd:29:f2:5c
```

#### f) Quais são os valores hexadecimais para os endereços de fonte (Source) e destino (Destination) no quadro Ethernet II contendo a mensagem de ARP Reply?
Fonte: `40:9b:cd:29:f2:5c`
Destino: `24:0a:64:94:10:73`

#### g) Escreva os endereços: Sender MAC Address, Sender IP Address, Target MAC Address, Target IP Address.

Sender MAC address: `40:9b:cd:29:f2:5c`
Sender IP address: `192.168.100.1`
Target MAC address: `24:0a:64:94:10:73`
Target IP address: `192.168.100.104`
