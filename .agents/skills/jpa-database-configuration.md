# Skill: JPA & Database Configuration

## Description
Configura persistência de dados com Spring Data JPA e Hibernate, incluindo mapeamento de entidades, repositórios, migrations com Flyway e boas práticas de modelagem.

## Instructions

### 1. Dependências (pom.xml)
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
```

### 2. Mapeamento de Entidade
```java
@Entity
@Table(name = "produtos")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Produto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 255)
    private String nome;

    @Column(columnDefinition = "TEXT")
    private String descricao;

    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal preco;

    @Column(nullable = false)
    private Integer estoque;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "categoria_id")
    private Categoria categoria;

    @CreationTimestamp
    @Column(updatable = false)
    private LocalDateTime criadoEm;

    @UpdateTimestamp
    private LocalDateTime atualizadoEm;
}
```

### 3. Repository com Queries Customizadas
```java
@Repository
public interface ProdutoRepository extends JpaRepository<Produto, Long> {

    Page<Produto> findByNomeContainingIgnoreCase(String nome, Pageable pageable);

    @Query("SELECT p FROM Produto p WHERE p.estoque > 0 AND p.categoria.id = :categoriaId")
    List<Produto> findDisponiveisPorCategoria(@Param("categoriaId") Long categoriaId);

    @Query(value = "SELECT * FROM produtos WHERE preco BETWEEN :min AND :max", nativeQuery = true)
    List<Produto> findByFaixaDePreco(@Param("min") BigDecimal min, @Param("max") BigDecimal max);

    boolean existsByNomeIgnoreCase(String nome);
}
```

### 4. Migration com Flyway
Crie o arquivo `src/main/resources/db/migration/V1__create_tables.sql`:
```sql
CREATE TABLE categorias (
    id         BIGSERIAL PRIMARY KEY,
    nome       VARCHAR(100) NOT NULL UNIQUE,
    criado_em  TIMESTAMP DEFAULT NOW()
);

CREATE TABLE produtos (
    id             BIGSERIAL PRIMARY KEY,
    nome           VARCHAR(255) NOT NULL,
    descricao      TEXT,
    preco          NUMERIC(10,2) NOT NULL,
    estoque        INTEGER NOT NULL DEFAULT 0,
    categoria_id   BIGINT REFERENCES categorias(id),
    criado_em      TIMESTAMP DEFAULT NOW(),
    atualizado_em  TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_produtos_nome ON produtos(nome);
CREATE INDEX idx_produtos_categoria ON produtos(categoria_id);
```

### 5. Configuração de Auditoria
```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.ofNullable(
            SecurityContextHolder.getContext().getAuthentication()
        ).map(Authentication::getName);
    }
}
```

### 6. Paginação e Ordenação
```java
// No Controller
@GetMapping
public ResponseEntity<Page<ProdutoDTO>> listar(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(defaultValue = "nome") String sortBy) {

    Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy).ascending());
    return ResponseEntity.ok(produtoService.listar(pageable));
}
```

### 7. Propriedades JPA recomendadas
```properties
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.open-in-view=false
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
```
