# Curso Prático de Hibernate 6 com Java, Maven e PostgreSQL

Baseado no livro "Introdução ao Hibernate 6", este curso prático oferece uma abordagem passo a passo com exemplos simples e diretos para dominar o Hibernate 6.

## 📋 Pré-requisitos
- **Java 17+** (recomendado)
- **Maven 3.8+**
- **PostgreSQL 15+** (ou Docker)
- **IDE** (IntelliJ IDEA, Eclipse ou VS Code)

## 🎯 Módulos do Curso

### **Módulo 1: Configuração Inicial**
**Objetivo:** Criar projeto Maven com Hibernate 6 e conectar ao PostgreSQL.

1. **Criar projeto Maven:**
```xml
<!-- pom.xml -->
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.exemplo</groupId>
    <artifactId>hibernate-curso</artifactId>
    <version>1.0.0</version>
    
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <hibernate.version>6.4.10.Final</hibernate.version>
    </properties>
    
    <dependencies>
        <!-- Hibernate Core -->
        <dependency>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>${hibernate.version}</version>
        </dependency>
        
        <!-- PostgreSQL Driver -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.7.2</version>
        </dependency>
        
        <!-- Logging -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.22.1</version>
        </dependency>
    </dependencies>
</project>
```

2. **Configurar Hibernate via `persistence.xml`:**
```xml
<!-- src/main/resources/META-INF/persistence.xml -->
<persistence version="3.0">
    <persistence-unit name="livraria">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <class>com.exemplo.model.Livro</class>
        <class>com.exemplo.model.Autor</class>
        
        <properties>
            <property name="jakarta.persistence.jdbc.url" 
                      value="jdbc:postgresql://localhost:5432/livraria"/>
            <property name="jakarta.persistence.jdbc.user" value="postgres"/>
            <property name="jakarta.persistence.jdbc.password" value="senha123"/>
            
            <property name="hibernate.dialect" 
                      value="org.hibernate.dialect.PostgreSQLDialect"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
            
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
        </properties>
    </persistence-unit>
</persistence>
```

3. **Configurar log4j2:**
```properties
# src/main/resources/log4j2.properties
rootLogger.level = INFO
rootLogger.appenderRef.stdout.ref = STDOUT

appender.console.type = Console
appender.console.name = STDOUT
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = %d{HH:mm:ss} %-5p %c{1}: %m%n

logger.hibernate.name = org.hibernate.SQL
logger.hibernate.level = DEBUG
```

---

### **Módulo 2: Entidades Básicas**
**Objetivo:** Criar primeira entidade e fazer CRUD simples.

```java
// src/main/java/com/exemplo/model/Livro.java
package com.exemplo.model;

import jakarta.persistence.*;
import java.time.LocalDate;

@Entity
@Table(name = "livros")
public class Livro {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 200)
    private String titulo;
    
    @Column(unique = true, nullable = false, length = 13)
    private String isbn;
    
    @Column(name = "data_publicacao")
    private LocalDate dataPublicacao;
    
    @Column(precision = 10, scale = 2)
    private Double preco;
    
    // Construtor padrão (obrigatório)
    public Livro() {}
    
    // Construtor com parâmetros
    public Livro(String titulo, String isbn, LocalDate dataPublicacao, Double preco) {
        this.titulo = titulo;
        this.isbn = isbn;
        this.dataPublicacao = dataPublicacao;
        this.preco = preco;
    }
    
    // Getters e Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getTitulo() { return titulo; }
    public void setTitulo(String titulo) { this.titulo = titulo; }
    
    public String getIsbn() { return isbn; }
    public void setIsbn(String isbn) { this.isbn = isbn; }
    
    public LocalDate getDataPublicacao() { return dataPublicacao; }
    public void setDataPublicacao(LocalDate dataPublicacao) { 
        this.dataPublicacao = dataPublicacao; 
    }
    
    public Double getPreco() { return preco; }
    public void setPreco(Double preco) { this.preco = preco; }
    
    @Override
    public String toString() {
        return String.format("Livro[id=%d, titulo='%s', isbn='%s']", 
                            id, titulo, isbn);
    }
}
```

---

### **Módulo 3: Operações CRUD Básicas**
**Objetivo:** Aprender a persistir, buscar, atualizar e remover entidades.

