import java.util.*;

public class Main {
    public static void main(String[] args) {
        // Configura o Scanner para ler a entrada do teclado
        Scanner scanner = new Scanner(System.in);
        scanner.useLocale(Locale.US); // Permite digitar notas com ponto (ex: 7.5)
        
        // Declaração das variáveis do tipo real (double em Java)
        double n1, n2, n3, n4, res;
        
        System.out.println("Digite as 4 notas do aluno (pressione Enter após cada uma):");
        
        // Leitura das notas (Equivalente ao "leia N1, N2, N3, N4")
        n1 = scanner.nextDouble();
        n2 = scanner.nextDouble();
        n3 = scanner.nextDouble();
        n4 = scanner.nextDouble();
        
        // Cálculo da média matemática
        res = (n1 + n2 + n3 + n4) / 4.0;
        
        // Exibe a média calculada formatada com duas casas decimais
        System.out.printf("Média final: %.2f%n", res);
        
        // Estrutura Condicional (Equivalente ao se / senao)
        if (res >= 7.0) {
            System.out.println("Aprovado");
        } else {
            System.out.println("Reprovado");
        }
        
        scanner.close(); // Fecha o scanner por boa prática
    }
}
