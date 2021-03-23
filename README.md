## Core Java IO<br> Como evitar o Java FileNotFoundException ao carregar recursos

# 1. Visão Geral
Neste tutorial, exploraremos um problema que pode surgir ao ler arquivos de recursos em um aplicativo Java: em tempo de execução, a pasta de recursos raramente está no mesmo local no disco que está em nosso código-fonte.

Vamos ver como o Java nos permite acessar arquivos de recursos depois que nosso código foi empacotado.

# 2. Lendo arquivos
Digamos que nosso aplicativo leia um arquivo durante a inicialização:

```
try (FileReader fileReader = new FileReader("src/main/resources/input.txt"); 
     BufferedReader reader = new BufferedReader(fileReader)) {
    String contents = reader.lines()
      .collect(Collectors.joining(System.lineSeparator()));
}
```

Se executarmos o código acima em um IDE, o arquivo carrega sem erro. Isso ocorre porque nosso IDE usa nosso diretório de projeto como seu diretório de trabalho atual e o diretório src/main/resources está ali para o aplicativo ler.

Agora, digamos que usamos o plugin Maven JAR para empacotar nosso código como um JAR.

Quando o executamos na linha de comando:

```
java -jar core-java-io2.jar
```

Veremos o seguinte erro:

```
Exception in thread "main" java.io.FileNotFoundException: 
    src/main/resources/input.txt (No such file or directory)
	at java.io.FileInputStream.open0(Native Method)
	at java.io.FileInputStream.open(FileInputStream.java:195)
	at java.io.FileInputStream.<init>(FileInputStream.java:138)
	at java.io.FileInputStream.<init>(FileInputStream.java:93)
	at java.io.FileReader.<init>(FileReader.java:58)
	at com.isaccanedo.resource.MyResourceLoader.loadResourceWithReader(MyResourceLoader.java:14)
	at com.isaccanedo.resource.MyResourceLoader.main(MyResourceLoader.java:37)
```

# 3. Código-fonte vs código compilado
Quando construímos um JAR, os recursos são colocados no diretório raiz dos artefatos empacotados.

Em nosso exemplo, vemos que a configuração do código-fonte tem input.txt em src/main/resources em nosso diretório de código-fonte.

Na estrutura JAR correspondente, no entanto, vemos:

```
META-INF/MANIFEST.MF
META-INF/
com/
com/isaccanedo/
com/isaccanedo/resource/
META-INF/maven/
META-INF/maven/com.isaccanedo/
META-INF/maven/com.isaccanedo/core-java-io-files/
input.txt
com/isaccanedo/resource/MyResourceLoader.class
META-INF/maven/com.isaccanedo/core-java-io-files/pom.xml
META-INF/maven/com.isaccanedo/core-java-io-files/pom.properties
```

Aqui, input.txt está no diretório raiz do JAR. Portanto, quando o código for executado, veremos a FileNotFoundException.

Mesmo se alterássemos o caminho para /input.txt, o código original não poderia carregar esse arquivo, pois os recursos geralmente não são endereçáveis como arquivos no disco. Os arquivos de recursos são empacotados dentro do JAR e, portanto, precisamos de uma maneira diferente de acessá-los.

# 4. Recursos
Em vez disso, vamos usar o carregamento de recursos para carregar recursos do caminho de classe em vez de um local de arquivo específico. Isso funcionará independentemente de como o código é empacotado:

```
try (InputStream inputStream = getClass().getResourceAsStream("/input.txt");
    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream))) {
    String contents = reader.lines()
      .collect(Collectors.joining(System.lineSeparator()));
}
```

ClassLoader.getResourceAsStream() olha o classpath para o recurso fornecido. A barra inicial na entrada para getResourceAsStream() diz ao carregador para ler a partir da base do classpath. O conteúdo de nosso arquivo JAR está no caminho de classe, portanto, esse método funciona.

Um IDE normalmente inclui src/main/resources em seu classpath e, assim, encontra os arquivos.

# 5. Conclusão
Neste artigo rápido, implementamos o carregamento de arquivos como recursos de caminho de classe, para permitir que nosso código funcione de forma consistente, independentemente de como foi empacotado.

# 

Hope that helps. Cheers!<br>
Isac Canedo 😉
