# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 9 syscalls  
- ex1b_write: 9 syscalls  

**2. Por que há diferença entre os dois métodos?**  
O `printf` usa um buffer interno da biblioteca C, então dependendo do contexto ele acumula dados e só chama `write` quando precisa.  
Já o `write` chama o kernel diretamente sem passar por esse buffer intermediário. Ou seja, o `printf` depende do mecanismo de buffering, enquanto o `write` é mais “direto”.  

**3. Qual método é mais previsível? Por quê?**  
O `write` é mais previsível porque cada chamada realmente gera uma syscall para o kernel. O `printf` pode variar, pois só envia ao kernel quando o buffer enche ou encontra um caractere especial (`
`).  

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: 3  
- Bytes lidos: 42  

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**  
O programa usou o FD 3, porque os descritores 0, 1 e 2 já estão reservados (stdin, stdout e stderr). O próximo disponível é o 3.  

**2. Como você sabe que o arquivo foi lido completamente?**  
Porque a última chamada de `read` retornou 0 (EOF).  

**3. Por que verificar retorno de cada syscall?**  
Porque uma syscall pode falhar por vários motivos (arquivo inexistente, permissão negada, disco cheio, etc.). Verificar retorno evita erros silenciosos.  

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: 25 (esperado: 25)  
- Caracteres: 1243  
- Chamadas read(): 20  
- Tempo: 0.001234 segundos  

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          | 72              | 0.004800  |
| 64          | 20              | 0.001234  |
| 256         | 6               | 0.000901  |
| 1024        | 2               | 0.000712  |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**  
Buffers menores → mais chamadas `read()`.  
Buffers maiores → menos chamadas `read()`.  

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes?**  
Não. As últimas leituras retornam menos bytes porque o arquivo termina antes de preencher o buffer.  

**3. Qual é a relação entre syscalls e performance?**  
Mais syscalls = mais custo de transição ao kernel.  
Menos syscalls (buffers maiores) = geralmente mais rápido.  

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1243  
- Operações: 20  
- Tempo: 0.002101 segundos  
- Throughput: 583.02 KB/s  

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [X] Idênticos [ ] Diferentes  

### 🔍 Análise

**1. Por que verificar que bytes_escritos == bytes_lidos?**  
Para garantir que o `write` escreveu exatamente o que o `read` retornou.  

**2. Que flags são essenciais no open() do destino?**  
`O_WRONLY`, `O_CREAT` e `O_TRUNC`.  

**3. O número de reads e writes é igual? Por quê?**  
Sim, pois cada leitura gera uma escrita correspondente.  

**4. Como saber se o disco ficou cheio?**  
O `write` retornaria menos bytes do que o esperado ou erro `ENOSPC`.  

**5. O que acontece se esquecer de fechar os arquivos?**  
Os descritores ficam abertos consumindo recursos, causando vazamento de FD em sistemas maiores.  

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**  
Cada syscall muda do espaço de usuário para o espaço do kernel, onde o kernel tem acesso direto ao hardware e sistema de arquivos.  

**2. Qual é a importância dos file descriptors?**  
São "identificadores" inteiros que o kernel usa para referenciar arquivos abertos. O programa não acessa o arquivo diretamente, apenas via FD.  

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**  
Buffers maiores reduzem chamadas, melhorando performance. Mas buffers muito grandes podem desperdiçar memória sem ganho real.  

### ⚡ Comparação de Performance

```bash
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** cp  

**Por que você acha que foi mais rápido?**  
Porque `cp` é otimizado, pode usar funções como `sendfile()` que copiam diretamente dentro do kernel sem precisar passar os dados para o espaço do usuário.  

---

## 📤 Entrega
- [X] Todos os códigos com TODOs completados  
- [X] Traces salvos em `traces/`  
- [X] Este relatório preenchido como `RELATORIO.md`  

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```

---
# Bom trabalho!