```java
// src/main/java/com/exemplo/service/LivroService.java
package com.exemplo.service;

import com.exemplo.model.Livro;
import jakarta.persistence.*;
import java.time.LocalDate;
import java.util.List;

public class LivroService {
    
    private EntityManagerFactory emf;
    
    public LivroService() {
        emf = Persistence.createEntityManagerFactory("livraria");
    }
    
    // CREATE
    public Livro salvarLivro(Livro livro) {
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        
        try {
            tx.begin();
            em.persist(livro);
            tx.commit();
            return livro;
        } catch (Exception e) {
            if (tx.isActive()) tx.rollback();
            throw e;
        } finally {
            em.close();
        }
    }
    
    // READ (buscar por ID)
    public Livro buscarPorId(Long id) {
        EntityManager em = emf.createEntityManager();
        try {
            return em.find(Livro.class, id);
        } finally {
            em.close();
        }
    }
    
    // READ (listar todos)
    public List<Livro> listarTodos() {
        EntityManager em = emf.createEntityManager();
        try {
            return em.createQuery("SELECT l FROM Livro l", Livro.class)
                    .getResultList();
        } finally {
            em.close();
        }
    }
    
    // UPDATE
    public Livro atualizarLivro(Livro livro) {
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        
        try {
            tx.begin();
            Livro atualizado = em.merge(livro);
            tx.commit();
            return atualizado;
        } catch (Exception e) {
            if (tx.isActive()) tx.rollback();
            throw e;
        } finally {
            em.close();
        }
    }
    
    // DELETE
    public void removerLivro(Long id) {
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        
        try {
            tx.begin();
            Livro livro = em.find(Livro.class, id);
            if (livro != null) {
                em.remove(livro);
            }
            tx.commit();
        } catch (Exception e) {
            if (tx.isActive()) tx.rollback();
            throw e;
        } finally {
            em.close();
        }
    }
    
    public void fechar() {
        if (emf != null && emf.isOpen()) {
            emf.close();
        }
    }
}
```

```java
// src/main/java/com/exemplo/Main.java
package com.exemplo;

import com.exemplo.model.Livro;
import com.exemplo.service.LivroService;
import java.time.LocalDate;

public class Main {
    public static void main(String[] args) {
        LivroService service = new LivroService();
        
        try {
            // Criar novo livro
            Livro livro = new Livro(
                "Hibernate 6 na Prática", 
                "9788550806032", 
                LocalDate.of(2025, 1, 15), 
                89.90
            );
            
            // Salvar
            Livro salvo = service.salvarLivro(livro);
            System.out.println("Livro salvo: " + salvo);
            
            // Buscar por ID
            Livro encontrado = service.buscarPorId(salvo.getId());
            System.out.println("Livro encontrado: " + encontrado);
            
            // Atualizar
            encontrado.setPreco(99.90);
            Livro atualizado = service.atualizarLivro(encontrado);
            System.out.println("Livro atualizado: " + atualizado);
            
            // Listar todos
            System.out.println("\nTodos os livros:");
            service.listarTodos().forEach(System.out::println);
            
            // Remover
            service.removerLivro(atualizado.getId());
            System.out.println("Livro removido com sucesso!");
            
        } finally {
            service.fechar();
        }
    }
}
```

---

### **Módulo 4: Relacionamentos (Associações)**
**Objetivo:** Aprender relacionamentos 1:N, N:1, N:N.

```java
// src/main/java/com/exemplo/model/Autor.java
package com.exemplo.model;

import jakarta.persistence.*;
import java.util.*;

@Entity
@Table(name = "autores")
public class Autor {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String nome;
    
    @Column(length = 100)
    private String nacionalidade;
    
    // Relacionamento 1:N (Um autor tem muitos livros)
    @OneToMany(mappedBy = "autor", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Livro> livros = new ArrayList<>();
    
    public Autor() {}
    
    public Autor(String nome, String nacionalidade) {
        this.nome = nome;
        this.nacionalidade = nacionalidade;
    }
    
    // Getters e Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }
    
    public String getNacionalidade() { return nacionalidade; }
    public void setNacionalidade(String nacionalidade) { 
        this.nacionalidade = nacionalidade; 
    }
    
    public List<Livro> getLivros() { return livros; }
    public void setLivros(List<Livro> livros) { this.livros = livros; }
    
    // Método helper para adicionar livro
    public void adicionarLivro(Livro livro) {
        livros.add(livro);
        livro.setAutor(this);
    }
}
```

