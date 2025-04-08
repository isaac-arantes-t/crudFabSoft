# crudFabSoft

Author.java
@Entity
public class Author {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @Column(length = 1000)
    private String description;

    @Enumerated(EnumType.STRING)
    @Column(updatable = false) // Especialidade não pode ser alterada
    private Category specialty;

    @OneToMany(mappedBy = "author", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Book> books = new ArrayList<>();
}

Book.java
@Entity
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @Enumerated(EnumType.STRING)
    private Category category;

    @ManyToOne
    @JoinColumn(name = "author_id")
    private Author author;
}

Category.java
public enum Category {
    TERROR, SUSPENSE, DRAMA, ACAO;
}

SWAGGER:
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
  <version>2.0.2</version>
</dependency>

http://localhost:8080/swagger-ui.html

AuthorRepository.Java
@Repository
public interface AuthorRepository extends JpaRepository<Author, Long> {
}

BookRepository.java
@Repository
public interface BookRepository extends JpaRepository<Book, Long> {
}

AuthorService.java
@Service
public class AuthorService {

    @Autowired
    private AuthorRepository authorRepository;

    @Autowired
    private BookRepository bookRepository;

    public Author createAuthor(Author author) {
        return authorRepository.save(author);
    }

    public List<Author> getAllAuthors() {
        return authorRepository.findAll();
    }

    public Author getAuthorById(Long id) {
        return authorRepository.findById(id).orElseThrow(() -> new RuntimeException("Autor não encontrado"));
    }

    public Author updateAuthor(Long id, Author updatedAuthor) {
        Author author = getAuthorById(id);
        author.setName(updatedAuthor.getName());
        author.setDescription(updatedAuthor.getDescription());
        return authorRepository.save(author);
    }

    public void deleteAuthor(Long id) {
        authorRepository.deleteById(id);
    }

    public Book addBookToAuthor(Long authorId, Book book) {
        Author author = getAuthorById(authorId);
        book.setAuthor(author);
        return bookRepository.save(book);
    }
}

BookService.java
@Service
public class BookService {

    @Autowired
    private BookRepository bookRepository;

    public Book createBook(Book book) {
        return bookRepository.save(book);
    }

    public List<Book> getAllBooks() {
        return bookRepository.findAll();
    }

    public Book getBookById(Long id) {
        return bookRepository.findById(id).orElseThrow(() -> new RuntimeException("Livro não encontrado"));
    }

    public Book updateBook(Long id, Book updatedBook) {
        Book book = getBookById(id);
        book.setName(updatedBook.getName());
        book.setCategory(updatedBook.getCategory());
        return bookRepository.save(book);
    }

    public void deleteBook(Long id) {
        bookRepository.deleteById(id);
    }
}

AuthorController.java
@RestController
@RequestMapping("/authors")
@Tag(name = "Autores")
public class AuthorController {

    @Autowired
    private AuthorService authorService;

    @PostMapping
    public ResponseEntity<Author> createAuthor(@RequestBody Author author) {
        return ResponseEntity.ok(authorService.createAuthor(author));
    }

    @GetMapping
    public ResponseEntity<List<Author>> getAllAuthors() {
        return ResponseEntity.ok(authorService.getAllAuthors());
    }

    @GetMapping("/{id}")
    public ResponseEntity<Author> getAuthorById(@PathVariable Long id) {
        return ResponseEntity.ok(authorService.getAuthorById(id));
    }

    @PutMapping("/{id}")
    public ResponseEntity<Author> updateAuthor(@PathVariable Long id, @RequestBody Author author) {
        return ResponseEntity.ok(authorService.updateAuthor(id, author));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteAuthor(@PathVariable Long id) {
        authorService.deleteAuthor(id);
        return ResponseEntity.noContent().build();
    }

    @PostMapping("/{id}/books")
    public ResponseEntity<Book> addBookToAuthor(@PathVariable Long id, @RequestBody Book book) {
        return ResponseEntity.ok(authorService.addBookToAuthor(id, book));
    }
}

BookController.java
@RestController
@RequestMapping("/books")
@Tag(name = "Livros")
public class BookController {

    @Autowired
    private BookService bookService;

    @PostMapping
    public ResponseEntity<Book> createBook(@RequestBody Book book) {
        return ResponseEntity.ok(bookService.createBook(book));
    }

    @GetMapping
    public ResponseEntity<List<Book>> getAllBooks() {
        return ResponseEntity.ok(bookService.getAllBooks());
    }

    @GetMapping("/{id}")
    public ResponseEntity<Book> getBookById(@PathVariable Long id) {
        return ResponseEntity.ok(bookService.getBookById(id));
    }

    @PutMapping("/{id}")
    public ResponseEntity<Book> updateBook(@PathVariable Long id, @RequestBody Book book) {
        return ResponseEntity.ok(bookService.updateBook(id, book));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteBook(@PathVariable Long id) {
        bookService.deleteBook(id);
        return ResponseEntity.noContent().build();
    }
}


application.properties
# Porta padrão
server.port=8080

# Banco de dados H2 (memória)
spring.datasource.url=jdbc:h2:mem:livrosdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# JPA / Hibernate
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Console H2
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

livros-api/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── seuusuario/
│   │   │           └── livrosapi/
│   │   │               ├── LivrosApiApplication.java
│   │   │
│   │   │               ├── controller/
│   │   │               │   ├── AuthorController.java
│   │   │               │   └── BookController.java
│   │   │
│   │   │               ├── model/
│   │   │               │   ├── Author.java
│   │   │               │   ├── Book.java
│   │   │               │   └── Category.java
│   │   │
│   │   │               ├── repository/
│   │   │               │   ├── AuthorRepository.java
│   │   │               │   └── BookRepository.java
│   │   │
│   │   │               └── service/
│   │   │                   ├── AuthorService.java
│   │   │                   └── BookService.java
│   │   │
│   │   └── resources/
│   │       ├── application.properties
│   │       └── static/ (opcional)
│   │       └── templates/ (opcional)
│
├── pom.xml
└── README.md (opcional)
