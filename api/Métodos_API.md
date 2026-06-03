## Métodos da API Java
O que você entende por método? 
Muitos poderiam responder que é uma forma organizada e garantida de se fazer algo (ou alguma outra resposta desse tipo). 
Bem, quando o assunto é programação, essa palavra adquire um significado diferente.
<br>
**"System.out.println", "System.out.printf" e "System.console"** aparecem dizendo: "Somos todos métodos. E com muito orgulho!"
<br>
  Um método, em programação, pode ser entendido como um comando que faz alguma coisa e pode retornar um resultado ou não. 
<br> 
<br>
Estes métodos presentes em Java estão disponíveis em uma biblioteca de recursos de programação chamada de API 
(**Application Programming Interface** ou, em português: **Interface de Programação de Aplicações**). 
<br>
## API 
  Os métodos estão organizados em unidades chamadas classes (você já viu essa palavra antes, lembra?). 
Uma classe pode conter um ou mais métodos.
<br>
Estes métodos podem ser classificados basicamente em dois tipos: métodos **estáticos** e métodos **não-estáticos**. 
### método estático 
É um método executado diretamente a partir do nome de uma classe, sem a necessidade da criação de objetos. 
### método não-estático 
É construído para ser executado a partir de um objeto. Tudo bem, você deve estar pensando “o que um objeto tem a ver com isso?”. 

Conhecer mais sobre objetos está fora do escopo deste curso. Porém outros cursos aqui do IFRS abordarão esta temática em breve. 
Por isso nos concentraremos neste módulo nos métodos estáticos.
<br>
Para se executar um método estático, precisamos basicamente de duas coisas: o *nome do método* e *uma lista de valores de parâmetros*. 
Você pode ver abaixo que a lista de valores de parâmetros deve vir sempre dentro de parênteses.
nome do método ( valores dos parâmetros )
<br>
O nome do método é formado pelo nome da classe onde ele foi definido e pelo seu nome propriamente dito, separados por um ponto final. 
Por exemplo, para executarmos o método sqrt da classe Math (que calcula a raiz quadrada de um número) deveríamos escrever Math.sqrt. 
Quando você escreve System.console está se referindo ao método console da classe System. Simples, não é?
<br>
Mas e os parâmetros? Um parâmetro é um valor que um método precisa para fazer o seu trabalho. Se alguém lhe pedisse para calcular um número ao quadrado, o que você perguntaria? 
Bem, você precisa saber qual é o número, certo? Esse número seria um parâmetro que você deveria saber para efetuar o cálculo. Entendeu?

<br>
Alguns métodos precisam de um parâmetro, outros de dois, outros de mais de dois e alguns de nenhum. Os valores dos parâmetros devem ser separados por vírgulas. 
O método sqrt, por exemplo, calcula a raiz quadrada de um número. Para fazer esse cálculo ele precisa saber qual é o número. 
Assim, ele deve receber um único parâmetro, que é o número do qual desejamos calcular a raiz quadrada. Logo, para efetuar a raiz quadrada de 2 poderíamos escrever:<br>

**Math.sqrt(2)**
<br>
O método sqrt aceita qualquer número como parâmetro, incluindo valores reais. Outro método, pow, também definido na classe Math, permite elevar um número em um expoente qualquer. 
Neste caso, o método precisa de dois parâmetros: o valor da base (o número que será elevado) e o valor do expoente. Assim, se desejamos calcular o número quatro elevado ao cubo, escrevemos:<br>
**Math.pow(4, 3)**
<br>
Alguns métodos não recebem parâmetros, pois não precisam de nenhum valor externo para realizar suas tarefas. 
Um exemplo de método sem parâmetro é o método console da classe System que você já utilizou:<br>
**System.console()**
<br>
Veja que ao escrevê-lo, você obrigatoriamente precisa abrir e fechar parênteses após o nome do método.  
Os parênteses são obrigatórios, mesmo em métodos sem parâmetros. <br>
<br>
Outro exemplo desse tipo é o método random da classe Math que retorna um número aleatório entre 0 e 1.
O último aspecto que devemos estudar sobre métodos é o valor de retorno. Muitos métodos geram um resultado. 
Por exemplo, o método sqrt gera um número double como resultado: a raiz quadrada do número recebido como parâmetro. 
Podemos utilizar esse valor em uma expressão ou armazená-lo em uma variável, como mostrado abaixo:

### double diagonal = Math.sqrt(2)*lado;  // Uso em expressão

### double dezesseis = Math.pow(2, 4);    // Resultado em variável

Alguns métodos não possuem valor de retorno. Nesse caso, deve-se escrevê-lo em uma linha isolada, finalizando a instrução com um ponto-e-vírgula. 
Um exemplo é o método sleep da classe Thread que faz com que o programa fique parado durante um determinado tempo em milissegundos. 
Para executá-lo, teríamos que escrever a instrução mostrada abaixo (considerando um tempo de espera de um segundo):<br>
**Thread.sleep(1000);**
<br>
Ufa! Enfim terminamos. Agora você conhece o necessário para executar alguns métodos estáticos da API Java. 
Você já conheceu alguns poucos deles, mas vamos conhecer outros que podem ser úteis daqui para frente.
Vamos começar pelos métodos da classe Math. Os métodos da classe Math permitem realizar cálculos matemáticos comuns. 
Os principais métodos dessa classe são mostrados na tabela abaixo.


<img width="1820" height="555" alt="image" src="https://github.com/user-attachments/assets/57af61a6-445b-4074-8192-fd1a24230161" />
<img width="1819" height="437" alt="image" src="https://github.com/user-attachments/assets/1c264f56-4955-4ad6-84c6-6361226d95d9" />



O método com a função que talvez pareça menos intuitiva é random. Este método gera um valor double de 0.0 até, mas não incluindo, 1.0. 
O método escolhe esse valor como se fosse um sorteio ou um jogo de dados, ou seja, a cada chamada do método um valor double diferente, 
maior ou igual a 0.0 e menor do que 1.0, é retornado (por isso não há um exemplo de uso dele na tabela).

Observe que os valores devolvidos por random são na verdade números pseudoaleatórios, uma sequência de valores produzida por um cálculo matemático complexo 
(é pseudoaleatório, pois sua sequência pode ser prevista, desde que se conheça a equação). 
Este cálculo usa a hora do dia atual para semear o gerador de números aleatórios, de modo que cada vez que o programa é executado, uma sequência diferente de valores é gerada.

O intervalo de valores produzido diretamente pelo método random muitas vezes é diferente do intervalo de valores necessário em um programa Java em particular. 
Por exemplo, o programa que simula o lançamento de uma moeda talvez exija somente os valores 0 para “cara” e 1 para “coroa”. 
O programa que simula o lançamento de um dado de seis faces exige inteiros aleatórios no intervalo de 1 a 6. 
Para gerar valores em faixas diferentes, precisamos trabalhar com o valor devolvido pelo método random, através de uma expressão aritmética na forma:

(int) (valor inicial + Math.random() * número de valores )

onde valor inicial é o primeiro valor do intervalo e número de valores é o número de valores desejado no intervalo. 
A palavra (int) à frente da expressão faz com que o resultado dela seja convertido para um valor inteiro. 
Logo, isso só será necessário se desejarmos resultados inteiros.

Assim, para gerarmos um valor inteiro no intervalo de 1 a 6 basta escrevermos a expressão:

(int) (1 + Math.random() * 6 )

Como exemplo completo, considere o programa abaixo, que simula cinco lançamentos de um dado de seis faces. 