```java
// Atualizar entidade Livro para incluir relacionamento
// src/main/java/com/exemplo/model/Livro.java (adicionar)
@Entity
@Table(name = "livros")
public class Livro {
    // ... atributos anteriores ...
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "autor_id")
    private Autor autor;
    
    // ... getters e setters ...
    
    public Autor getAutor() { return autor; }
    public void setAutor(Autor autor) { this.autor = autor; }
}
```

```java
// src/main/java/com/exemplo/service/TesteRelacionamento.java
package com.exemplo.service;

import com.exemplo.model.Autor;
import com.exemplo.model.Livro;
import jakarta.persistence.*;
import java.time.LocalDate;

public class TesteRelacionamento {
    
    public static void main(String[] args) {
        EntityManagerFactory emf = 
            Persistence.createEntityManagerFactory("livraria");
        EntityManager em = emf.createEntityManager();
        
        try {
            em.getTransaction().begin();
            
            // Criar autor
            Autor autor = new Autor("Gavin King", "Australiano");
            em.persist(autor);
            
            // Criar livros associados ao autor
            Livro livro1 = new Livro(
                "Hibernate in Action", 
                "9781932394153", 
                LocalDate.of(2004, 11, 1), 
                59.90
            );
            
            Livro livro2 = new Livro(
                "Java Persistence with Hibernate", 
                "9781617290459", 
                LocalDate.of(2015, 12, 1), 
                79.90
            );
            
            // Associar livros ao autor
            autor.adicionarLivro(livro1);
            autor.adicionarLivro(livro2);
            
            // Persistir (cascata salva os livros também)
            em.persist(autor);
            
            em.getTransaction().commit();
            
            // Buscar autor com livros (JOIN FETCH)
            em.clear();
            Autor autorComLivros = em.createQuery(
                "SELECT a FROM Autor a " +
                "LEFT JOIN FETCH a.livros " +
                "WHERE a.id = :id", Autor.class)
                .setParameter("id", autor.getId())
                .getSingleResult();
            
            System.out.println("Autor: " + autorComLivros.getNome());
            System.out.println("Livros:");
            autorComLivros.getLivros().forEach(l -> 
                System.out.println("  - " + l.getTitulo()));
            
        } finally {
            em.close();
            emf.close();
        }
    }
}
```

---

### **Módulo 5: Consultas HQL e Criteria API**
**Objetivo:** Aprender diferentes formas de consultar dados.

```java
// src/main/java/com/exemplo/service/ConsultasService.java
package com.exemplo.service;

import com.exemplo.model.Livro;
import jakarta.persistence.*;
import jakarta.persistence.criteria.*;
import java.util.List;

public class ConsultasService {
    
    private EntityManagerFactory emf;
    
    public ConsultasService() {
        emf = Persistence.createEntityManagerFactory("livraria");
    }
    
    // 1. Consulta HQL com parâmetros
    public List<Livro> buscarPorTitulo(String palavraChave) {
        EntityManager em = emf.createEntityManager();
        try {
            return em.createQuery(
                "SELECT l FROM Livro l " +
                "WHERE LOWER(l.titulo) LIKE LOWER(:palavra) " +
                "ORDER BY l.titulo", Livro.class)
                .setParameter("palavra", "%" + palavraChave + "%")
                .getResultList();
        } finally {
            em.close();
        }
    }
    
    // 2. Consulta HQL com JOIN
    public List<Object[]> buscarLivrosComAutores() {
        EntityManager em = emf.createEntityManager();
        try {
            return em.createQuery(
                "SELECT l.titulo, a.nome FROM Livro l " +
                "JOIN l.autor a " +
                "WHERE l.preco > :precoMinimo", Object[].class)
                .setParameter("precoMinimo", 50.0)
                .getResultList();
        } finally {
            em.close();
        }
    }
    
    // 3. Consulta com Criteria API (tipada)
    public List<Livro> buscarPorPrecoRange(Double min, Double max) {
        EntityManager em = emf.createEntityManager();
        try {
            CriteriaBuilder cb = em.getCriteriaBuilder();
            CriteriaQuery<Livro> query = cb.createQuery(Livro.class);
            Root<Livro> livro = query.from(Livro.class);
            
            Predicate precoMin = cb.ge(livro.get("preco"), min);
            Predicate precoMax = cb.le(livro.get("preco"), max);
            
            query.select(livro)
                 .where(cb.and(precoMin, precoMax))
                 .orderBy(cb.asc(livro.get("titulo")));
            
            return em.createQuery(query).getResultList();
        } finally {
            em.close();
        }
    }
    
    // 4. Consulta paginada
    public List<Livro> listarPaginado(int pagina, int itensPorPagina) {
        EntityManager em = emf.createEntityManager();
        try {
            return em.createQuery("SELECT l FROM Livro l ORDER BY l.id", Livro.class)
                .setFirstResult((pagina - 1) * itensPorPagina)
                .setMaxResults(itensPorPagina)
                .getResultList();
        } finally {
            em.close();
        }
    }
    
    // 5. Consulta nativa SQL
    public List<Livro> buscarMaisCaros(int limite) {
        EntityManager em = emf.createEntityManager();
        try {
            return em.createNativeQuery(
                "SELECT * FROM livros " +
                "ORDER BY preco DESC " +
                "LIMIT :limite", Livro.class)
                .setParameter("limite", limite)
                .getResultList();
        } finally {
            em.close();
        }
    }
    
    public void fechar() {
        if (emf != null && emf.isOpen()) {
            emf.close();
        }
    }
}
```

