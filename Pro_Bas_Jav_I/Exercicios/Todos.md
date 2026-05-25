## 1. Subtração entre dois números

```
import java.util.Scanner;

public class Exercicio1 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double num1, num2, subtracao;

        System.out.print("Digite o primeiro número: ");
        num1 = entrada.nextDouble();

        System.out.print("Digite o segundo número: ");
        num2 = entrada.nextDouble();

        subtracao = num1 - num2;

        System.out.println("Resultado da subtração: " + subtracao);
    }
}
```

## 2. Divisão entre dois números

```java
import java.util.Scanner;

public class Exercicio2 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double num1, num2, divisao;

        System.out.print("Digite o primeiro número: ");
        num1 = entrada.nextDouble();

        System.out.print("Digite o segundo número: ");
        num2 = entrada.nextDouble();

        divisao = num1 / num2;

        System.out.println("Resultado da divisão: " + divisao);
    }
}

```

## 3. Média aritmética de três notas

```
import java.util.Scanner;

public class Exercicio3 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double nota1, nota2, nota3, media;

        System.out.print("Digite a primeira nota: ");
        nota1 = entrada.nextDouble();

        System.out.print("Digite a segunda nota: ");
        nota2 = entrada.nextDouble();

        System.out.print("Digite a terceira nota: ");
        nota3 = entrada.nextDouble();

        media = (nota1 + nota2 + nota3) / 3;

        System.out.println("Média aritmética: " + media);
    }
}

```

## 4. Produto com desconto de 10%

```
import java.util.Scanner;

public class Exercicio4 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double preco, desconto, novoPreco;

        System.out.print("Digite o preço do produto: ");
        preco = entrada.nextDouble();

        desconto = preco * 0.10;
        novoPreco = preco - desconto;

        System.out.println("Novo preço: " + novoPreco);
    }
}
```

## 5. Salário com gratificação e imposto

```
import java.util.Scanner;

public class Exercicio5 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double salarioBase, gratificacao, imposto, salarioFinal;

        System.out.print("Digite o salário base: ");
        salarioBase = entrada.nextDouble();

        gratificacao = salarioBase * 0.05;
        imposto = salarioBase * 0.07;

        salarioFinal = salarioBase + gratificacao - imposto;

        System.out.println("Salário a receber: " + salarioFinal);
    }
}
```

## 6. Salário com comissão

```
import java.util.Scanner;

public class Exercicio6 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double salarioFixo, vendas, comissao, salarioFinal;

        System.out.print("Digite o salário fixo: ");
        salarioFixo = entrada.nextDouble();

        System.out.print("Digite o valor das vendas: ");
        vendas = entrada.nextDouble();

        comissao = vendas * 0.04;
        salarioFinal = salarioFixo + comissao;

        System.out.println("Comissão: " + comissao);
        System.out.println("Salário final: " + salarioFinal);
    }
}
```

## 7. Média ponderada

```
import java.util.Scanner;

public class Exercicio7 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double nota1, nota2, nota3;
        double peso1, peso2, peso3;
        double media;

        System.out.print("Digite a primeira nota: ");
        nota1 = entrada.nextDouble();

        System.out.print("Digite o peso da primeira nota: ");
        peso1 = entrada.nextDouble();

        System.out.print("Digite a segunda nota: ");
        nota2 = entrada.nextDouble();

        System.out.print("Digite o peso da segunda nota: ");
        peso2 = entrada.nextDouble();

        System.out.print("Digite a terceira nota: ");
        nota3 = entrada.nextDouble();

        System.out.print("Digite o peso da terceira nota: ");
        peso3 = entrada.nextDouble();

        media = ((nota1 * peso1) + (nota2 * peso2) + (nota3 * peso3))
                / (peso1 + peso2 + peso3);

        System.out.println("Média ponderada: " + media);
    }
}
```

## 8. Área do triângulo

Fórmula:

A= base×altura / 2

```
import java.util.Scanner;

public class Exercicio8 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double base, altura, area;

        System.out.print("Digite a base: ");
        base = entrada.nextDouble();

        System.out.print("Digite a altura: ");
        altura = entrada.nextDouble();

        area = (base * altura) / 2;

        System.out.println("Área do triângulo: " + area);
    }
}
```

## 9. Área do quadrado

A=lado2

