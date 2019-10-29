# circuit-breaker-hystrix-poc

This project contains:

1. circuit-breaker-bookstore  

    circuit-breaker-bookstore is a simple springboot appliaction which is to recommend books
    
    it is having one REST end point (localhost:8090/recommended) which gives list of books
  
  
    ------ Implementation of circuit-breaker-bookstore ------
    
      --> create spring-boot application from https://start.spring.io/ 
    
          with com.stackroute as group Id, circuit-breaker-bookstore as artifact Id and java version is 11.
    
          Depedencies to be added is (web).
    
      --> Once project is generated unzip and open in IDE. 
      
          Add annotaion @RestController and add one method like below in main method class(CircuitBreakerBookstoreApplication)
      
          @RequestMapping(value = "/recommended")
	        public String readingList(){
	      	  return "Spring in Action (Manning), Cloud Native Java (O'Reilly), Learning Spring Boot (Packt)";
	        }
          
      --> In application.properties file add property
      
          server.port=8090
    
      
2. circuit-breaker-reading

   circuit-breaker-reading is a simple springboot application which to see the recommended books by 
   
   circuit-breaker-bookstore. 
   
   circuit-breaker-reading have one REST end point (localhost:8080/to-read) to get recommeded books from bookstore service.
   
   If bookstore is up then it will fetch from it and display, but if bookstore is down....
   
   Here comes the implementation of hystrix circuit breaker, by implementing this even if bookstore is down we will create 
   
   one REST end point where it will give some default list of books.
   
   This list will be available at (localhost:8080/to-read) same REST end point.
   
    ------ Implementation of circuit-breaker-reading ------
    
     --> create spring-boot application from https://start.spring.io/ 
    
          with com.stackroute as group Id, circuit-breaker-reading as artifact Id and java version is 11.
    
          Depedencies to be added is (web,Hystrix).
          
     --> create one class (BookService) this acts as service
     
         Add code like below 
         
          @Service
          public class BookService {

          private final RestTemplate restTemplate;

          public BookService(RestTemplate rest) {
             this.restTemplate = rest;
           }
          
           //this method have the functionality of circuit breaker like if http://localhost:8090/recommended is not available 
           
           //it picks from reliable method and make these default values available at (localhost:8080/to-read).
           
           
          @HystrixCommand(fallbackMethod = "reliable")
          public String readingList() {
              
              URI uri = URI.create("http://localhost:8090/recommended");

              return this.restTemplate.getForObject(uri, String.class);
          }

          public String reliable() {
              return "Cloud Native Java (O'Reilly)";
          }

        }
          
     --> Once project is generated unzip and open in IDE. 
      
          Add annotaions @RestController, @EnableCircuitBreaker and add code like below in main method class(CircuitBreakerReadingApplication)
      
          @Autowired
	        private BookService bookService;

	        @Bean
        	public RestTemplate rest(RestTemplateBuilder builder) {
	        	return builder.build();
	        }

	        @RequestMapping("/to-read")
	        public String toRead() {
		          return bookService.readingList();
	        }
    
      --> In application.properties file add property
      
          server.port=8080

   
   