---

### **Módulo 6: Herança e Mapeamento Avançado**
**Objetivo:** Aprender estratégias de herança e tipos incorporáveis.

```java
// src/main/java/com/exemplo/model/heranca/Pessoa.java
package com.exemplo.model.heranca;

import jakarta.persistence.*;

// Estratégia SINGLE_TABLE (tabela única)
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "tipo_pessoa", discriminatorType = DiscriminatorType.STRING)
public abstract class Pessoa {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String nome;
    
    @Column(length = 100)
    private String email;
    
    // Getters e Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getNome() { return nome; }
    public void setNome(String nome) { this.nome = nome; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

```java
// src/main/java/com/exemplo/model/heranca/Cliente.java
package com.exemplo.model.heranca;

import jakarta.persistence.*;
import java.time.LocalDate;

@Entity
@DiscriminatorValue("CLIENTE")
public class Cliente extends Pessoa {
    
    @Column(name = "data_cadastro")
    private LocalDate dataCadastro;
    
    @Column(precision = 10, scale = 2)
    private Double limiteCredito;
    
    public Cliente() {}
    
    public Cliente(String nome, String email, Double limiteCredito) {
        setNome(nome);
        setEmail(email);
        this.dataCadastro = LocalDate.now();
        this.limiteCredito = limiteCredito;
    }
    
    // Getters e Setters
    public LocalDate getDataCadastro() { return dataCadastro; }
    public void setDataCadastro(LocalDate dataCadastro) { 
        this.dataCadastro = dataCadastro; 
    }
    
    public Double getLimiteCredito() { return limiteCredito; }
    public void setLimiteCredito(Double limiteCredito) { 
        this.limiteCredito = limiteCredito; 
    }
}
```

```java
// src/main/java/com/exemplo/model/heranca/Funcionario.java
package com.exemplo.model.heranca;

import jakarta.persistence.*;
import java.math.BigDecimal;

@Entity
@DiscriminatorValue("FUNCIONARIO")
public class Funcionario extends Pessoa {
    
    @Column(name = "salario", precision = 10, scale = 2)
    private BigDecimal salario;
    
    @Column(name = "cargo", length = 50)
    private String cargo;
    
    public Funcionario() {}
    
    public Funcionario(String nome, String email, BigDecimal salario, String cargo) {
        setNome(nome);
        setEmail(email);
        this.salario = salario;
        this.cargo = cargo;
    }
    
    // Getters e Setters
    public BigDecimal getSalario() { return salario; }
    public void setSalario(BigDecimal salario) { this.salario = salario; }
    
    public String getCargo() { return cargo; }
    public void setCargo(String cargo) { this.cargo = cargo; }
}
```

---

### **Módulo 7: Performance e Otimização**
**Objetivo:** Aprender técnicas para melhorar performance.

```java
// src/main/java/com/exemplo/service/OtimizacaoService.java
package com.exemplo.service;

import com.exemplo.model.Autor;
import com.exemplo.model.Livro;
import jakarta.persistence.*;
import java.util.List;

public class OtimizacaoService {
    
