# First project using Spring Batch


### Step: Maven dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```
* Required: Include a implementation JDBC (Database H2)

### Step: Job and Step Configuration

```java
@Configuration
public class BatchConfig {

	@Bean
	public Job job(JobRepository jobRepository, Step step) {
		return new JobBuilder("job", jobRepository)
				.start(step)
				.build();
	}

	@Bean
	public Step step(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
		return new StepBuilder("step", jobRepository)
				.tasklet((StepContribution contribution, ChunkContext chunkContext) -> {
					System.out.println("Olá, mundo!");
					return RepeatStatus.FINISHED;
				}, transactionManager)
				.build();
	}
}
```

### Step: Refactoring project

```java
@Configuration
public class BatchConfig {

	@Bean
	public Job job(JobRepository jobRepository, Step printHelloStep) {
		return new JobBuilder("job", jobRepository)
				.start(printHelloStep)
				.incrementer(new RunIdIncrementer())
				.build();
	}
}
```

```java
@Configuration
public class PrintHelloStepConfig {

	@Bean
	public Step printHelloStep(JobRepository jobRepository, PlatformTransactionManager transactionManager, Tasklet printHelloTasklet) {
		return new StepBuilder("step", jobRepository)
				.tasklet(printHelloTasklet, transactionManager)
				.build();
	}

}
```

```java
@Component
@StepScope
public class PrintHelloTasklet implements Tasklet {

	@Value("#{jobParameters['nome']}")
  private String nome;

	@Override
	public RepeatStatus execute(StepContribution contribution, ChunkContext chunkContext) throws Exception {
		System.out.println("Olá, " + nome +" !");
		return RepeatStatus.FINISHED;
	}
}
```

### Docker-Compose

```
version: "3.7"
services:
  # ===============================================================================
  # MYSQL SERVER
  # ===============================================================================
  mysql-docker:
    image: mysql:8.0
    container_name: dev-mysql
    environment:
      MYSQL_DATABASE: mydatabase
      MYSQL_USER: user
      MYSQL_PASSWORD: 102030
      MYSQL_ROOT_PASSWORD: 102030
    ports:
      - 3307:3306
    volumes:
      - ./.data/mysql/data:/var/lib/mysql
    networks:
      - dev-network
  # ===============================================================================
  # PHPMYADMIN
  # ===============================================================================
  phpmyadmin-docker:
    image: phpmyadmin/phpmyadmin
    container_name: dev-phpmyadmin
    links:
      - mysql-docker
    environment:
      PMA_HOST: mysql-docker
      PMA_PORT: 3306
      PMA_ARBITRARY: 1
    restart: always
    ports:
      - 5050:80
    depends_on:
      - mysql-docker
    networks:
      - dev-network
# ===============================================================================
# REDE
# ===============================================================================
networks:
  dev-network:
    driver: bridge

```





