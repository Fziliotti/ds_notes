# Communicação

## Abaixo os Sockets!

O desenvolvimento de sistemas distribuídos usando diretamente Sockets como forma de comunicação entre componentes não é para os fracos de coração.
Sua grande vantagem está no **acesso baixo nível à rede**, e todo o ganho de desempenho que isso pode trazer.
Suas desvantagens, entretanto, são várias:

* interface de "arquivo" para se ler e escrever bytes;
* controle de fluxo de "objetos" é por conta da aplicação, isto é, a aplicação precisa sinalizar quantos bytes serão escritos de um lado, para que o outro saiba quanto ler para obter um "objeto" correto;
* logo, a serialização e desserialização de objetos é também por conta da aplicação;
* tratamento de desconexões e eventuais reconexões também é gerenciado pela aplicação, e nem a tão famosa confiabilidade do TCP ajuda.

### Representação de dados

Enquanto se poderia argumentar que algumas destas desvantagens podem ser descartadas em função da discussão de incluir ou não API na comunicação [fim-a-fim](http://web.mit.edu/Saltzer/www/publications/endtoend/endtoend.pdf), é certo que algumas funcionalidades são ubíquas em aplicações distribuídas.
Uma delas é a serialização de dados complexos.
Imagine-se usando um tipo abstrato de daados com diversos campos, incluindo valores numéricos de diversos tipos, strings, aninhamentos, tudo somando vários KB.
Você terá que se preocupar com diversos fatores na hora de colocar esta estrutura *no fio*:

* tipos com definição imprecisa
  * Inteiro: 16, 32, 64 ... bits?
* ordem dos bytes
  * little endian?
    * Intel x64, 
    * IA-32
  * big endian?
    * IP
    * SPARC (< V9), 
    * Motorola, 
    * PowerPC
  * bi-endian
    * ARM, 
    * MIPS, 
    * IA-64
* Representação de ponto flutuante
* Conjunto de caracteres
* Alinhamento de bytes
* Linguagem mais adequada ao problema e não à API socket
  * Classe x Estrutura
* Sistema operacional
  * crlf (DOS) x lf (Unix)
* fragmentação <br>
  [![Fragmentação](images/ipfrag.png)](http://www.acsa.net/IP/)

Uma abordagem comumente usada é a representação em formato textual "amigável a humanos".
Veja o exemplo de como o protocolo HTTP requisita e recebe uma página HTML.
```HTML
telnet www.google.com 80
Trying 187.72.192.217...
Connected to www.google.com.
Escape character is '^]'.
GET / HTTP/1.1
host: www.google.com

```
As linhas 5 e 6 são entradas pelo cliente para requisitar a página raiz do sítio [www.google.com](https://www.google.com).
A linha 7, vazia, indica ao servidor que a requisição está terminada.

Em resposta a esta requisição, o servidor envia o seguinte, em que as primeiras linhas trazem metadados da página requisitada e, após a linha em branco, vem a resposta em HTML à requisição.

```HTML
HTTP/1.1 302 Found
Location: http://www.google.com.br/?gws_rd=cr&ei=HTDqWJ3BDYe-wATs_a3ACA
Cache-Control: private
Content-Type: text/html; charset=UTF-8
P3P: CP="This is not a P3P policy! See https://www.google.com/support/accounts/answer/151657?hl=en for more info."
Date: Sun, 09 Apr 2017 12:59:09 GMT
Server: gws
Content-Length: 262
X-XSS-Protection: 1; mode=block
X-Frame-Options: SAMEORIGIN
Set-Cookie: NID=100=NB_AruuFWL0hXk2-h7VDduHO_UkjAr6RaqgG7VbccTsfLzFfhxEKx21Xpa2EH7IgshgczE9vU4W1TyKsa07wQeuZosl5DbyZluR1ViDRf0C-5lRpd9cCpCD5JXXjy-UE; expires=Mon, 09-Oct-2017 12:59:09 GMT; path=/; domain=.google.com; HttpOnly

<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>302 Moved</TITLE></HEAD><BODY>
<H1>302 Moved</H1>
The document has moved
<A HREF="http://www.google.com.br/?gws_rd=cr&amp;ei=HTDqWJ3BDYe-wATs_a3ACA">here</A>.
</BODY></HTML>
```

Representações textuais são usadas em diversos protocolos como SMTP, POP, e telnet.
Algumas destas representações seguem padrões formalizados, o que facilita a geração e interpretação dos dados. 
Dois padrões bem conhecidas são XML e JSON.

[XML](https://xml.org) é o acrônimo para *Extensible Markup Language*, ou seja, uma linguagem marcação que pode ser estendida para representar diferentes tipos de informação.
A HTML, por exemplo, é uma instância de XML destinada à representação de hipertexto (A bem da verdade, XML foi uma generalização de HTML).

Por exemplo, para representarmos os dados relativos à uma pessoa, podemos ter uma instância XML assim:

```xml
<person>
    <name>John Doe</name>
    <id>112234556</id>
    <email>jdoe@example.com</email>
    <telephones>
       <telephone type="mobile">123 321 123</telephone>
       <telephone type="home">321 123 321</telephone>
    </telephones>
</person>
```

Uma das grandes vantagens do uso de XML é a possibilidade de se formalizar o que pode ou não estar em um arquivo para um certo domínio utilizando um [XML *Domain Object Model*](https://docs.microsoft.com/pt-br/dotnet/standard/data/xml/xml-document-object-model-dom). Há, por exemplo, modelos para representação de documentos de texto, governos eletrônicos, representação de conhecimento, [etc](http://www.xml.org/).
Sua maior desvantagem é que é muito verborrágico e por vezes complicado de se usar, abrindo alas para o seu mais famoso concorrente, JSON.


[JSON](http://json.org/) é o acrônimo de *Javascript Object Notation*, isto é, o formato para representação de objetos da linguagem Javascript.
Devido à sua simplicidade e versatilidade, entretanto, foi adotado como forma de representação de dados em sistemas desenvolvidos nas mais diferentes linguagens.
O mesmo exemplo visto anteriormente, em XML, é representado em JSON assim:

```json
{
    "name": "John Doe",
    "id": 112234556,
    "email": "jdoe@example.com",
    "telephones": [
        { "type": "mobile", "number": "123 321 123"},
        { "type": "home", "number": "321 123 321"},
    ]
}
```

Em Python, por exemplo, JSON são gerados e interpretados nativamente, sem a necessidade de *frameworks* externos, facilitando seu uso.
Mas de fato, a opção final por XML ou JSON é questão de preferência, uma vez que os dois formatos são, de fato, equivalentes na questão da representação de informação.

Outros formatos, binários, oferecem vantagens no uso de espaço para armazenar e transmitir dados, e por isso são frequentemente usados como forma de *serialização* de dados em sistemas distribuídos, isto é, na transformação de TAD para sequências de bytes que seguirão "no fio".

* ASN.1 (Abstract Syntax Notation), pela ISO
* XDR (eXternal Data Representation)
* Java serialization
* Google Protocol Buffers
* Thrift

ASN.1 e XDR são de interesse histórico, mas não os discutiremos aqui.
Quanto à serialização feita nativamente pelo Java, por meio de `ObjectOutputStreams`, como neste [exemplo](https://www.tutorialspoint.com/java/java_serialization.htm), embora seja tentadora para quem usa Java, é necessário saber que ela é restrita à JVM e que usa muito espaço, embora minimize riscos de uma desserialização para uma classe diferente.

Outras alternativas, com codificações binárias são interessantes, dentre elas, ProtoBuffers e Thrift.

#### ProtoBuffers

Nas palavras dos [criadores](https://developers.google.com/protocol-buffers/),
> Protocol buffers are a language-neutral, platform-neutral extensible mechanism for serializing structured data.

Por meio de protobuffers, é possível estruturar dados e gerar o código correspondente em diversas linguagens, for forma compartilhável entre as mesmas. Veja o exemplo a seguir, que especifica os dados referentes a uma pessoa. 
Observe a presença de campos de preenchimento opcional (**optional**), de enumerações (**enum**), e de coleções (**repeated**).

```protobuf
message Person {
	required string name = 1;
	required int32 id = 2;
	optional string email = 3;
	enum PhoneType {
		MOBILE = 0;
		HOME = 1;
		WORK = 2;
	}
	message PhoneNumber {
		required string number = 1;
		optional PhoneType type = 2 [default = HOME];
	}
	repeated PhoneNumber phone = 4;
}
```

Com tal definição é possível gerar código como o seguinte, em C++, que serializa os dados para escrita em um arquivo...

```c++
Person person;
person.set_name("John Doe");
person.set_id(1234);
person.set_email("jdoe@example.com");
fstream output("myfile", ios::out | ios::binary);
person.SerializeToOstream(&output);
```

e lê do arquivo e desserializa para hidratar um novo objeto.

```c++
fstream input("myfile", ios::in | ios::binary);
Person person;
person.ParseFromIstream(&input);
cout << "Name: " << person.name() << endl;
cout << "E-mail: " << person.email() << endl;
```

De acordo com *benchmarks* do próprio [projeto](https://developers.google.com/protocol-buffers/docs/overview), a operação em XML seria mais órdens de grandeza mais lenta e ocuparia mais espaço.

> When this message is encoded to the protocol buffer binary format, it would probably be 28 bytes long and take around 100-200 nanoseconds to parse. The XML version is at least 69 bytes if you remove whitespace, and would take around 5,000-10,000 nanoseconds to parse.

## Invocação Remota de Procedimentos - RPC

### Estudo de Caso RPC: gRPC

gRPC é um framework para invocação remota de procedimentos multi-linguagem e sistema operacional, usando internamente pelo Google há vários anos para implementar sua arquitetura de micro-serviços.
Inicialmente desenvolvido pelo Google, o gRPC é hoje de código livre encubado pela Cloud Native Computing Foundation.

O sítio [https://grpc.io](https://grpc.io) documenta muito bem o gRPC, inclusive os [princípios](https://grpc.io/blog/principles/) que nortearam seu projeto.

O seu uso segue, em linhas gerais, o modelo discutido nas seções anteriores, isto é, inicia-se pela definição de estruturas de dados e serviços, "compila-se" a definição para gerar stubs na linguagem desejada, e compila-se os stubs juntamente com os códigos cliente e servidor para gerar os binários correspondentes.
Vejamos a seguir um tutorial passo a passo, em Java, baseado no [quickstart guide](https://grpc.io/docs/quickstart/java.html).

#### Instalação

Os procedimentos de instalação dependem da linguagem em que pretende usar o gRPC, tanto para cliente quanto para servidor.
No caso do **Java**, **não há instalação propriamente dita**.

#### Exemplo Java

Observe que o repositório base apontado no tutorial serve de exemplo para diversas linguagens e diversos serviços, então sua estrutura é meio complicada. Nós nos focaremos aqui no exemplo mais simples, uma espécie de "hello word" do RPC.

##### Pegando o código
Para usar os exemplos, você precisa clonar o repositório com o tutorial, usando o comando a seguir.


```bash
git clone -b v1.19.0 https://github.com/grpc/grpc-java
```

Uma vez clonado, entre na pasta de exemplo do Java e certifique-se que está na versão 1.19, usada neste tutorial.

```bash
cd grpc-java\examples
git checkout v1.19.0
```

##### Compilando e executando
O projeto usa [gradle](https://gradle.org/) para gerenciar as dependências. Para, use o *wrapper* do gradle como se segue.

```bash
./gradlew installDist
```

Caso esteja na UFU, coloque também informação sobre o proxy no comando.

```bash
./gradlew -Dhttp.proxyHost=proxy.ufu.br -Dhttp.proxyPort=3128 -Dhttps.proxyHost=proxy.ufu.br -Dhttps.proxyPort=3128 installDist
```

Como quando usamos sockets diretamente, para usar o serviço definido neste exemplo, primeiros temos que executar o servidor.

```bash
./build/install/examples/bin/hello-world-server
```

Agora, em **um terminal distinto** e a partir da mesma localização, execute o cliente, quantas vezes quiser.

```bash
./build/install/examples/bin/hello-world-client
```

##### O serviço

O exemplo não é muito excitante, pois tudo o que o serviço faz é enviar uma saudação aos clientes.
O serviço é definido no seguinte arquivo `.proto`, localizado em `./src/main/proto/helloworld.proto`.

```protobuf
message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}


// The greeting service definition.
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}
```

No arquivo, inicialmente são definidas duas mensagens, usadas como requisição (cliente para servidor) e outra como resposta (servidor para cliente) do serviço definido em seguida.

A mensagem `HelloRequest` tem apenas um campo denominado `name`, do tipo `string`. Esta mensagem conterá o nome do cliente, usado na resposta gerada pelo servidor.

A mensagem `HelloReply` também tem um campo do tipo `string`, denominado `message`, que conterá a resposta do servidor.

O serviço disponível é definido pela palavra chave `service`e de nome `Greeter`; é importante entender que este nome será usado em todo o código gerado pelo compilador gRPC e que se for mudado, todas as referências ao código gerado devem ser atualizadas.

O serviço possui apenas uma operação, `SayHello`, que recebe como entrada uma mensagem `HelloRequest` e gera como resposta uma mensagem `HelloReply`.
Caso a operação precisasse de mais do que o conteúdo de `name` para executar, a mensagem `HelloRequest` deveria ser estendida, pois não há passar mais de uma mensagem para a operação.
Por outro lado, embora seja possível passar zero mensagens, esta não é uma prática recomendada.
Isto porquê caso o serviço precisasse ser modificado no futuro, embora seja possível estender uma mensagem, não é possível modificar a assinatura do serviço. 
Assim, caso não haja a necessidade de se passar qualquer informação para a operação, recomenda-se que seja usada uma mensagem de entrada vazia, que poderia ser estendida no futuro.
O mesmo se aplica ao resultado da operação.

Observe também que embora o serviço de exemplo tenha apenas uma operação, poderia ter múltiplas.
Por exemplo, para definir uma versão em português da operação `SayHello`, podemos fazer da seguinte forma.

```protobuf
message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}

message OlaRequest {     // <<<<<====
  string name = 1;
}

message OlaReply {       // <<<<<====
  string message = 1;
}

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
  rpc DigaOla (OlaRequest) returns (OlaReply) {}// <<<<<====
}
...
```

Observe que a nova operação recebe como entrada  mensagens `OlaRequest` e `OlaReply`, que tem definições exatamente iguais a `HellorRequest` e `HelloReply`.
Logo, em vez de definir novas mensagens, poderíamos ter usado as já definidas. Novamente, esta não é uma boa prática, pois caso fosse necessário evoluir uma das operações para atender a novos requisitos e estender suas mensagens, não será necessário tocar o restante do serviço.
Apenas reforçando, é boa prática definir *requests* e *responses* para cada método, a não ser que não haja dúvida de que serão para sempre iguais.


##### Implementando um serviço

Agora modifique o arquivo `.proto` como acima, para incluir a operação `DigaOla`, recompile e reexecute o serviço.
Não dá certo, não é mesmo? Isto porquê você adicionou a definição de uma nova operação, mas não incluiu o código para implementá-la.
Façamos então a modificação do código, começando por `./src/main/java/io/grpc/examples/helloworld/HelloWorldServer.java`.
Este arquivo define a classe que **implementa** o serviço `Greeter`, `GreeterImpl`, com um método para cada uma das operações definidas. 
Para confirmar, procure por `sayHello`para encontrar a implementação de `SayHello`; observe que a diferença do `casing` vem das boas práticas de Java, de definir métodos e variáveis em *Camel casing*.

Para que sua versão estendida do serviço `Greeter` funcione, defina um método correspondendo à `DigaOla`, sem consultar o código exemplo abaixo, mas usando o código de `sayHello` como base; não se importe por enquanto com os métodos sendo invocados.
Note que os `...` indicam que parte do código, que não sofreu modificações, foi omitido.

```java
...
private class GreeterImpl extends GreeterGrpc.GreeterImplBase {
...

  @Override
  public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
      ...
  }

  @Override
  public void digaOla(OlaRequest req, StreamObserver<OlaReply> responseObserver) {
    OlaReply reply = 
      OlaReply.newBuilder().setMessage("Ola " + req.getName()).build();
    responseObserver.onNext(reply);
    responseObserver.onCompleted();
  }
}
```

Se você recompilar e reexecutar o código, não perceberá qualquer mudança na saída do programa. Isto porquê embora tenha definido um novo serviço, você não o utilizou. Para tanto, agora modifique o cliente, em `src/main/java/io/grpc/examples/helloworld/HelloWorldClient.java`, novamente se baseando no código existente e não se preocupando com "detalhes".

```java
public void greet(String name) {
  logger.info("Will try to greet " + name + " ...");
...
  OlaRequest request2 = OlaRequest.newBuilder().setName(name).build();
  OlaReply response2;
  try {
    response2 = blockingStub.digaOla(request2);
  } catch (StatusRuntimeException e) {
    logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
   return;
  }
  logger.info("Greeting: " + response2.getMessage());
}
```

Agora sim, você pode reexecutar cliente e servidor.

```bash
./gradlew installDist
./build/install/examples/bin/hello-world-server &
./build/install/examples/bin/hello-world-client
```

Percebeu como foi fácil adicionar uma operação ao serviço? Agora nos foquemos nos detalhes.

##### Stub do servidor

* Como criar o servidor
* Como definir o serviço
* Como "startar" o servidor.

##### Stub do cliente

* Stub bloqueante
* Stub não bloqueante

##### IDL gRPC

Outras características da IDL do gRPC

* Tipos básicos
  * bool: boolean (true/false)
  * double: 64-bit; ponto-flutuante 
  * float: 32-bit; ponto-flutuante 
  * i32: 32-bit; inteiro sinalizado 
  * i64: 64-bit; inteiro sinalizado
  * siXX: signed
  * uiXX: unsigned
  * sfixedXX: codificação de tamanho fixo
  * bytes: 8-bit; inteiro sinalizado
  * string: string UTF-8 ou ASCII 7-bit
  * Any: tipo indefinido

* [Diferentes traduções](https://developers.google.com/protocol-buffers/docs/proto3)

* Coleções
Defina e implemente uma operação `DigaOlas` em que uma lista de nomes é enviada ao servidor e tal que o servidor responda com uma longa string cumprimentando todos os nomes, um ap;os o outro.

* *Streams*
  - Do lado do servidor

  ```java
   List<String> listOfHi = Arrays.asList("e aih", "ola", "ciao", "bao", "howdy", "s'up");

   @Override
   public void digaOlas(OlaRequest req, StreamObserver<OlaReply> responseObserver) {
   for (String hi: listOfHi)
   {
     OlaReply reply = OlaReply.newBuilder().setMessage(hi + ", " req.getName()).build();
     responseObserver.onNext(reply);
   }
   responseObserver.onCompleted();
   }
  ```
  - Do lado do cliente
  
  ```java
   OlaRequest request = OlaRequest.newBuilder().setName(name).build();
   try {
       Iterator<OlaReply> it = blockingStub.digaOlas(request);
       while (it.hasNext()){
         OlaReply response = it.next();
         logger.info("Greeting: " + response.getMessage());
       }
    } catch (StatusRuntimeException e) {
       logger.log(Level.WARNING, "RPC failed: {0}", e.getStatus());
       return;
    }
  ```




#### Exemplo Python

```bash
apt-get install python3
apt-get install python3-pip
python3 -m pip install --upgrade pip
python3 -m pip install grpcio
python3 -m pip install grpcio-tools

git clone -b v1.10.x https://github.com/grpc/grpc
cd grpc/examples/python/helloworld
python3 greeter\_server.py
python3 greeter\_client.py
```

Para recompilar os stubs, faça

```bash
python3 -m grpc_tools.protoc -I../../protos --python_out=. --grpc_python_out=. ../../protos/helloworld.proto
```

Modifique o servidor

```Python
def DigaOla(self, request, context):
	return helloworld_pb2.OlaReply(message='Ola, %s!' + request.name)
```

Modifique o cliente

```Python
response = stub.DigaOla(helloworld_pb2.OlaRequest(name='zelelele'))
print("Greeter client received: " + response.message)
```

### Estudo de Caso RPC: Thrift

[Thrift](https://thrift.apache.org/)

#### Instalação

* [Baixe](http://www.apache.org/dyn/closer.cgi?path=/thrift/0.10.0/thrift-0.10.0.tar.gz) e compile o thrift
* ou instale-o usando apt-get, por exemplo. `apt-get install thrift-compiler`
* execute "thrift" na linha de comando.
* Para thrift com Java, também precisarão dos seguintes arquivos
  * [slf4j](http://mvnrepository.com/artifact/org.slf4j/slf4j-api/1.7.21)
  * [libthrift0.9.3.jar](https://sites.google.com/site/lasaro/sistemasdistribuidos)
  * coloque-os na pasta `jars`



#### IDL Thrift

*  Tipos básicos
    * bool: boolean (true/false)
    * byte: 8-bit; inteiro sinalizado
	* i16: 16-bit; inteiro sinalizado
	* i32: 32-bit; inteiro sinalizado
	* i64: 64-bit; inteiro sinalizado
	* double: 64-bit; ponto-flutuante 
	* string: string UTF-8
	* binary: sequência de bytes
* Estruturas
```thrift
struct Example {
    1:i32 number,
    2:i64 bigNumber,
    3:double decimals,
    4:string name="thrifty"
}
```	
* Serviços
```thrift
service ChaveValor {
    void set(1:i32 key, 2:string value),
    string get(1:i32 key) throws (1:KeyNotFound knf),
    void delete(1:i32 key)
}
```
* **Não se pode retornar NULL!!!**
* Exceções
```thrift
exception KeyNotFound {
   1:i64 hora r,
   2:string chaveProcurada="thrifty"
}
```
*  Containers
    * List
	* Map
	* Set


Exemplo: chavevalor.thrift

```Thrift
namespace java chavevalor
namespace py chavevalor


exception KeyNotFound
{
}


service ChaveValor
{
    string getKV(1:i32 key) throws (1:KeyNotFound knf),
    bool setKV(1:i32 key, 2:string value),
    void delKV(1:i32 key)
}  
``` 	

Compilação

`thrift --gen java chavevalor.thrift`

`thrift --gen py chavevalor.thrift`

ChaveValorHandler.java
```Java
namespace java chavevalor
namespace py chavevalor


exception KeyNotFound
{
}


service ChaveValor
{
    string getKV(1:i32 key) throws (1:KeyNotFound knf),
    bool setKV(1:i32 key, 2:string value),
    void delKV(1:i32 key)
}  
 	
package chavevalor;

import org.apache.thrift.TException;
import java.util.HashMap;
import chavevalor.*;

public class ChaveValorHandler implements ChaveValor.Iface {
   private HashMap<Integer,String> kv = new HashMap<>();
   @Override
   public String getKV(int key) throws TException {
       if(kv.containsKey(key))
          return kv.get(key);
       else
          throw new KeyNotFound();
   }
   @Override
   public boolean setKV(int key, String valor) throws TException {
       kv.put(key,valor);
       return true;
   }
   @Override
   public void delKV(int key) throws TException {
       kv.remove(key);
   }    
}
```

#### Arquitetura 

* Runtime library -- componentes podem ser selecionados em tempo de execução e implementações podem ser trocadas
* Protocol -- responsável pela serializaçãoo dos dados
    * TBinaryProtocol
	* TJSONProtocol
	* TDebugProtocol
	* ...
* Transport -- I/O no ``fio''
    * TSocket
	* TFramedTransport (non-blocking server)
	* TFileTransport
	* TMemoryTransport
* Processor -- Conecta protocolos de entrada e saída com o \emph{handler}
		
* Handler -- Implementação das operações oferecidas
* Server -- Escuta portas e repassa dados (protocolo) para o processors
    * TSimpleServer
	* TThreadPool
	* TNonBlockingChannel



#### Classpath

```bash
javac  -cp jars/libthrift0.9.3.jar:jars/slf4japi1.7.21.jar:gen-java  -d . *.java 
	
java -cp jars/libthrift0.9.3.jar:jars/slf4japi1.7.21.jar:gen-java:. chavevalor.ChaveValorServer
	
java -cp jars/libthrift0.9.3.jar:jars/slf4japi1.7.21.jar:gen-java:. chavevalor.ChaveValorClient	
```

#### Referências

[Tutorial](http://thrift-tutorial.readthedocs.org/en/latest/index.html)


### Estudo de Caso RPC: RMI

<h1>TODO</h1>