    // 1. Evitar problema N+1 com JOIN FETCH
    public List<Autor> buscarAutoresComLivros() {
        EntityManagerFactory emf = 
            Persistence.createEntityManagerFactory("livraria");
        EntityManager em = emf.createEntityManager();
        
        try {
            // RUIM: N+1 queries (busca autores, depois cada autor busca livros)
            // List<Autor> autores = em.createQuery("SELECT a FROM Autor a", Autor.class)
            //     .getResultList();
            
            // BOM: 1 query com JOIN FETCH
            return em.createQuery(
                "SELECT DISTINCT a FROM Autor a " +
                "LEFT JOIN FETCH a.livros", Autor.class)
                .getResultList();
        } finally {
            em.close();
            emf.close();
        }
    }
    
    // 2. Usar batch processing para inserções
    public void inserirLivrosEmLote(List<Livro> livros) {
        EntityManagerFactory emf = 
            Persistence.createEntityManagerFactory("livraria");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        
        try {
            tx.begin();
            
            int batchSize = 30;
            int count = 0;
            
            for (Livro livro : livros) {
                em.persist(livro);
                count++;
                
                // Liberar batch periodicamente
                if (count % batchSize == 0) {
                    em.flush();
                    em.clear();
                }
            }
            
            tx.commit();
        } finally {
            em.close();
            emf.close();
        }
    }
    
    // 3. Usar StatelessSession para operações em massa
    public void atualizarPrecosEmMassa(Double percentualAumento) {
        EntityManagerFactory emf = 
            Persistence.createEntityManagerFactory("livraria");
        EntityManager em = emf.createEntityManager();
        
        try {
            // Usando HQL para update em massa (mais eficiente)
            em.getTransaction().begin();
            
            int atualizados = em.createQuery(
                "UPDATE Livro l SET l.preco = l.preco * :fator " +
                "WHERE l.preco < :precoMaximo")
                .setParameter("fator", 1 + (percentualAumento / 100))
                .setParameter("precoMaximo", 100.0)
                .executeUpdate();
            
            em.getTransaction().commit();
            System.out.println("Livros atualizados: " + atualizados);
            
        } finally {
            em.close();
            emf.close();
        }
    }
}
```

---

### **Módulo 8: Transações e Controle de Concorrência**
**Objetivo:** Aprender a gerenciar transações e evitar problemas de concorrência.

```java
// src/main/java/com/exemplo/service/TransacaoService.java
package com.exemplo.service;

import com.exemplo.model.Livro;
import jakarta.persistence.*;

public class TransacaoService {
    
    // 1. Transação com controle de exceções
    public void transferirEstoque(Long origemId, Long destinoId, int quantidade) {
        EntityManagerFactory emf = 
            Persistence.createEntityManagerFactory("livraria");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        
        try {
            tx.begin();
            
            // Lógica de negócio
            Livro origem = em.find(Livro.class, origemId);
            Livro destino = em.find(Livro.class, destinoId);
            
            if (origem == null || destino == null) {
                throw new IllegalArgumentException("Livro não encontrado");
            }
            
            // Atualizar estoque
            // origem.removerEstoque(quantidade);
            // destino.adicionarEstoque(quantidade);
            
            em.merge(origem);
            em.merge(destino);
            
            tx.commit();
            System.out.println("Transferência realizada com sucesso!");
            
        } catch (Exception e) {
            if (tx != null && tx.isActive()) {
                tx.rollback();
                System.out.println("Transação revertida: " + e.getMessage());
            }
            throw e;
        } finally {
            em.close();
            emf.close();
        }
    }
    
    // 2. Bloqueio otimista com @Version
    public void atualizarComControleConcorrencia(Long livroId, String novoTitulo) {
        EntityManagerFactory emf = 
            Persistence.createEntityManagerFactory("livraria");
        EntityManager em = emf.createEntityManager();
        
        try {
            em.getTransaction().begin();
            
            // Buscar com bloqueio otimista
            Livro livro = em.find(Livro.class, livroId, 
                LockModeType.OPTIMISTIC_FORCE_INCREMENT);
            
            if (livro != null) {
                livro.setTitulo(novoTitulo);
                // @Version será incrementado automaticamente
            }
            
            em.getTransaction().commit();
            
        } catch (OptimisticLockException e) {
            System.out.println("Conflito de concorrência! " +
                             "Outro usuário modificou o registro.");
            throw e;
        } finally {
            em.close();
            emf.close();
        }
    }
}
```

---

### **Projeto Final: Sistema de Biblioteca Completo**

```java
// src/main/java/com/exemplo/biblioteca/SistemaBiblioteca.java
package com.exemplo.biblioteca;

