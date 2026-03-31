# Skill: REST API Development with Spring

## Description
Implementa endpoints RESTful seguindo boas práticas com Spring MVC, incluindo tratamento de erros, validação de dados e documentação com OpenAPI/Swagger.

## Instructions

### 1. Controller REST
```java
@RestController
@RequestMapping("/api/v1/produtos")
@RequiredArgsConstructor
public class ProdutoController {

    private final ProdutoService produtoService;

    @GetMapping
    public ResponseEntity<Page<ProdutoDTO>> listar(Pageable pageable) {
        return ResponseEntity.ok(produtoService.listar(pageable));
    }

    @GetMapping("/{id}")
    public ResponseEntity<ProdutoDTO> buscarPorId(@PathVariable Long id) {
        return ResponseEntity.ok(produtoService.buscarPorId(id));
    }

    @PostMapping
    public ResponseEntity<ProdutoDTO> criar(@RequestBody @Valid ProdutoDTO dto) {
        ProdutoDTO criado = produtoService.criar(dto);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{id}").buildAndExpand(criado.id()).toUri();
        return ResponseEntity.created(location).body(criado);
    }

    @PutMapping("/{id}")
    public ResponseEntity<ProdutoDTO> atualizar(@PathVariable Long id,
                                                @RequestBody @Valid ProdutoDTO dto) {
        return ResponseEntity.ok(produtoService.atualizar(id, dto));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletar(@PathVariable Long id) {
        produtoService.deletar(id);
        return ResponseEntity.noContent().build();
    }
}
```

### 2. Service Layer
```java
@Service
@RequiredArgsConstructor
@Transactional
public class ProdutoService {

    private final ProdutoRepository repository;
    private final ProdutoMapper mapper;

    @Transactional(readOnly = true)
    public Page<ProdutoDTO> listar(Pageable pageable) {
        return repository.findAll(pageable).map(mapper::toDTO);
    }

    @Transactional(readOnly = true)
    public ProdutoDTO buscarPorId(Long id) {
        Produto produto = repository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Produto não encontrado: " + id));
        return mapper.toDTO(produto);
    }

    public ProdutoDTO criar(ProdutoDTO dto) {
        Produto produto = mapper.toEntity(dto);
        return mapper.toDTO(repository.save(produto));
    }

    public ProdutoDTO atualizar(Long id, ProdutoDTO dto) {
        Produto produto = repository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Produto não encontrado: " + id));
        mapper.updateFromDTO(dto, produto);
        return mapper.toDTO(repository.save(produto));
    }

    public void deletar(Long id) {
        if (!repository.existsById(id)) {
            throw new ResourceNotFoundException("Produto não encontrado: " + id);
        }
        repository.deleteById(id);
    }
}
```

### 3. Tratamento Global de Erros
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(), ex.getMessage(), LocalDateTime.now());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult().getFieldErrors().stream()
            .map(fe -> fe.getField() + ": " + fe.getDefaultMessage())
            .collect(Collectors.toList());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(), "Erros de validação: " + errors, LocalDateTime.now());
        return ResponseEntity.badRequest().body(error);
    }
}
```

### 4. DTO com Validação
```java
public record ProdutoDTO(
    Long id,
    @NotBlank(message = "Nome é obrigatório") String nome,
    @NotBlank(message = "Descrição é obrigatória") String descricao,
    @NotNull @Positive(message = "Preço deve ser positivo") BigDecimal preco,
    @NotNull @Min(0) Integer estoque
) {}
```

### 5. Configuração OpenAPI (Swagger)
Adicione a dependência no `pom.xml`:
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

Acesse a documentação em: `http://localhost:8080/swagger-ui.html`
