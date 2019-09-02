---
layout: post
title: Programacao para redes usando linguagem C
categories: [Redes, C, Sockets]
---

# Programacao para redes

Hoje em dia, e dificil encontrarmos aplicacoes que trabalham isoladamente.
Na maioria das vezes, os projetos ja se iniciam se comunicando com a rede.

O linux possui uma serie de protocolos ja implementados no kernel, sem sombra
de duvidas, os mais utilizados sao o TCP e o UDP.

Para trabalhar com a rede em linguagem C, recorremos a socket API, essa API e
derivada do BSD.

Quando abrimos um socket, que basicamente e um descritor voltado a rede, temos
funcoes que teriamos em um arquivo normal, como, read, write, close, etc. Ao abrir
um socket, mesmo que seja somente para leitura, precisamos espeficipar um endereco, porta
e o protocolo associado ao socket.

# Structs importantes

Struct      | Nome                  | Utilizacao
------------|-----------------------|-----------
sockaddr    |socket address         |Informacoes sobre o endereco do socket
sockaddr_in |socket address internet|Uma forma de enderecar mais facilmente os elementos do socket
in_addr     |internet address       |O proprio endereco internet
hostent     |host entry             |Recebe dados sobre o nome/endereco e protocolos associados
tcphdr      |TCP Header             |Descricao do cabecalho TCP
iphdr       |IP Header              |Descricao do cabecalho IP
ifreq       |Interface data         |Dados e flags da interface de rede

### Cabecalhos que precisam ser incluidos

```c
#include <unistd.h>
#include <errno.h>
#include <netdb.h>
#include <netinet.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
```

# Protocolo TCP

O protocolo TCP/IP inicia uma conexao ponto a ponto, com checagem de erros, integridade e controle
do trafego de rede.

Para exemplificar, vamos criar um servidor e um cliente utilizando linguagem C.
Nao e necessaria programar o cliente, voce pode utulizar um alternativo, aconcelho usar o netcat.

# Desenvolvendo nosso servidor