```
import java.util.Scanner;

public class Exercicio9 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double lado, area;

        System.out.print("Digite o lado do quadrado: ");
        lado = entrada.nextDouble();

        area = Math.pow(lado, 2);

        System.out.println("Área do quadrado: " + area);
    }
}
```

## 10. Conversão de medidas

```
import java.util.Scanner;

public class Exercicio10 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double pes, polegadas, jardas, milhas;

        System.out.print("Digite a medida em pés: ");
        pes = entrada.nextDouble();

        polegadas = pes * 12;
        jardas = pes / 3;
        milhas = jardas / 1760;

        System.out.println("Polegadas: " + polegadas);
        System.out.println("Jardas: " + jardas);
        System.out.println("Milhas: " + milhas);
    }
}

```

## 11. Idade em anos, meses, dias e semanas

```
import java.util.Scanner;

public class Exercicio11 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        int anoNascimento, anoAtual;
        int idadeAnos, idadeMeses, idadeDias, idadeSemanas;

        System.out.print("Digite o ano de nascimento: ");
        anoNascimento = entrada.nextInt();

        System.out.print("Digite o ano atual: ");
        anoAtual = entrada.nextInt();

        idadeAnos = anoAtual - anoNascimento;
        idadeMeses = idadeAnos * 12;
        idadeDias = idadeAnos * 365;
        idadeSemanas = idadeAnos * 52;

        System.out.println("Idade em anos: " + idadeAnos);
        System.out.println("Idade em meses: " + idadeMeses);
        System.out.println("Idade em dias: " + idadeDias);
        System.out.println("Idade em semanas: " + idadeSemanas);
    }
}
```

## 12. Custo final de um carro

```
import java.util.Scanner;

public class Exercicio12 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double precoFabrica, percentualLucro, percentualImpostos;
        double lucro, impostos, precoFinal;

        System.out.print("Digite o preço de fábrica: ");
        precoFabrica = entrada.nextDouble();

        System.out.print("Digite o percentual de lucro do distribuidor: ");
        percentualLucro = entrada.nextDouble();

        System.out.print("Digite o percentual de impostos: ");
        percentualImpostos = entrada.nextDouble();

        lucro = precoFabrica * (percentualLucro / 100);
        impostos = precoFabrica * (percentualImpostos / 100);

        precoFinal = precoFabrica + lucro + impostos;

        System.out.println("Lucro do distribuidor: " + lucro);
        System.out.println("Impostos: " + impostos);
        System.out.println("Preço final: " + precoFinal);
    }
}
```

## 13. Salário baseado em horas trabalhadas

```
import java.util.Scanner;

public class Exercicio13 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double horasTrabalhadas, salarioMinimo;
        double valorHora, salarioBruto, imposto, salarioReceber;

        System.out.print("Digite as horas trabalhadas: ");
        horasTrabalhadas = entrada.nextDouble();

        System.out.print("Digite o salário mínimo: ");
        salarioMinimo = entrada.nextDouble();

        valorHora = salarioMinimo / 10;
        salarioBruto = horasTrabalhadas * valorHora;
        imposto = salarioBruto * 0.03;
        salarioReceber = salarioBruto - imposto;

        System.out.println("Salário a receber: " + salarioReceber);
    }
}
```

## 14. Valor do quilowatt

```
import java.util.Scanner;

public class Exercicio14 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double salarioMinimo, quilowatts;
        double valorQuilowatt, valorTotal, valorDesconto;

        System.out.print("Digite o salário mínimo: ");
        salarioMinimo = entrada.nextDouble();

        System.out.print("Digite a quantidade de quilowatts: ");
        quilowatts = entrada.nextDouble();

        valorQuilowatt = salarioMinimo / 5;
        valorTotal = quilowatts * valorQuilowatt;
        valorDesconto = valorTotal * 0.85;

        System.out.println("Valor de cada quilowatt: " + valorQuilowatt);
        System.out.println("Valor total: " + valorTotal);
        System.out.println("Valor com desconto: " + valorDesconto);
    }
}

```

## 15. Celsius para Fahrenheit

```
import java.util.Scanner;

public class Exercicio15 {
    public static void main(String[] args) {
        Scanner entrada = new Scanner(System.in);

        double celsius, fahrenheit;

        System.out.print("Digite a temperatura em Celsius: ");
        celsius = entrada.nextDouble();

        fahrenheit = (9 * celsius / 5) + 32;

        System.out.println("Temperatura em Fahrenheit: " + fahrenheit);
    }
}

```

