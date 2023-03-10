# Realizando pruebas con StepVerified

Implementación de pruebas con StepVerified en un proyecto SpringBoot con WebFlux
Se implementan pruebas sobre funciones reactivas, utilizando StepVerified.

Se prueban funciones con componentes Mono y Flux, analizando flujos finitos e infinitos.

#Funciones del servicio a realizar pruebas
@Service
public class Servicio {
public Mono<String> buscarUno(){
return Mono.just("Pedro");
}

    public Flux<String> buscarTodos(){
        return Flux.just("Pedro", "Maria", "Jesus", "Carmen");
    }

    public Flux<String> buscarTodosLento(){
        return Flux.just("Pedro", "Maria", "Jesus", "Carmen").delaySequence(Duration.ofSeconds(20));
    }

    //Implementando publicador
    public Flux<String> buscarTodosFiltro() {
        Flux<String> source = Flux.just("John", "Monica", "Mark", "Cloe", 
                                        "Frank", "Casper", "Olivia", "Emily", "Cate")
                .filter(name -> name.length() == 4)
                .map(String::toUpperCase);

        Flux<String> error = source.concatWith(
                Mono.error(new IllegalArgumentException("Mensaje de Error"))
        );
        return source;
    }

}

# Pruebas realizadas
@SpringBootTest
public class ServicioTest {
@Autowired
Servicio servicio;

    //Trabajando con Mono verifico un solo valor
    @Test
    void testMono(){
        Mono<String> uno = servicio.buscarUno();
        StepVerifier.create(uno).expectNext("Pedro").verifyComplete();
    }

    //Trabajando con Flux trabajo verificando un flujo de valores
    @Test
    void testVarios(){
        Flux<String> uno = servicio.buscarTodos();
        StepVerifier.create(uno).expectNext("Pedro").expectNext("Maria").expectNext("Jesus").expectNext("Carmen").verifyComplete();
    }

    //Trabajando con valores obtenidos en un determinado período de espera
    @Test
    void testVariosLento(){
        Flux<String> uno = servicio.buscarTodosLento();

        StepVerifier.create(uno)
                .expectNext("Pedro")
                .thenAwait(Duration.ofSeconds(1))
                .expectNext("Maria")
                .thenAwait(Duration.ofSeconds(1))
                .expectNext("Jesus")
                .thenAwait(Duration.ofSeconds(1))
                .expectNext("Carmen")
                .thenAwait(Duration.ofSeconds(1)).verifyComplete();
    }

    //Probando el publicador implementado
    @Test
    void testTodosFiltro(){
        Flux<String> source = servicio.buscarTodosFiltro();
        StepVerifier
                .create(source)
                .expectNext("JOHN")
                .expectNextMatches(name -> name.startsWith("MA"))
                .expectNext("CLOE", "CATE")
                .expectComplete()
                .verify();
    }

}