### Comecamos incluindo os headers necessarios
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <netdb.h>
#include <netinet/in.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
```


### Vamos definir uma porta e um backlog

Definimos a porta e o server backlog utilizando constante.
PS: O server backlog indica quantas conexoes simultaneas nosso servidor escutara, caso tenha mais de 10,
as outras entrarao em uma fila de conexoes.

```c
#define SERVER_PORT 5000
#define SERVER_BACKLOG 10
```
### Criando socket

Declaramos nossa funcao main, criamos nossas estruturas client e server.
Tambem, inicializamos as variaveis sockfd_server, sockfd_client que sao justamente
os descritores de arquivos. Voce pode estar se perguntando porque eles foram inicializadas
como int, a resposta e que a funcao socket, retorna um inteiro, logo, precisamos validar se
o valor retornado e sucesso ou nao. A funcao retorna um valor < 0 caso tenha ocorrido algum
erro.


```c
int main (int argc, char *argv[]){
    struct sockaddr_in client;
    struct sockaddr_in server;

    int sockfd_server, sockfd_client, nbytes;
    int size, visits = 0;
    char msg[1024];

    if((sockfd_server = socket(AF_INET, SOCK_STREAM, 0)) < 0){
        fprintf(stderr, "could not create socket\n");
        exit(1);
    }
```

Utilizamos a funcao memset para limpar nossa estrutura, por padrao, essa estrutura ja vem
preenchida lixo de memoria, para evitarmos problemas, preenchemos seus valores com 0.
Depois disso, inicializamos a estrutura especificamos o protocolo, a porta e o endereco que
meu servidor ira escutar. O INADDR_ANY, indica meu endereco local.

```c
    memset(&server, 0, sizeof(struct sockaddr_in));

    server.sin_family = AF_INET;
    server.sin_port = htons(SERVER_PORT);
    server.sin_addr.s_addr = INADDR_ANY;
```

Com o socket criado e nossa estrutura de dados preenchida, precisamos fazer um bind do endereco
com a porta, obviamente, para isso, utilizamos a funcao bind. Para que nosso servidor possa ouvir
requisicoes, utilizamos a funcao listen, passando o backlog de conexoes que ela atendera.

```c
    if(bind(sockfd_server, (struct sockaddr *)&server, sizeof(struct sockaddr)) < 0){
        fprintf(stderr, "could not create bind\n");
        exit(1);
    }

    if(listen(sockfd_server, SERVER_BACKLOG)){
        fprintf(stderr, "could not create listen\n");
        exit(1);
    }
```

Por ultimo, vamos criar um loop para atender nossos clientes.
```c
    while(1){
        size = sizeof(struct sockaddr_in);

        if((sockfd_client = accept(sockfd_server, (struct sockaddr *)&client, &size)) < 0){
            fprintf(stderr, "could not create accept\n");
            exit(1);
        }
        
        visits++;
        fprintf(stderr, "Connection [%d]\n", visits);
        memset(msg, 0, sizeof(msg));

        if((nbytes = recv(sockfd_client, msg, 50, 0)) < 0){
            fprintf(stderr, "clould not receveid message\n");
            exit(1);
        }

        msg[nbytes] = '\0';

        fprintf(stdout, "Message: %s\n", msg);
        close(sockfd_client);
    }

    close(sockfd_server);
    return 0;
```

# Codigo completo.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <netdb.h>
#include <netinet/in.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>

#define SERVER_PORT 4500
#define SERVER_BACKLOG 10

int
main (int argc, char *argv[]){
    struct sockaddr_in client;
    struct sockaddr_in server;

    int sockfd_server, sockfd_client, nbytes;
    int size, visits = 0;
    char msg[1024];

    if((sockfd_server = socket(AF_INET, SOCK_STREAM, 0)) < 0){
        fprintf(stderr, "could not create socket\n");
        exit(1);
    }

    //clean struct sockaddr_in
    memset(&server, 0, sizeof(struct sockaddr_in));

    server.sin_family = AF_INET;
    server.sin_port = htons(SERVER_PORT);
    server.sin_addr.s_addr = INADDR_ANY;

    if(bind(sockfd_server, (struct sockaddr *)&server, sizeof(struct sockaddr)) < 0){
        fprintf(stderr, "could not create bind\n");
        exit(1);
    }

    if(listen(sockfd_server, SERVER_BACKLOG)){
        fprintf(stderr, "could not create listen\n");
        exit(1);
    }

    while(1){
        size = sizeof(struct sockaddr_in);

        if((sockfd_client = accept(sockfd_server, (struct sockaddr *)&client, &size)) < 0){
            fprintf(stderr, "could not create accept\n");
            exit(1);
        }
        
        visits++;
        fprintf(stderr, "Connection [%d]\n", visits);
        memset(msg, 0, sizeof(msg));

        if((nbytes = recv(sockfd_client, msg, 50, 0)) < 0){
            fprintf(stderr, "clould not receveid message\n");
            exit(1);
        }

        msg[nbytes] = '\0';

        fprintf(stdout, "Message: %s\n", msg);
        close(sockfd_client);
    }

    close(sockfd_server);
    return 0;
}

```

# Desenvolvendo o cliente

Nosso cliente deve se conectar em um endereco preestabelecido, e enviar uma mensagem.

A programacao do cliente e bem parecida com o servidor, comecamos incluindo os headers
necessarios.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <netdb.h>
#include <netinet/in.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
```

Tambem definimos a porta que o servidor que nos vamos se conectar esta ouvindo.

```c
#define SERVER_PORT 5000
```

Comecamos a definir nossa funcao main, criamos uma estrutura para o cliente, e a estrutura
hostent que e responsavel por tentar resolver o endereco, caso seja passado um DNS ao inves de
um endereco IP.

```c
int
main (int argc, char *argv[]){
    struct sockaddr_in client;
    struct hostent *h;
    int sockfd;

    char *message = "Hello world\n";

    if(argc != 2){
        fprintf(stderr, "Use: ./client <server>\n");
        exit(1);
    }
    
    if((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0 ){
        perror("socket");
        exit(1);

    }

    if((h = gethostbyname(argv[1])) == NULL){
        perror("gethostbyname");
        exit(1);
    }
```

Preenchendo a estrutura do cliente.

```c
    client.sin_family = AF_INET;
    client.sin_port = htons(SERVER_PORT);
    client.sin_addr = *((struct in_addr *)h->h_addr);
```

A funcao connect e validada da mesma forma que fizemos com o a funcao socket. Caso seja retornado
um valor < 0, siginifica que nos nao conseguimos nos conectar com o servidor e a funcao e encerrada.

```c
    if(connect(sockfd, (struct sockaddr *)&client, sizeof(client)) < 0){
        perror("connect");
        exit(1);
    }
```

Enviando mensagem para o servidor usando a funcao send.

```c
    send(sockfd, message, strlen(message), 0);
    close(sockfd);
    return 0;
```


# Codigo completo

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <netdb.h>
#include <netinet/in.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>

#define SERVER_PORT 4500

int
main (int argc, char *argv[]){
    struct sockaddr_in client;
    struct hostent *h;
    int sockfd;

    char *message = "Hello world\n";

    if(argc != 2){
        fprintf(stderr, "Use: ./client <server>\n");
        exit(1);
    }
    
    if((sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0 ){
        perror("socket");
        exit(1);

    }

    if((h = gethostbyname(argv[1])) == NULL){
        perror("gethostbyname");
        exit(1);
    }

    client.sin_family = AF_INET;
    client.sin_port = htons(SERVER_PORT);
    client.sin_addr = *((struct in_addr *)h->h_addr);

    if(connect(sockfd, (struct sockaddr *)&client, sizeof(client)) < 0){
        perror("connect");
        exit(1);
    }

    send(sockfd, message, strlen(message), 0);
    close(sockfd);
    return 0;
}
```
