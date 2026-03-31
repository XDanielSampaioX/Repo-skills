# Skill: Testing with JUnit and Spring Boot

## Description
Implementa testes unitários e de integração em projetos Spring Boot utilizando JUnit 5, Mockito, MockMvc e Testcontainers para garantir qualidade e confiabilidade do código.

## Instructions

### 1. Dependências de Teste (pom.xml)
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>
```

### 2. Teste Unitário de Service com Mockito
```java
@ExtendWith(MockitoExtension.class)
class ProdutoServiceTest {

    @Mock
    private ProdutoRepository repository;

    @Mock
    private ProdutoMapper mapper;

    @InjectMocks
    private ProdutoService service;

    @Test
    @DisplayName("Deve retornar produto quando encontrado por ID")
    void buscarPorId_quandoExiste_deveRetornarDTO() {
        Produto produto = Produto.builder().id(1L).nome("Notebook").build();
        ProdutoDTO dto = new ProdutoDTO(1L, "Notebook", "Desc", BigDecimal.TEN, 5);

        when(repository.findById(1L)).thenReturn(Optional.of(produto));
        when(mapper.toDTO(produto)).thenReturn(dto);

        ProdutoDTO resultado = service.buscarPorId(1L);

        assertThat(resultado).isNotNull();
        assertThat(resultado.id()).isEqualTo(1L);
        verify(repository).findById(1L);
    }

    @Test
    @DisplayName("Deve lançar exceção quando produto não encontrado")
    void buscarPorId_quandoNaoExiste_deveLancarExcecao() {
        when(repository.findById(99L)).thenReturn(Optional.empty());

        assertThatThrownBy(() -> service.buscarPorId(99L))
            .isInstanceOf(ResourceNotFoundException.class)
            .hasMessageContaining("99");
    }
}
```

### 3. Teste de Controller com MockMvc
```java
@WebMvcTest(ProdutoController.class)
@AutoConfigureMockMvc(addFilters = false)
class ProdutoControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ProdutoService produtoService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("GET /api/v1/produtos/{id} deve retornar 200 com produto")
    void buscarPorId_deveRetornar200() throws Exception {
        ProdutoDTO dto = new ProdutoDTO(1L, "Notebook", "Desc", BigDecimal.TEN, 5);
        when(produtoService.buscarPorId(1L)).thenReturn(dto);

        mockMvc.perform(get("/api/v1/produtos/1")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1L))
            .andExpect(jsonPath("$.nome").value("Notebook"));
    }

    @Test
    @DisplayName("POST /api/v1/produtos deve retornar 201 ao criar produto válido")
    void criar_comDadosValidos_deveRetornar201() throws Exception {
        ProdutoDTO dto = new ProdutoDTO(null, "Notebook", "Desc", BigDecimal.TEN, 5);
        ProdutoDTO criado = new ProdutoDTO(1L, "Notebook", "Desc", BigDecimal.TEN, 5);
        when(produtoService.criar(any())).thenReturn(criado);

        mockMvc.perform(post("/api/v1/produtos")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(dto)))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"));
    }
}
```

### 4. Teste de Integração com Testcontainers
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
@ActiveProfiles("test")
class ProdutoIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureDataSource(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void criarEBuscarProduto_deveRetornarProdutoCriado() {
        ProdutoDTO dto = new ProdutoDTO(null, "Teclado", "Teclado mecânico", new BigDecimal("199.99"), 10);

        ResponseEntity<ProdutoDTO> criacao = restTemplate.postForEntity("/api/v1/produtos", dto, ProdutoDTO.class);
        assertThat(criacao.getStatusCode()).isEqualTo(HttpStatus.CREATED);

        Long id = criacao.getBody().id();
        ResponseEntity<ProdutoDTO> busca = restTemplate.getForEntity("/api/v1/produtos/" + id, ProdutoDTO.class);
        assertThat(busca.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(busca.getBody().nome()).isEqualTo("Teclado");
    }
}
```

### 5. Boas Práticas de Teste
- Use `@DisplayName` para descrever o comportamento esperado
- Siga o padrão AAA: **Arrange**, **Act**, **Assert**
- Teste cenários de sucesso e de falha
- Mantenha os testes independentes entre si
- Use `@BeforeEach` para setup comum
- Prefira asserções com AssertJ (fluent API)
