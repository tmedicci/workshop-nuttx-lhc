---
title: 'Explorando o NuttX com o ESP32: Desenvolva Produtos de Forma Profissional'
author: Tiago Medicci Serrano
description: Neste workshop imersivo exploraremos o empolgante ecossistema do NuttX! Este curso introdutório não apenas oferecerá insights práticos sobre o uso do NuttX, mas também destacará sua compatibilidade com POSIX, tornando-o uma escolha robusta para projetos profissionais.
keywords: IoT, embedded, embarcados, software, RTOS, NuttX, ESP32, firmware.
marp: true
paginate: true
theme: default

---

<style>
  :root {
    --color-fg-default: #000;
    --color-canvas-default: #fff;
    /* ... */
  }
</style>

**Laboratório Hacker de Campinas (LHC), apresenta:**

# Explorando o NuttX com o ESP32

**Desenvolva Produtos de Forma Profissional**

<!-- footer: ![h:50](./images/lhc-pretoebranco.png) Link para esta apresentação: [https://bit.ly/nuttx-workshop](https://bit.ly/nuttx-workshop) -->

<style>
footer {
    display: flex;
    align-items:center;
}
</style>

---

## Para os apressados e apressadas:

[Aqui](#maquina-virtual) está o slide com os links (e instruções) para baixar a máquina virtual para o workshop.

> Nota do autor: *podem ir baixando/instalando/atualizando, mas tentem prestar atenção na apresentação, hein!* :wink:

---

**Laboratório Hacker de Campinas (LHC), apresenta:**

# Explorando o NuttX com o ESP32

**Desenvolva Produtos de Forma Profissional**

---

## LHC?

O Laboratório Hacker de Campinas é *"hackerspace de Campinas que fornece um espaço aberto e comunitário para que entusiastas de tecnologia possam desenvolver seus projetos em áreas como eletrônica, robótica, mecânica, computação, jogos, culinária, artes, ou o que mais a criatividade e o espaço físico disponível permitirem."* (de [lhc.net.br](lhc.net.br))

---

### Workshop no LHC?

Este workshop é uma iniciativa minha - Tiago (membro do LHC desde 2018) - com o apoio de colegas. E, claro, é totalmente gratuito!

Gostou e quer ajudar de alguma forma? Faça uma doação ao LHC (pix para batman@lhc.net.br) e considere tornar-se um associado!

---

## Quem sou eu?

*Prazer, me chamo Tiago Medicci.*

Sou Engenheiro Eletricista e Mestre em Eng. Eletrônica pela Unicamp. Amante de eletrônica e áudio. Maker. Trabalho profissionalmente com o desenvolvimento de produtos e soluções de eletrônica embarcada desde 2014. Atualmente (jan/2024), sou Eng. de Software na Espressif, provendo suporte aos chips da família ESP32 para o NuttX.

---

## O que é o NuttX?

*ChatGPT-generated? Yes* :innocent:

O NuttX é um sistema operacional em tempo real (RTOS) que visa fornecer uma plataforma escalável e eficiente para sistemas embarcados. Ele é baseado no padrão POSIX e é projetado para ser altamente configurável e "portável".

---

### RTOS?

#### Software Embarcado

- Bare-metal;
- Sistemas operacionais:
  - Sistemas operaticionais de tempo real (RTOS);
  - Linux Embarcado;

---

### RTOS (Real Time Operating System)

#### Wikipedia (em tradução livre):

> Sistemas operacionais em tempo real são orientados a eventos e preemptivos, o que significa que o sistema operacional pode monitorar a prioridade relevante de tarefas concorrentes e fazer alterações na prioridade da tarefa. Sistemas orientados a eventos alternam entre tarefas com base em suas prioridades, enquanto sistemas de compartilhamento de tempo alternam a tarefa com base em interrupções de relógio.

https://en.wikipedia.org/wiki/Real-time_operating_system, acesso em 08/01/2024.

---

### RTOSes:

<iframe src="https://en.wikipedia.org/wiki/Comparison_of_real-time_operating_systems" height="80%" width="100%" frameBorder="1"></iframe>

---

## Por que o NuttX?

- Licença: Apache 2.0;
- Small footprint:
  - Suportado do Z80 a RISC-V 64 bits;
- Ampla comunidade;

**E...** :arrow_heading_down:

---

<style>
img[alt~="center"] {
  display: block;
  margin: 0 auto;
}
</style>

![w:100% center](./images/pergunta_app_comum_c.png)

---

```c
#include <stdio.h>

int main() {
    printf("Hello, World!\n");
    return 0;
}
```

![bg left:50% height: 80%](./images/resposta_app_comum_c.png)

---

**Ainda perguntando ao chatGPT...** :thinking:

![w:100% center](./images/pergunta_exemplo_tcp_server.png)

---

![bg top:50% height: 70%](./images/resposta_exemplo_tcp_server.png)

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8080
#define MAX_BUFFER_SIZE 1024

int main() {
    int server_socket, client_socket;
    struct sockaddr_in server_addr, client_addr;
    char buffer[MAX_BUFFER_SIZE];

    // Criação do socket
    if ((server_socket = socket(AF_INET, SOCK_STREAM, 0)) == -1) {
        perror("Erro ao criar o socket");
        exit(EXIT_FAILURE);
    }

    // Configuração do endereço do servidor
    server_addr.sin_family = AF_INET;
    server_addr.sin_addr.s_addr = INADDR_ANY;
    server_addr.sin_port = htons(PORT);

    // Associação do socket ao endereço
    if (bind(server_socket, (struct sockaddr*)&server_addr, sizeof(server_addr)) == -1) {
        perror("Erro ao associar o socket ao endereço");
        exit(EXIT_FAILURE);
    }

    // Coloca o socket em modo de escuta
    if (listen(server_socket, 5) == -1) {
        perror("Erro ao colocar o socket em modo de escuta");
        exit(EXIT_FAILURE);
    }

    printf("Servidor escutando na porta %d...\n", PORT);

    // Aguarda a conexão do cliente
    socklen_t client_addr_len = sizeof(client_addr);
    if ((client_socket = accept(server_socket, (struct sockaddr*)&client_addr, &client_addr_len)) == -1) {
        perror("Erro ao aceitar a conexão do cliente");
        exit(EXIT_FAILURE);
    }

    printf("Cliente conectado: %s\n", inet_ntoa(client_addr.sin_addr));

    // Loop para receber e enviar dados de volta ao cliente
    while (1) {
        ssize_t bytes_received = recv(client_socket, buffer, MAX_BUFFER_SIZE, 0);
        if (bytes_received <= 0) {
            printf("Conexão encerrada pelo cliente.\n");
            break;
        }

        printf("Recebido: %s", buffer);

        send(client_socket, buffer, bytes_received, 0);
        memset(buffer, 0, sizeof(buffer));
    }

    // Fechamento dos sockets
    close(client_socket);
    close(server_socket);

    return 0;
}
```

---

## Por que o NuttX?

- Licença: Apache 2.0;
- Small footprint:
  - Suportado do Z80 a RISC-V 64 bits;
- Ampla comunidade;

**E...**:

---

## Por que o NuttX?

- Licença: Apache 2.0;
- Small footprint:
  - Suportado do Z80 a RISC-V 64 bits;
- Ampla comunidade;
- **Compatível com o padrão POSIX!**

---

### POSIX?

De novo, do wikipedia :memo::

> POSIX (um acrônimo para: Portable Operating System Interface, que pode ser traduzido como Interface Portável entre Sistemas Operativos) é uma família de normas definidas pelo IEEE para a manutenção de compatibilidade entre sistemas operacionais e designada formalmente por IEEE 1003. POSIX define a interface de programação de aplicações (API), juntamente com shells de linha e comando e interfaces utilitárias, para compatibilidade de software com variantes de Unix e outros sistemas operacionais.

https://pt.wikipedia.org/wiki/POSIX, acesso em 13 de janeiro de 2024.

---

### POSIX! :raised_hands:

De novo, do wikipedia:

> **Tem como objetivo garantir a portabilidade do código-fonte de um programa a partir de um sistema operacional que atenda às normas POSIX para outro sistema POSIX, desta forma as regras atuam como uma interface entre sistemas operacionais distintos.**

https://pt.wikipedia.org/wiki/POSIX, acesso em 13 de janeiro de 2024.

---

### POSIX :heart:

<sub>*O Super Trunfo do NuttX!* :punch:</sub>

#### [Reinventing the wheel](https://en.wikipedia.org/wiki/Reinventing_the_wheel) :negative_squared_cross_mark:

Alguém já teve que fazer ~~quase~~ a mesma aplicação que você!

#### [Code reuse](https://en.wikipedia.org/wiki/Code_reuse) :white_check_mark:

Provavelmente, esta aplicação já foi desenvolvida para sistemas operacionais Unix com interfaces POSIX :wink:

---

#### Complete os espaços em branco :writing_hand:

*"Ah, mas o meu programa é muito específico! Ele lê um dado do sensor ________ pela interface ________ e depois manda para ________ via ________."*

OK, mas alguém *provavelmente*:
1. Já implementou um driver para ler o sensor em sistema Linux (incluindo o acesso à interface);
2. Já criou uma aplicação que recebe dados genéricos e os processa (enviando através de alguma interface de rede, por exemplo).

---

##### Por exemplo...

###### Placa de som via Wi-Fi :speaker:

Utilizar o ESP32 como uma placa de áudio externa que recebe pacotes via rede Wi-Fi e reproduzi-lo em um codec de áudio I2S.

**Trabalhoso?** :tired_face:

---

###### Solução :clipboard:

Existe (desde 1996) um protocolo de rede chamado *RTP* (Real-Time Protocol) que é capaz de enviar e receber pacotes de áudio pela rede.

[***RTP Tools***](https://github.com/irtlab/rtptools)

*RTP Tools is a set of small applications that can be used for processing RTP data.*

:arrow_right: Utiliza interfaces POSIX para receber pacotes RTP pela rede

---

###### Quer saber mais? :nerd_face:

Artigo no *embarcados.com.br* :books:

[***NuttX: Criando (ou Copiando!) uma Aplicação para o NuttX***](https://embarcados.com.br/nuttx-criando-ou-copiando-uma-aplicacao-para-o-nuttx/)

---

##### Aliás... :books:

Uma série de artigos em português no *embarcados.com.br*:
###### [*Primeiros Passos com o ESP32 e NuttX*](https://embarcados.com.br/serie/primeiros-passos-com-o-esp32-e-o-nuttx/)

<iframe src="https://embarcados.com.br/serie/primeiros-passos-com-o-esp32-e-o-nuttx/" height="300px" width="100%" frameBorder="0"></iframe>

---

## Espressif ESP32

- Primeira versão lançada oficialmente em 6 de setembro de 2016;
- Sucessor do ESP8266 (*"IoT Game Changer"*);
- Focado em conectividade!
  - Wi-Fi e Bluetooth;

![bg left:50% height: 80%](./images/esp32_block_diagram.png)

---

### ESP32 e o NuttX :computer:

- Primeiro commit em out/2016 (sim, um mês depois do lançamento!) pelo próprio Gregory Nutt (criador do NuttX);
- Em 29 de outubro de 2021 a Espressif anuncia oficialmente o suporte aos OSes NuttX e Zephyr (além, claro, do ESP-IDF, que continua sendo o OS oficial da Espressif para o ESP32);
- ESP32, ESP32-S2, ESP32-S3, ESP32-C3, ESP32-C6 e ESP32-H2 são suportados no NuttX hoje (23/01/2024);

---

### ESP32 e o NuttX :computer:

[Documentação do ESP32 no NuttX](https://nuttx.apache.org/docs/latest/platforms/xtensa/esp32/index.html):

<iframe src="https://nuttx.apache.org/docs/latest/platforms/xtensa/esp32/index.html" height="600px" width="100%" frameBorder="0"></iframe>

---

## Mão na Massa! :construction_worker:

---

### Máquina Virtual <a id="maquina-virtual"></a>

Para facilitar os experimentps, disponibilizei uma máquina virtual já configurada com o ambiente de desenvolvimento do NuttX configurado e testado!

#### Instruções:
1. Baixe e instale o [VirtualBox](https://www.virtualbox.org/wiki/Downloads);
2. Baixe o arquivo disponibilizado **.vdi*: https://bit.ly/nuttx-vm
3. Siga as instruções disponibilizadas na sessão [*"How to Import a VM Into VirtualBox"*](https://www.makeuseof.com/how-to-export-virtualbox-vm-image-to-another-computer/#how-to-import-a-vm-into-virtualbox)

---

*Mas VMs são pesadas, quero fazer tudo na mão!* :nerd_face:

### Ambiente de Desenvolvimento via Docker

Ok, são inúmeros os tutoriais de como configurar o ambiente de build, mas não teremos tempo de fazer estas etapas hoje. Uma dica: a imagem docker que o NuttX utiliza em seu CI pode ser usada sem problemas :wink:

```
docker run -it --rm \
        -v `pwd`:/sources \
        --workdir=/sources \
        --env REMOTE_USER="builder" \
        --env NEW_UID=$(id -u) \
        --env NEW_GID=$(id -g) \
        --device=/dev/ttyUSB0 \
        ghcr.io/apache/nuttx/apache-nuttx-ci-linux
```

---

## Dúvidas, Perguntas e Sugestões :memo:

Este workshop é uma introdução ao NuttX com o ESP32. Muito conteúdo ficou de fora, eu sei, mas espero que tenham gostado!

---

## Referências Gerais

- Página Oficial do NuttX: [https://nuttx.apache.org/](https://nuttx.apache.org/)
- Documentação Oficial do NuttX: [https://nuttx.apache.org/docs/latest/](https://nuttx.apache.org/docs/latest/)
- Kernel do NuttX no GitHub: [https://github.com/apache/nuttx](https://github.com/apache/nuttx)
- Aplicações do NuttX no GitHub: [https://github.com/apache/nuttx-apps](https://github.com/apache/nuttx-apps)