import com.exemplo.model.*;
import com.exemplo.service.*;
import jakarta.persistence.*;
import java.time.LocalDate;
import java.util.*;

public class SistemaBiblioteca {
    
    private EntityManagerFactory emf;
    private LivroService livroService;
    private AutorService autorService;
    
    public SistemaBiblioteca() {
        emf = Persistence.createEntityManagerFactory("livraria");
        livroService = new LivroService(emf);
        autorService = new AutorService(emf);
    }
    
    public void executar() {
        Scanner scanner = new Scanner(System.in);
        
        while (true) {
            System.out.println("\n=== SISTEMA BIBLIOTECA ===");
            System.out.println("1. Cadastrar Livro");
            System.out.println("2. Listar Livros");
            System.out.println("3. Buscar Livro por Título");
            System.out.println("4. Cadastrar Autor");
            System.out.println("5. Listar Autores com Livros");
            System.out.println("6. Sair");
            System.out.print("Escolha: ");
            
            int opcao = scanner.nextInt();
            scanner.nextLine(); // Consumir newline
            
            switch (opcao) {
                case 1:
                    cadastrarLivro(scanner);
                    break;
                case 2:
                    listarLivros();
                    break;
                case 3:
                    buscarLivro(scanner);
                    break;
                case 4:
                    cadastrarAutor(scanner);
                    break;
                case 5:
                    listarAutoresComLivros();
                    break;
                case 6:
                    System.out.println("Saindo...");
                    fechar();
                    return;
                default:
                    System.out.println("Opção inválida!");
            }
        }
    }
    
    private void cadastrarLivro(Scanner scanner) {
        System.out.print("Título: ");
        String titulo = scanner.nextLine();
        
        System.out.print("ISBN: ");
        String isbn = scanner.nextLine();
        
        System.out.print("Preço: ");
        Double preco = scanner.nextDouble();
        scanner.nextLine();
        
        Livro livro = new Livro(titulo, isbn, LocalDate.now(), preco);
        livroService.salvarLivro(livro);
        
        System.out.println("Livro cadastrado com sucesso!");
    }
    
    private void listarLivros() {
        System.out.println("\n=== LIVROS CADASTRADOS ===");
        livroService.listarTodos().forEach(livro -> {
            System.out.printf("ID: %d | Título: %s | ISBN: %s | Preço: R$%.2f%n",
                livro.getId(), livro.getTitulo(), livro.getIsbn(), livro.getPreco());
        });
    }
    
    private void buscarLivro(Scanner scanner) {
        System.out.print("Palavra-chave do título: ");
        String palavra = scanner.nextLine();
        
        List<Livro> resultados = livroService.buscarPorTitulo(palavra);
        
        if (resultados.isEmpty()) {
            System.out.println("Nenhum livro encontrado.");
        } else {
            System.out.println("\n=== RESULTADOS ===");
            resultados.forEach(System.out::println);
        }
    }
    
    private void cadastrarAutor(Scanner scanner) {
        System.out.print("Nome do Autor: ");
        String nome = scanner.nextLine();
        
        System.out.print("Nacionalidade: ");
        String nacionalidade = scanner.nextLine();
        
        Autor autor = new Autor(nome, nacionalidade);
        autorService.salvarAutor(autor);
        
        System.out.println("Autor cadastrado com sucesso!");
    }
    
    private void listarAutoresComLivros() {
        System.out.println("\n=== AUTORES E SEUS LIVROS ===");
        autorService.listarTodosComLivros().forEach(autor -> {
            System.out.printf("\nAutor: %s (%s)%n", 
                autor.getNome(), autor.getNacionalidade());
            
            if (autor.getLivros().isEmpty()) {
                System.out.println("  Nenhum livro cadastrado");
            } else {
                System.out.println("  Livros:");
                autor.getLivros().forEach(livro -> 
                    System.out.printf("    - %s%n", livro.getTitulo()));
            }
        });
    }
    
    private void fechar() {
        if (emf != null && emf.isOpen()) {
            emf.close();
        }
    }
    
