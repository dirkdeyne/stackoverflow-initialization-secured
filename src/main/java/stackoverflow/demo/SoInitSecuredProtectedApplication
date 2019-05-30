package stackoverflow.demo;

import java.security.Principal;
import java.util.stream.Stream;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.event.EventListener;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
public class SoInitSecuredProtectedApplication {
   
	public static void main(String[] args) {
		SpringApplication.run(SoInitSecuredProtectedApplication.class, args);
	}
	
}

@Component
class AllowInitialization {
  public boolean empty() {
    return true;
 }
}


@Component
class InitializeDataIfNoDataPresent {
  private final StudentRepo repository;

  public InitializeDataIfNoDataPresent(StudentRepo repo) {
      this.repository = repo;
  }
  
  @EventListener(ApplicationReadyEvent.class)
  public void setup() {
    init();
  }
  
  @PreAuthorize("hasRole('ADMIN')")
  public void init() {
    if(0 == repository.count()) {
     Stream.of("Dirk", "Mark", "Peter", "Bart", "Jhon", "Luc").map(name -> new Student(name)).forEach(s -> repository.save(s));
     System.err.println("DATA INITIALIZED");
    } else {
      System.err.println("DATA PRESENT");
    }
  }
}

@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled=true)
class SecurityConfiguration extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
      http.csrf().disable()
        .authorizeRequests().anyRequest().authenticated()
        .and().httpBasic();
 }
}

@Entity
class Student {
  @Id
  @GeneratedValue
  private Long id;
  private String name;

  public Student() {}

  public Student(String name) {this.name = name;}

  public Long getId() {return id;}

  public String getName() {return name;}

}

interface StudentRepo extends PagingAndSortingRepository<Student, Long> {}

@RestController
class StudentController {

  StudentRepo repo;
  InitializeDataIfNoDataPresent dataPresent;
  
  public StudentController(StudentRepo repo, InitializeDataIfNoDataPresent dataPresent) {
    this.repo = repo;
    this.dataPresent = dataPresent;
  }

  @GetMapping("/students")
  public Page<Student> students(Pageable pageable) {
    return repo.findAll(pageable);
  }
  
  @GetMapping("/init")
  public  Page<Student> init(Principal principal) {
    System.err.println(principal);
    dataPresent.init();
    return students(Pageable.unpaged());
  }

}