    public static void main(String[] args) {
        SistemaBiblioteca sistema = new SistemaBiblioteca();
        sistema.executar();
    }
}
```

---

## 📊 Banco de Dados PostgreSQL

```sql
-- Script SQL para criar o banco de dados
CREATE DATABASE livraria;

-- Conectar ao banco
\c livraria

-- Tabela de livros (será criada automaticamente pelo Hibernate)
-- Ou criar manualmente:
CREATE TABLE livros (
    id BIGSERIAL PRIMARY KEY,
    titulo VARCHAR(200) NOT NULL,
    isbn VARCHAR(13) UNIQUE NOT NULL,
    data_publicacao DATE,
    preco DECIMAL(10,2),
    autor_id BIGINT
);

-- Tabela de autores
CREATE TABLE autores (
    id BIGSERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    nacionalidade VARCHAR(100)
);

-- Adicionar chave estrangeira
ALTER TABLE livros 
ADD CONSTRAINT fk_livro_autor 
FOREIGN KEY (autor_id) REFERENCES autores(id);
```

---

## 📚 Exercícios Práticos

1. **Exercício 1:** Modifique o sistema para incluir uma entidade `Editora` com relacionamento N:1 com `Livro`.

2. **Exercício 2:** Implemente uma funcionalidade de busca avançada que permita filtrar livros por:
   - Intervalo de preço
   - Data de publicação
   - Autor específico

3. **Exercício 3:** Crie um relatório que mostre:
   - Total de livros por autor
   - Valor total do estoque
   - Média de preço dos livros

4. **Exercício 4:** Implemente cache de segundo nível usando EHCache.

5. **Exercício 5:** Crie uma API REST simples usando JAX-RS ou Spring Boot com as operações CRUD.

---

## 🔧 Configurações Adicionais do Maven

```xml
<!-- Adicionar ao pom.xml para recursos adicionais -->
<dependencies>
    <!-- Hibernate Validator -->
    <dependency>
        <groupId>org.hibernate.validator</groupId>
        <artifactId>hibernate-validator</artifactId>
        <version>8.0.1.Final</version>
    </dependency>
    
    <!-- Cache de segundo nível -->
    <dependency>
        <groupId>org.hibernate.orm</groupId>
        <artifactId>hibernate-jcache</artifactId>
        <version>6.4.10.Final</version>
    </dependency>
    <dependency>
        <groupId>org.ehcache</groupId>
        <artifactId>ehcache</artifactId>
        <version>3.10.8</version>
    </dependency>
    
    <!-- Metamodel Generator para Criteria API tipada -->
    <dependency>
        <groupId>org.hibernate.orm</groupId>
        <artifactId>hibernate-jpamodelgen</artifactId>
        <version>6.4.10.Final</version>
        <scope>provided</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- Compilador Java -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <source>17</source>
                <target>17</target>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.hibernate.orm</groupId>
                        <artifactId>hibernate-jpamodelgen</artifactId>
                        <version>6.4.10.Final</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

---

## 🎯 Melhores Práticas Aprendidas

1. **Sempre use `fetch = LAZY`** em relacionamentos para evitar problemas de performance
2. **Utilize `JOIN FETCH`** para buscar relacionamentos necessários em uma única query
3. **Gerencie transações cuidadosamente** - sempre faça rollback em caso de erro
4. **Use Criteria API** para consultas dinâmicas complexas
5. **Implemente `equals()` e `hashCode()`** corretamente nas entidades
6. **Evite o problema N+1** com técnicas apropriadas de fetching
7. **Use batch processing** para operações em massa
8. **Configure logging SQL** durante o desenvolvimento
9. **Teste com banco de dados real** (não apenas H2) para garantir compatibilidade
10. **Documente seus modelos** com comentários e anotações claras

---

## 📞 Suporte e Recursos

1. **Documentação Oficial:** https://hibernate.org/orm/documentation/6.4/
2. **Javadoc:** https://docs.jboss.org/hibernate/orm/6.4/javadocs/
3. **Fórum:** https://discourse.hibernate.org/
4. **GitHub:** https://github.com/hibernate/hibernate-orm

Este curso cobre todos os tópicos principais do livro "Introdução ao Hibernate 6" de forma prática e progressiva. Cada módulo pode ser expandido com exercícios adicionais conforme a necessidade do aprendizado.
