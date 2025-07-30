========================================================================================================================================
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>
    <groupId>com.example</groupId>
    <artifactId>employee-crud-api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>employee-crud-api</name>
    <description>Employee CRUD API with Spring Boot 3</description>
    <properties>
        <java.version>17</java.version>
    </properties>
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Database -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- Testing Dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        
        <!-- Mockito (included in spring-boot-starter-test but explicitly mentioned) -->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-junit-jupiter</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
====================================================================================================================================
package com.example.employeecrud.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.math.BigDecimal;
import java.time.LocalDate;

@Entity
@Table(name = "employees")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Employee {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank(message = "First name is required")
    @Column(name = "first_name", nullable = false, length = 50)
    private String firstName;
    
    @NotBlank(message = "Last name is required")
    @Column(name = "last_name", nullable = false, length = 50)
    private String lastName;
    
    @Email(message = "Email should be valid")
    @NotBlank(message = "Email is required")
    @Column(name = "email", nullable = false, unique = true, length = 100)
    private String email;
    
    @NotBlank(message = "Department is required")
    @Column(name = "department", nullable = false, length = 50)
    private String department;
    
    @NotNull(message = "Salary is required")
    @Column(name = "salary", nullable = false, precision = 10, scale = 2)
    private BigDecimal salary;
    
    @Column(name = "hire_date")
    private LocalDate hireDate;
    
    @Column(name = "phone", length = 15)
    private String phone;
    
    @Column(name = "address", length = 200)
    private String address;
}
=======================================================================
package com.example.employeecrud.repository;

import com.example.employeecrud.entity.Employee;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    
    Optional<Employee> findByEmail(String email);
    
    List<Employee> findByDepartment(String department);
    
    @Query("SELECT e FROM Employee e WHERE e.firstName LIKE %:name% OR e.lastName LIKE %:name%")
    List<Employee> findByNameContaining(@Param("name") String name);
    
    boolean existsByEmail(String email);
}
=======================================================================
package com.example.employeecrud.service;

import com.example.employeecrud.entity.Employee;
import com.example.employeecrud.exception.ResourceNotFoundException;
import com.example.employeecrud.repository.EmployeeRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDate;
import java.util.List;
import java.util.Optional;

@Service
@RequiredArgsConstructor
@Slf4j
@Transactional
public class EmployeeService {
    
    private final EmployeeRepository employeeRepository;
    
    public List<Employee> getAllEmployees() {
        log.info("Fetching all employees");
        return employeeRepository.findAll();
    }
    
    public Employee getEmployeeById(Long id) {
        log.info("Fetching employee with id: {}", id);
        return employeeRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Employee not found with id: " + id));
    }
    
    public Employee createEmployee(Employee employee) {
        log.info("Creating new employee with email: {}", employee.getEmail());
        
        if (employeeRepository.existsByEmail(employee.getEmail())) {
            throw new IllegalArgumentException("Employee with email " + employee.getEmail() + " already exists");
        }
        
        if (employee.getHireDate() == null) {
            employee.setHireDate(LocalDate.now());
        }
        
        return employeeRepository.save(employee);
    }
    
    public Employee updateEmployee(Long id, Employee employeeDetails) {
        log.info("Updating employee with id: {}", id);
        
        Employee employee = getEmployeeById(id);
        
        // Check if email is being changed and if new email already exists
        if (!employee.getEmail().equals(employeeDetails.getEmail()) && 
            employeeRepository.existsByEmail(employeeDetails.getEmail())) {
            throw new IllegalArgumentException("Employee with email " + employeeDetails.getEmail() + " already exists");
        }
        
        employee.setFirstName(employeeDetails.getFirstName());
        employee.setLastName(employeeDetails.getLastName());
        employee.setEmail(employeeDetails.getEmail());
        employee.setDepartment(employeeDetails.getDepartment());
        employee.setSalary(employeeDetails.getSalary());
        employee.setPhone(employeeDetails.getPhone());
        employee.setAddress(employeeDetails.getAddress());
        
        return employeeRepository.save(employee);
    }
    
    public void deleteEmployee(Long id) {
        log.info("Deleting employee with id: {}", id);
        Employee employee = getEmployeeById(id);
        employeeRepository.delete(employee);
    }
    
    @Transactional(readOnly = true)
    public List<Employee> getEmployeesByDepartment(String department) {
        log.info("Fetching employees by department: {}", department);
        return employeeRepository.findByDepartment(department);
    }
    
    @Transactional(readOnly = true)
    public List<Employee> searchEmployeesByName(String name) {
        log.info("Searching employees by name: {}", name);
        return employeeRepository.findByNameContaining(name);
    }
    
    @Transactional(readOnly = true)
    public Optional<Employee> getEmployeeByEmail(String email) {
        log.info("Fetching employee by email: {}", email);
        return employeeRepository.findByEmail(email);
    }
}
======================================================================================
package com.example.employeecrud.exception;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
======================================================================
package com.example.employeecrud.controller;

import com.example.employeecrud.entity.Employee;
import com.example.employeecrud.service.EmployeeService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/employees")
@RequiredArgsConstructor
@CrossOrigin(origins = "*")
public class EmployeeController {
    
    private final EmployeeService employeeService;
    
    @GetMapping
    public ResponseEntity<List<Employee>> getAllEmployees() {
        List<Employee> employees = employeeService.getAllEmployees();
        return ResponseEntity.ok(employees);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Employee> getEmployeeById(@PathVariable Long id) {
        Employee employee = employeeService.getEmployeeById(id);
        return ResponseEntity.ok(employee);
    }
    
    @PostMapping
    public ResponseEntity<Employee> createEmployee(@Valid @RequestBody Employee employee) {
        Employee createdEmployee = employeeService.createEmployee(employee);
        return ResponseEntity.status(HttpStatus.CREATED).body(createdEmployee);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Employee> updateEmployee(@PathVariable Long id, 
                                                 @Valid @RequestBody Employee employeeDetails) {
        Employee updatedEmployee = employeeService.updateEmployee(id, employeeDetails);
        return ResponseEntity.ok(updatedEmployee);
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteEmployee(@PathVariable Long id) {
        employeeService.deleteEmployee(id);
        return ResponseEntity.noContent().build();
    }
    
    @GetMapping("/department/{department}")
    public ResponseEntity<List<Employee>> getEmployeesByDepartment(@PathVariable String department) {
        List<Employee> employees = employeeService.getEmployeesByDepartment(department);
        return ResponseEntity.ok(employees);
    }
    
    @GetMapping("/search")
    public ResponseEntity<List<Employee>> searchEmployeesByName(@RequestParam String name) {
        List<Employee> employees = employeeService.searchEmployeesByName(name);
        return ResponseEntity.ok(employees);
    }
    
    @GetMapping("/email/{email}")
    public ResponseEntity<Employee> getEmployeeByEmail(@PathVariable String email) {
        return employeeService.getEmployeeByEmail(email)
                .map(employee -> ResponseEntity.ok(employee))
                .orElse(ResponseEntity.notFound().build());
    }
}
==================================================================================================
package com.example.employeecrud.exception;

import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;

@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFoundException(ResourceNotFoundException ex) {
        log.error("Resource not found: {}", ex.getMessage());
        ErrorResponse errorResponse = ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.NOT_FOUND.value())
                .error("Not Found")
                .message(ex.getMessage())
                .build();
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(errorResponse);
    }
    
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgumentException(IllegalArgumentException ex) {
        log.error("Illegal argument: {}", ex.getMessage());
        ErrorResponse errorResponse = ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.BAD_REQUEST.value())
                .error("Bad Request")
                .message(ex.getMessage())
                .build();
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(errorResponse);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationExceptions(MethodArgumentNotValidException ex) {
        log.error("Validation error: {}", ex.getMessage());
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        
        ErrorResponse errorResponse = ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.BAD_REQUEST.value())
                .error("Validation Failed")
                .message("Invalid input data")
                .validationErrors(errors)
                .build();
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(errorResponse);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        log.error("Unexpected error: {}", ex.getMessage(), ex);
        ErrorResponse errorResponse = ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.INTERNAL_SERVER_ERROR.value())
                .error("Internal Server Error")
                .message("An unexpected error occurred")
                .build();
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(errorResponse);
    }
}
==============================================================================
package com.example.employeecrud.exception;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;
import java.util.Map;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class ErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private Map<String, String> validationErrors;
}
==================================================================
package com.example.employeecrud.controller;

import com.example.employeecrud.entity.Employee;
import com.example.employeecrud.exception.ResourceNotFoundException;
import com.example.employeecrud.service.EmployeeService;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.Arrays;
import java.util.List;
import java.util.Optional;

import static org.hamcrest.Matchers.hasSize;
import static org.hamcrest.Matchers.is;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.eq;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@ExtendWith(MockitoExtension.class)
@WebMvcTest(EmployeeController.class)
class EmployeeControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private EmployeeService employeeService;

    @Autowired
    private ObjectMapper objectMapper;

    private Employee employee1;
    private Employee employee2;
    private List<Employee> employeeList;

    @BeforeEach
    void setUp() {
        employee1 = new Employee(1L, "John", "Doe", "john.doe@example.com", 
                "IT", new BigDecimal("50000.00"), LocalDate.of(2023, 1, 15), 
                "123-456-7890", "123 Main St");
        
        employee2 = new Employee(2L, "Jane", "Smith", "jane.smith@example.com", 
                "HR", new BigDecimal("55000.00"), LocalDate.of(2023, 2, 20), 
                "098-765-4321", "456 Oak Ave");
        
        employeeList = Arrays.asList(employee1, employee2);
    }

    @Test
    void getAllEmployees_ShouldReturnEmployeeList() throws Exception {
        // Given
        when(employeeService.getAllEmployees()).thenReturn(employeeList);

        // When & Then
        mockMvc.perform(get("/api/employees"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$", hasSize(2)))
                .andExpect(jsonPath("$[0].id", is(1)))
                .andExpect(jsonPath("$[0].firstName", is("John")))
                .andExpected(jsonPath("$[0].lastName", is("Doe")))
                .andExpect(jsonPath("$[0].email", is("john.doe@example.com")))
                .andExpect(jsonPath("$[1].id", is(2)))
                .andExpect(jsonPath("$[1].firstName", is("Jane")));

        verify(employeeService, times(1)).getAllEmployees();
    }

    @Test
    void getEmployeeById_WhenEmployeeExists_ShouldReturnEmployee() throws Exception {
        // Given
        when(employeeService.getEmployeeById(1L)).thenReturn(employee1);

        // When & Then
        mockMvc.perform(get("/api/employees/1"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.id", is(1)))
                .andExpect(jsonPath("$.firstName", is("John")))
                .andExpect(jsonPath("$.lastName", is("Doe")))
                .andExpect(jsonPath("$.email", is("john.doe@example.com")))
                .andExpect(jsonPath("$.department", is("IT")))
                .andExpect(jsonPath("$.salary", is(50000.00)));

        verify(employeeService, times(1)).getEmployeeById(1L);
    }

    @Test
    void getEmployeeById_WhenEmployeeNotExists_ShouldReturnNotFound() throws Exception {
        // Given
        when(employeeService.getEmployeeById(999L))
                .thenThrow(new ResourceNotFoundException("Employee not found with id: 999"));

        // When & Then
        mockMvc.perform(get("/api/employees/999"))
                .andDo(print())
                .andExpect(status().isNotFound());

        verify(employeeService, times(1)).getEmployeeById(999L);
    }

    @Test
    void createEmployee_WithValidData_ShouldReturnCreatedEmployee() throws Exception {
        // Given
        Employee newEmployee = new Employee(null, "Alice", "Johnson", "alice.johnson@example.com", 
                "Finance", new BigDecimal("60000.00"), null, "111-222-3333", "789 Pine St");
        
        Employee savedEmployee = new Employee(3L, "Alice", "Johnson", "alice.johnson@example.com", 
                "Finance", new BigDecimal("60000.00"), LocalDate.now(), "111-222-3333", "789 Pine St");

        when(employeeService.createEmployee(any(Employee.class))).thenReturn(savedEmployee);

        // When & Then
        mockMvc.perform(post("/api/employees")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(newEmployee)))
                .andDo(print())
                .andExpect(status().isCreated())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.id", is(3)))
                .andExpect(jsonPath("$.firstName", is("Alice")))
                .andExpect(jsonPath("$.lastName", is("Johnson")))
                .andExpect(jsonPath("$.email", is("alice.johnson@example.com")))
                .andExpect(jsonPath("$.department", is("Finance")))
                .andExpect(jsonPath("$.salary", is(60000.00)));

        verify(employeeService, times(1)).createEmployee(any(Employee.class));
    }

    @Test
    void createEmployee_WithInvalidData_ShouldReturnBadRequest() throws Exception {
        // Given - Employee with missing required fields
        Employee invalidEmployee = new Employee(null, "", "", "invalid-email", 
                "", null, null, "", "");

        // When & Then
        mockMvc.perform(post("/api/employees")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(invalidEmployee)))
                .andDo(print())
                .andExpect(status().isBadRequest());

        verify(employeeService, never()).createEmployee(any(Employee.class));
    }

    @Test
    void updateEmployee_WithValidData_ShouldReturnUpdatedEmployee() throws Exception {
        // Given
        Employee updatedEmployeeDetails = new Employee(null, "John Updated", "Doe Updated", 
                "john.updated@example.com", "IT Updated", new BigDecimal("55000.00"), 
                null, "999-888-7777", "Updated Address");
        
        Employee updatedEmployee = new Employee(1L, "John Updated", "Doe Updated", 
                "john.updated@example.com", "IT Updated", new BigDecimal("55000.00"), 
                LocalDate.of(2023, 1, 15), "999-888-7777", "Updated Address");

        when(employeeService.updateEmployee(eq(1L), any(Employee.class))).thenReturn(updatedEmployee);

        // When & Then
        mockMvc.perform(put("/api/employees/1")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(updatedEmployeeDetails)))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.id", is(1)))
                .andExpect(jsonPath("$.firstName", is("John Updated")))
                .andExpect(jsonPath("$.lastName", is("Doe Updated")))
                .andExpect(jsonPath("$.email", is("john.updated@example.com")))
                .andExpect(jsonPath("$.department", is("IT Updated")))
                .andExpect(jsonPath("$.salary", is(55000.00)));

        verify(employeeService, times(1)).updateEmployee(eq(1L), any(Employee.class));
    }

    @Test
    void updateEmployee_WhenEmployeeNotExists_ShouldReturnNotFound() throws Exception {
        // Given
        Employee updatedEmployeeDetails = new Employee(null, "John", "Doe", 
                "john.doe@example.com", "IT", new BigDecimal("50000.00"), 
                null, "123-456-7890", "123 Main St");

        when(employeeService.updateEmployee(eq(999L), any(Employee.class)))
                .thenThrow(new ResourceNotFoundException("Employee not found with id: 999"));

        // When & Then
        mockMvc.perform(put("/api/employees/999")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(updatedEmployeeDetails)))
                .andDo(print())
                .andExpect(status().isNotFound());

        verify(employeeService, times(1)).updateEmployee(eq(999L), any(Employee.class));
    }

    @Test
    void deleteEmployee_WhenEmployeeExists_ShouldReturnNoContent() throws Exception {
        // Given
        doNothing().when(employeeService).deleteEmployee(1L);

        // When & Then
        mockMvc.perform(delete("/api/employees/1"))
                .andDo(print())
                .andExpect(status().isNoContent());

        verify(employeeService, times(1)).deleteEmployee(1L);
    }

    @Test
    void delete

    ========================================================================================================
    package com.example.employeecrud.controller;

import com.example.employeecrud.entity.Employee;
import com.example.employeecrud.exception.ResourceNotFoundException;
import com.example.employeecrud.service.EmployeeService;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.Arrays;
import java.util.List;
import java.util.Optional;

import static org.hamcrest.Matchers.hasSize;
import static org.hamcrest.Matchers.is;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.eq;
import static org.mockito.Mockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@ExtendWith(MockitoExtension.class)
@WebMvcTest(EmployeeController.class)
class EmployeeControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private EmployeeService employeeService;

    @Autowired
    private ObjectMapper objectMapper;

    private Employee employee1;
    private Employee employee2;
    private List<Employee> employeeList;

    @BeforeEach
    void setUp() {
        employee1 = new Employee(1L, "John", "Doe", "john.doe@example.com", 
                "IT", new BigDecimal("50000.00"), LocalDate.of(2023, 1, 15), 
                "123-456-7890", "123 Main St");
        
        employee2 = new Employee(2L, "Jane", "Smith", "jane.smith@example.com", 
                "HR", new BigDecimal("55000.00"), LocalDate.of(2023, 2, 20), 
                "098-765-4321", "456 Oak Ave");
        
        employeeList = Arrays.asList(employee1, employee2);
    }

    @Test
    void getAllEmployees_ShouldReturnEmployeeList() throws Exception {
        // Given
        when(employeeService.getAllEmployees()).thenReturn(employeeList);

        // When & Then
        mockMvc.perform(get("/api/employees"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$", hasSize(2)))
                .andExpect(jsonPath("$[0].id", is(1)))
                .andExpect(jsonPath("$[0].firstName", is("John")))
                .andExpected(jsonPath("$[0].lastName", is("Doe")))
                .andExpect(jsonPath("$[0].email", is("john.doe@example.com")))
                .andExpect(jsonPath("$[1].id", is(2)))
                .andExpect(jsonPath("$[1].firstName", is("Jane")));

        verify(employeeService, times(1)).getAllEmployees();
    }

    @Test
    void getEmployeeById_WhenEmployeeExists_ShouldReturnEmployee() throws Exception {
        // Given
        when(employeeService.getEmployeeById(1L)).thenReturn(employee1);

        // When & Then
        mockMvc.perform(get("/api/employees/1"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.id", is(1)))
                .andExpect(jsonPath("$.firstName", is("John")))
                .andExpect(jsonPath("$.lastName", is("Doe")))
                .andExpect(jsonPath("$.email", is("john.doe@example.com")))
                .andExpect(jsonPath("$.department", is("IT")))
                .andExpect(jsonPath("$.salary", is(50000.00)));

        verify(employeeService, times(1)).getEmployeeById(1L);
    }

    @Test
    void getEmployeeById_WhenEmployeeNotExists_ShouldReturnNotFound() throws Exception {
        // Given
        when(employeeService.getEmployeeById(999L))
                .thenThrow(new ResourceNotFoundException("Employee not found with id: 999"));

        // When & Then
        mockMvc.perform(get("/api/employees/999"))
                .andDo(print())
                .andExpect(status().isNotFound());

        verify(employeeService, times(1)).getEmployeeById(999L);
    }

    @Test
    void createEmployee_WithValidData_ShouldReturnCreatedEmployee() throws Exception {
        // Given
        Employee newEmployee = new Employee(null, "Alice", "Johnson", "alice.johnson@example.com", 
                "Finance", new BigDecimal("60000.00"), null, "111-222-3333", "789 Pine St");
        
        Employee savedEmployee = new Employee(3L, "Alice", "Johnson", "alice.johnson@example.com", 
                "Finance", new BigDecimal("60000.00"), LocalDate.now(), "111-222-3333", "789 Pine St");

        when(employeeService.createEmployee(any(Employee.class))).thenReturn(savedEmployee);

        // When & Then
        mockMvc.perform(post("/api/employees")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(newEmployee)))
                .andDo(print())
                .andExpect(status().isCreated())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.id", is(3)))
                .andExpect(jsonPath("$.firstName", is("Alice")))
                .andExpect(jsonPath("$.lastName", is("Johnson")))
                .andExpect(jsonPath("$.email", is("alice.johnson@example.com")))
                .andExpect(jsonPath("$.department", is("Finance")))
                .andExpect(jsonPath("$.salary", is(60000.00)));

        verify(employeeService, times(1)).createEmployee(any(Employee.class));
    }

    @Test
    void createEmployee_WithInvalidData_ShouldReturnBadRequest() throws Exception {
        // Given - Employee with missing required fields
        Employee invalidEmployee = new Employee(null, "", "", "invalid-email", 
                "", null, null, "", "");

        // When & Then
        mockMvc.perform(post("/api/employees")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(invalidEmployee)))
                .andDo(print())
                .andExpect(status().isBadRequest());

        verify(employeeService, never()).createEmployee(any(Employee.class));
    }

    @Test
    void updateEmployee_WithValidData_ShouldReturnUpdatedEmployee() throws Exception {
        // Given
        Employee updatedEmployeeDetails = new Employee(null, "John Updated", "Doe Updated", 
                "john.updated@example.com", "IT Updated", new BigDecimal("55000.00"), 
                null, "999-888-7777", "Updated Address");
        
        Employee updatedEmployee = new Employee(1L, "John Updated", "Doe Updated", 
                "john.updated@example.com", "IT Updated", new BigDecimal("55000.00"), 
                LocalDate.of(2023, 1, 15), "999-888-7777", "Updated Address");

        when(employeeService.updateEmployee(eq(1L), any(Employee.class))).thenReturn(updatedEmployee);

        // When & Then
        mockMvc.perform(put("/api/employees/1")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(updatedEmployeeDetails)))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.id", is(1)))
                .andExpect(jsonPath("$.firstName", is("John Updated")))
                .andExpect(jsonPath("$.lastName", is("Doe Updated")))
                .andExpect(jsonPath("$.email", is("john.updated@example.com")))
                .andExpect(jsonPath("$.department", is("IT Updated")))
                .andExpect(jsonPath("$.salary", is(55000.00)));

        verify(employeeService, times(1)).updateEmployee(eq(1L), any(Employee.class));
    }

    @Test
    void updateEmployee_WhenEmployeeNotExists_ShouldReturnNotFound() throws Exception {
        // Given
        Employee updatedEmployeeDetails = new Employee(null, "John", "Doe", 
                "john.doe@example.com", "IT", new BigDecimal("50000.00"), 
                null, "123-456-7890", "123 Main St");

        when(employeeService.updateEmployee(eq(999L), any(Employee.class)))
                .thenThrow(new ResourceNotFoundException("Employee not found with id: 999"));

        // When & Then
        mockMvc.perform(put("/api/employees/999")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(updatedEmployeeDetails)))
                .andDo(print())
                .andExpect(status().isNotFound());

        verify(employeeService, times(1)).updateEmployee(eq(999L), any(Employee.class));
    }

    @Test
    void deleteEmployee_WhenEmployeeNotExists_ShouldReturnNotFound() throws Exception {
        // Given
        doThrow(new ResourceNotFoundException("Employee not found with id: 999"))
                .when(employeeService).deleteEmployee(999L);

        // When & Then
        mockMvc.perform(delete("/api/employees/999"))
                .andDo(print())
                .andExpect(status().isNotFound());

        verify(employeeService, times(1)).deleteEmployee(999L);
    }

    @Test
    void getEmployeesByDepartment_ShouldReturnEmployeeList() throws Exception {
        // Given
        List<Employee> itEmployees = Arrays.asList(employee1);
        when(employeeService.getEmployeesByDepartment("IT")).thenReturn(itEmployees);

        // When & Then
        mockMvc.perform(get("/api/employees/department/IT"))
                .andDo(print())
                .andExpected(status().isOk())
                .andExpected(content().contentType(MediaType.APPLICATION_JSON))
                .andExpected(jsonPath("$", hasSize(1)))
                .andExpected(jsonPath("$[0].department", is("IT")));

        verify(employeeService, times(1)).getEmployeesByDepartment("IT");
    }

    @Test
    void searchEmployeesByName_ShouldReturnEmployeeList() throws Exception {
        // Given
        List<Employee> searchResults = Arrays.asList(employee1);
        when(employeeService.searchEmployeesByName("John")).thenReturn(searchResults);

        // When & Then
        mockMvc.perform(get("/api/employees/search?name=John"))
                .andDo(print())
                .andExpected(status().isOk())
                .andExpected(content().contentType(MediaType.APPLICATION_JSON))
                .andExpected(jsonPath("$", hasSize(1)))
                .andExpected(jsonPath("$[0].firstName", is("John")));

        verify(employeeService, times(1)).searchEmployeesByName("John");
    }

    @Test
    void getEmployeeByEmail_WhenEmployeeExists_ShouldReturnEmployee() throws Exception {
        // Given
        when(employeeService.getEmployeeByEmail("john.doe@example.com"))
                .thenReturn(Optional.of(employee1));

        // When & Then
        mockMvc.perform(get("/api/employees/email/john.doe@example.com"))
                .andDo(print())
                .andExpected(status().isOk())
                .andExpected(content().contentType(MediaType.APPLICATION_JSON))
                .andExpected(jsonPath("$.email", is("john.doe@example.com")));

        verify(employeeService, times(1)).getEmployeeByEmail("john.doe@example.com");
    }

    @Test
    void getEmployeeByEmail_WhenEmployeeNotExists_ShouldReturnNotFound() throws Exception {
        // Given
        when(employeeService.getEmployeeByEmail("nonexistent@example.com"))
                .thenReturn(Optional.empty());

        // When & Then
        mockMvc.perform(get("/api/employees/email/nonexistent@example.com"))
                .andDo(print())
                .andExpected(status().isNotFound());

        verify(employeeService, times(1)).getEmployeeByEmail("nonexistent@example.com");
    }Employee_WhenEmployeeExists_ShouldReturnNoContent() throws Exception {
        // Given
        doNothing().when(employeeService).deleteEmployee(1L);

        // When & Then
        mockMvc.perform(delete("/api/employees/1"))
                .andDo(print())
                .andExpect(status().isNoContent());

        verify(employeeService, times(1)).deleteEmployee(1L);
    }

    @Test
    void delete
  =========================================================================================================
  package com.example.employeecrud.service;

import com.example.employeecrud.entity.Employee;
import com.example.employeecrud.exception.ResourceNotFoundException;
import com.example.employeecrud.repository.EmployeeRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.Arrays;
import java.util.List;
import java.util.Optional;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.anyString;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class EmployeeServiceTest {

    @Mock
    private EmployeeRepository employeeRepository;

    @InjectMocks
    private EmployeeService employeeService;

    private Employee employee1;
    private Employee employee2;
    private Employee newEmployee;

    @BeforeEach
    void setUp() {
        employee1 = new Employee(1L, "John", "Doe", "john.doe@example.com", 
                "IT", new BigDecimal("50000.00"), LocalDate.of(2023, 1, 15), 
                "123-456-7890", "123 Main St");
        
        employee2 = new Employee(2L, "Jane", "Smith", "jane.smith@example.com", 
                "HR", new BigDecimal("55000.00"), LocalDate.of(2023, 2, 20), 
                "098-765-4321", "456 Oak Ave");
        
        newEmployee = new Employee(null, "Alice", "Johnson", "alice.johnson@example.com", 
                "Finance", new BigDecimal("60000.00"), null, "111-222-3333", "789 Pine St");
    }

    @Test
    void getAllEmployees_ShouldReturnAllEmployees() {
        // Given
        List<Employee> expectedEmployees = Arrays.asList(employee1, employee2);
        when(employeeRepository.findAll()).thenReturn(expectedEmployees);

        // When
        List<Employee> actualEmployees = employeeService.getAllEmployees();

        // Then
        assertThat(actualEmployees).isNotNull();
        assertThat(actualEmployees).hasSize(2);
        assertThat(actualEmployees).containsExactly(employee1, employee2);
        verify(employeeRepository, times(1)).findAll();
    }

    @Test
    void getAllEmployees_WhenNoEmployeesExist_ShouldReturnEmptyList() {
        // Given
        when(employeeRepository.findAll()).thenReturn(Arrays.asList());

        // When
        List<Employee> actualEmployees = employeeService.getAllEmployees();

        // Then
        assertThat(actualEmployees).isNotNull();
        assertThat(actualEmployees).isEmpty();
        verify(employeeRepository, times(1)).findAll();
    }

    @Test
    void getEmployeeById_WhenEmployeeExists_ShouldReturnEmployee() {
        // Given
        when(employeeRepository.findById(1L)).thenReturn(Optional.of(employee1));

        // When
        Employee actualEmployee = employeeService.getEmployeeById(1L);

        // Then
        assertThat(actualEmployee).isNotNull();
        assertThat(actualEmployee.getId()).isEqualTo(1L);
        assertThat(actualEmployee.getFirstName()).isEqualTo("John");
        assertThat(actualEmployee.getLastName()).isEqualTo("Doe");
        assertThat(actualEmployee.getEmail()).isEqualTo("john.doe@example.com");
        verify(employeeRepository, times(1)).findById(1L);
    }

    @Test
    void getEmployeeById_WhenEmployeeNotExists_ShouldThrowResourceNotFoundException() {
        // Given
        Long nonExistentId = 999L;
        when(employeeRepository.findById(nonExistentId)).thenReturn(Optional.empty());

        // When & Then
        assertThatThrownBy(() -> employeeService.getEmployeeById(nonExistentId))
                .isInstanceOf(ResourceNotFoundException.class)
                .hasMessage("Employee not found with id: " + nonExistentId);
        
        verify(employeeRepository, times(1)).findById(nonExistentId);
    }

    @Test
    void createEmployee_WithValidData_ShouldCreateEmployee() {
        // Given
        Employee savedEmployee = new Employee(3L, "Alice", "Johnson", "alice.johnson@example.com", 
                "Finance", new BigDecimal("60000.00"), LocalDate.now(), "111-222-3333", "789 Pine St");
        
        when(employeeRepository.existsByEmail("alice.johnson@example.com")).thenReturn(false);
        when(employeeRepository.save(any(Employee.class))).thenReturn(savedEmployee);

        // When
        Employee actualEmployee = employeeService.createEmployee(newEmployee);

        // Then
        assertThat(actualEmployee).isNotNull();
        assertThat(actualEmployee.getId()).isEqualTo(3L);
        assertThat(actualEmployee.getFirstName()).isEqualTo("Alice");
        assertThat(actualEmployee.getEmail()).isEqualTo("alice.johnson@example.com");
        assertThat(actualEmployee.getHireDate()).isNotNull();
        
        verify(employeeRepository, times(1)).existsByEmail("alice.johnson@example.com");
        verify(employeeRepository, times(1)).save(any(Employee.class));
    }

    @Test
    void createEmployee_WithExistingEmail_ShouldThrowIllegalArgumentException() {
        // Given
        when(employeeRepository.existsByEmail("alice.johnson@example.com")).thenReturn(true);

        // When & Then
        assertThatThrownBy(() -> employeeService.createEmployee(newEmployee))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessage("Employee with email alice.johnson@example.com already exists");
        
        verify(employeeRepository, times(1)).existsByEmail("alice.johnson@example.com");
        verify(employeeRepository, never()).save(any(Employee.class));
    }

    @Test
    void createEmployee_WithNullHireDate_ShouldSetCurrentDate() {
        // Given
        Employee savedEmployee = new Employee(3L, "Alice", "Johnson", "alice.johnson@example.com", 
                "Finance", new BigDecimal("60000.00"), LocalDate.now(), "111-222-3333", "789 Pine St");
        
        when(employeeRepository.existsByEmail("alice.johnson@example.com")).thenReturn(false);
        when(employeeRepository.save(any(Employee.class))).thenReturn(savedEmployee);

        // When
        Employee actualEmployee = employeeService.createEmployee(newEmployee);

        // Then
        assertThat(actualEmployee.getHireDate()).isNotNull();
        verify(employeeRepository, times(1)).save(any(Employee.class));
    }

    @Test
    void updateEmployee_WithValidData_ShouldUpdateEmployee() {
        // Given
        Employee updatedDetails = new Employee(null, "John Updated", "Doe Updated", 
                "john.updated@example.com", "IT Updated", new BigDecimal("55000.00"), 
                null, "999-888-7777", "Updated Address");
        
        Employee updatedEmployee = new Employee(1L, "John Updated", "Doe Updated", 
                "john.updated@example.com", "IT Updated", new BigDecimal("55000.00"), 
                LocalDate.of(2023, 1, 15), "999-888-7777", "Updated Address");

        when(employeeRepository.findById(1L)).thenReturn(Optional.of(employee1));
        when(employeeRepository.existsByEmail("john.updated@example.com")).thenReturn(false);
        when(employeeRepository.save(any(Employee.class))).thenReturn(updatedEmployee);

        // When
        Employee actualEmployee = employeeService.updateEmployee(1L, updatedDetails);

        // Then
        assertThat(actualEmployee).isNotNull();
        assertThat(actualEmployee.getId()).isEqualTo(1L);
        assertThat(actualEmployee.getFirstName()).isEqualTo("John Updated");
        assertThat(actualEmployee.getLastName()).isEqualTo("Doe Updated");
        assertThat(actualEmployee.getEmail()).isEqualTo("john.updated@example.com");
        assertThat(actualEmployee.getDepartment()).isEqualTo("IT Updated");
        assertThat(actualEmployee.getSalary()).isEqualTo(new BigDecimal("55000.00"));
        
        verify(employeeRepository, times(1)).findById(1L);
        verify(employeeRepository, times(1)).existsByEmail("john.updated@example.com");
        verify(employeeRepository, times(1)).save(any(Employee.class));
    }

    @Test
    void updateEmployee_WhenEmployeeNotExists_ShouldThrowResourceNotFoundException() {
        // Given
        Long nonExistentId = 999L;
        Employee updatedDetails = new Employee();
        when(employeeRepository.findById(nonExistentId)).thenReturn(Optional.empty());

        // When & Then
        assertThatThrownBy(() -> employeeService.updateEmployee(nonExistentId, updatedDetails))
                .isInstanceOf(ResourceNotFoundException.class)
                .hasMessage("Employee not found with id: " + nonExistentId);
        
        verify(employeeRepository, times(1)).findById(nonExistentId);
        verify(employeeRepository, never()).save(any(Employee.class));
    }

    @Test
    void updateEmployee_WithExistingEmailForDifferentEmployee_ShouldThrowIllegalArgumentException() {
        // Given
        Employee updatedDetails = new Employee(null, "John", "Doe", 
                "jane.smith@example.com", "IT", new BigDecimal("50000.00"), 
                null, "123-456-7890", "123 Main St");

        when(employeeRepository.findById(1L)).thenReturn(Optional.of(employee1));
        when(employeeRepository.existsByEmail("jane.smith@example.com")).thenReturn(true);

        // When & Then
        assertThatThrownBy(() -> employeeService.updateEmployee(1L, updatedDetails))
                .isInstanceOf(IllegalArgumentException.class)
                .hasMessage("Employee with email jane.smith@example.com already exists");
        
        verify(employeeRepository, times(1)).findById(1L);
        verify(employeeRepository, times(1)).existsByEmail("jane.smith@example.com");
        verify(employeeRepository, never()).save(any(Employee.class));
    }

    @Test
    void updateEmployee_WithSameEmail_ShouldUpdateSuccessfully() {
        // Given
        Employee updatedDetails = new Employee(null, "John Updated", "Doe Updated", 
                "john.doe@example.com", "IT Updated", new BigDecimal("55000.00"), 
                null, "999-888-7777", "Updated Address");
        
        Employee updatedEmployee = new Employee(1L, "John Updated", "Doe Updated", 
                "john.doe@example.com", "IT Updated", new BigDecimal("55000.00"), 
                LocalDate.of(2023, 1, 15), "999-888-7777", "Updated Address");

        when(employeeRepository.findById(1L)).thenReturn(Optional.of(employee1));
        when(employeeRepository.save(any(Employee.class))).thenReturn(updatedEmployee);

        // When
        Employee actualEmployee = employeeService.updateEmployee(1L, updatedDetails);

        // Then
        assertThat(actualEmployee).isNotNull();
        assertThat(actualEmployee.getEmail()).isEqualTo("john.doe@example.com");
        
        verify(employeeRepository, times(1)).findById(1L);
        verify(employeeRepository, never()).existsByEmail(anyString());
        verify(employeeRepository, times(1)).save(any(Employee.class));
    }

    @Test
    void deleteEmployee_WhenEmployeeExists_ShouldDeleteEmployee() {
        // Given
        when(employeeRepository.findById(1L)).thenReturn(Optional.of(employee1));
        doNothing().when(employeeRepository).delete(employee1);

        // When
        employeeService.deleteEmployee(1L);

        // Then
        verify(employeeRepository, times(1)).findById(1L);
        verify(employeeRepository, times(1)).delete(employee1);
    }

    @Test
    void deleteEmployee_WhenEmployeeNotExists_ShouldThrowResourceNotFoundException() {
        // Given
        Long nonExistentId = 999L;
        when(employeeRepository.findById(nonExistentId)).thenReturn(Optional.empty());

        // When & Then
        assertThatThrownBy(() -> employeeService.deleteEmployee(nonExistentId))
                .isInstanceOf(ResourceNotFoundException.class)
                .hasMessage("Employee not found with id: " + nonExistentId);
        
        verify(employeeRepository, times(1)).findById(nonExistentId);
        verify(employeeRepository, never()).delete(any(Employee.class));
    }

    @Test
    void getEmployeesByDepartment_ShouldReturnEmployeesInDepartment() {
        // Given
        List<Employee> itEmployees = Arrays.asList(employee1);
        when(employeeRepository.findByDepartment("IT")).thenReturn(itEmployees);

        // When
        List<Employee> actualEmployees = employeeService.getEmployeesByDepartment("IT");

        // Then
        assertThat(actualEmployees).isNotNull();
        assertThat(actualEmployees).hasSize(1);
        assertThat(actualEmployees.get(0).getDepartment()).isEqualTo("IT");
        verify(employeeRepository, times(1)).findByDepartment("IT");
    }

    @Test
    void searchEmployeesByName_ShouldReturnMatchingEmployees() {
        // Given
        List<Employee> searchResults = Arrays.asList(employee1);
        when(employeeRepository.findByNameContaining("John")).thenReturn(searchResults);

        // When
        List<Employee> actualEmployees = employeeService.searchEmployeesByName("John");

        // Then
        assertThat(actualEmployees).isNotNull();
        assertThat(actualEmployees).hasSize(1);
        assertThat(actualEmployees.get(0).getFirstName()).contains("John");
        verify(employeeRepository, times(1)).findByNameContaining("John");
    }

    @Test
    void getEmployeeByEmail_WhenEmployeeExists_ShouldReturnEmployee() {
        // Given
        when(employeeRepository.findByEmail("john.doe@example.com")).thenReturn(Optional.of(employee1));

        // When
        Optional<Employee> actualEmployee = employeeService.getEmployeeByEmail("john.doe@example.com");

        // Then
        assertThat(actualEmployee).isPresent();
        assertThat(actualEmployee.get().getEmail()).isEqualTo("john.doe@example.com");
        verify(employeeRepository, times(1)).findByEmail("john.doe@example.com");
    }

    @Test
    void getEmployeeByEmail_WhenEmployeeNotExists_ShouldReturnEmpty() {
        // Given
        when(employeeRepository.findByEmail("nonexistent@example.com")).thenReturn(Optional.empty());

        // When
        Optional<Employee> actualEmployee = employeeService.getEmployeeByEmail("nonexistent@example.com");

        // Then
        assertThat(actualEmployee).isEmpty();
        verify(employeeRepository, times(1)).findByEmail("nonexistent@example.com");
    }
}
==================================================================================================================
package com.example.employeecrud.repository;

import com.example.employeecrud.entity.Employee;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;

import java.math.BigDecimal;
import java.time.LocalDate;
import java.util.List;
import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;

@DataJpaTest
class EmployeeRepositoryTest {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private EmployeeRepository employeeRepository;

    private Employee employee1;
    private Employee employee2;
    private Employee employee3;

    @BeforeEach
    void setUp() {
        employee1 = new Employee(null, "John", "Doe", "john.doe@example.com", 
                "IT", new BigDecimal("50000.00"), LocalDate.of(2023, 1, 15), 
                "123-456-7890", "123 Main St");
        
        employee2 = new Employee(null, "Jane", "Smith", "jane.smith@example.com", 
                "HR", new BigDecimal("55000.00"), LocalDate.of(2023, 2, 20), 
                "098-765-4321", "456 Oak Ave");
        
        employee3 = new Employee(null, "Bob", "Johnson", "bob.johnson@example.com", 
                "IT", new BigDecimal("52000.00"), LocalDate.of(2023, 3, 10), 
                "555-123-4567", "789 Pine St");

        // Persist test data
        entityManager.persistAndFlush(employee1);
        entityManager.persistAndFlush(employee2);
        entityManager.persistAndFlush(employee3);
    }

    @Test
    void findByEmail_WhenEmployeeExists_ShouldReturnEmployee() {
        // When
        Optional<Employee> found = employeeRepository.findByEmail("john.doe@example.com");

        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getFirstName()).isEqualTo("John");
        assertThat(found.get().getLastName()).isEqualTo("Doe");
        assertThat(found.get().getEmail()).isEqualTo("john.doe@example.com");
    }

    @Test
    void findByEmail_WhenEmployeeNotExists_ShouldReturnEmpty() {
        // When
        Optional<Employee> found = employeeRepository.findByEmail("nonexistent@example.com");

        // Then
        assertThat(found).isEmpty();
    }

    @Test
    void findByDepartment_ShouldReturnEmployeesInDepartment() {
        // When
        List<Employee> itEmployees = employeeRepository.findByDepartment("IT");

        // Then
        assertThat(itEmployees).hasSize(2);
        assertThat(itEmployees).extracting(Employee::getDepartment)
                .containsOnly("IT");
        assertThat(itEmployees).extracting(Employee::getFirstName)
                .containsExactlyInAnyOrder("John", "Bob");
    }

    @Test
    void findByDepartment_WhenNoDepartmentExists_ShouldReturnEmptyList() {
        // When
        List<Employee> employees = employeeRepository.findByDepartment("Finance");

        // Then
        assertThat(employees).isEmpty();
    }

    @Test
    void findByNameContaining_WithFirstName_ShouldReturnMatchingEmployees() {
        // When
        List<Employee> employees = employeeRepository.findByNameContaining("John");

        // Then
        assertThat(employees).hasSize(2);
        assertThat(employees).extracting(Employee::getFirstName)
                .containsExactlyInAnyOrder("John", "Johnson");
    }

    @Test
    void findByNameContaining_WithLastName_ShouldReturnMatchingEmployees() {
        // When
        List<Employee> employees = employeeRepository.findByNameContaining("Doe");

        // Then
        assertThat(employees).hasSize(1);
        assertThat(employees.get(0).getLastName()).isEqualTo("Doe");
    }

    @Test
    void findByNameContaining_WithPartialName_ShouldReturnMatchingEmployees() {
        // When
        List<Employee> employees = employeeRepository.findByNameContaining("oh");

        // Then
        assertThat(employees).hasSize(2);
        assertThat(employees).extracting(Employee::getFirstName)
                .containsExactlyInAnyOrder("John", "Johnson");
    }

    @Test
    void findByNameContaining_WhenNoMatch_ShouldReturnEmptyList() {
        // When
        List<Employee> employees = employeeRepository.findByNameContaining("xyz");

        // Then
        assertThat(employees).isEmpty();
    }

    @Test
    void existsByEmail_WhenEmailExists_ShouldReturnTrue() {
        // When
        boolean exists = employeeRepository.existsByEmail("john.doe@example.com");

        // Then
        assertThat(exists).isTrue();
    }

    @Test
    void existsByEmail_WhenEmailNotExists_ShouldReturnFalse() {
        // When
        boolean exists = employeeRepository.existsByEmail("nonexistent@example.com");

        // Then
        assertThat(exists).isFalse();
    }

    @Test
    void save_NewEmployee_ShouldPersistEmployee() {
        // Given
        Employee newEmployee = new Employee(null, "Alice", "Wilson", "alice.wilson@example.com", 
                "Finance", new BigDecimal("60000.00"), LocalDate.now(), 
                "777-888-9999", "321 Elm St");

        // When
        Employee savedEmployee = employeeRepository.save(newEmployee);

        // Then
        assertThat(savedEmployee.getId()).isNotNull();
        assertThat(savedEmployee.getFirstName()).isEqualTo("Alice");
        assertThat(savedEmployee.getEmail()).isEqualTo("alice.wilson@example.com");

        // Verify in database
        Employee foundEmployee = entityManager.find(Employee.class, savedEmployee.getId());
        assertThat(foundEmployee).isNotNull();
        assertThat(foundEmployee.getFirstName()).isEqualTo("Alice");
    }

    @Test
    void update_ExistingEmployee_ShouldUpdateEmployee() {
        // Given
        Employee existingEmployee = entityManager.find(Employee.class, employee1.getId());
        existingEmployee.setFirstName("John Updated");
        existingEmployee.setSalary(new BigDecimal("60000.00"));

        // When
        Employee updatedEmployee = employeeRepository.save(existingEmployee);

        // Then
        assertThat(updatedEmployee.getFirstName()).isEqualTo("John Updated");
        assertThat(updatedEmployee.getSalary()).isEqualTo(new BigDecimal("60000.00"));

        // Verify in database
        entityManager.clear();
        Employee foundEmployee = entityManager.find(Employee.class, existingEmployee.getId());
        assertThat(foundEmployee.getFirstName()).isEqualTo("John Updated");
        assertThat(foundEmployee.getSalary()).isEqualTo(new BigDecimal("60000.00"));
    }

    @Test
    void delete_ExistingEmployee_ShouldRemoveEmployee() {
        // Given
        Long employeeId = employee1.getId();

        // When
        employeeRepository.delete(employee1);

        // Then
        Employee deletedEmployee = entityManager.find(Employee.class, employeeId);
        assertThat(deletedEmployee).isNull();
    }

    @Test
    void findAll_ShouldReturnAllEmployees() {
        // When
        List<Employee> employees = employeeRepository.findAll();

        // Then
        assertThat(employees).hasSize(3);
        assertThat(employees).extracting(Employee::getEmail)
                .containsExactlyInAnyOrder(
                    "john.doe@example.com", 
                    "jane.smith@example.com", 
                    "bob.johnson@example.com"
                );
    }

    @Test
    void findById_WhenEmployeeExists_ShouldReturnEmployee() {
        // When
        Optional<Employee> found = employeeRepository.findById(employee1.getId());

        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getEmail()).isEqualTo("john.doe@example.com");
    }

    @Test
    void findById_WhenEmployeeNotExists_ShouldReturnEmpty() {
        // When
        Optional<Employee> found = employeeRepository.findById(999L);

        // Then
        assertThat(found).isEmpty();
    }
}
==================================================================================
spring:
  application:
    name: employee-crud-api
  
  datasource:
    url: jdbc:h2:mem:testdb
    driverClassName: org.h2.Driver
    username: sa
    password: password

  h2:
    console:
      enabled: true
      path: /h2-console

  jpa:
    database-platform: org.hibernate.dialect.H2Dialect
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        format_sql: true

  web:
    resources:
      add-mappings: true

server:
  port: 8080

logging:
  level:
    com.example.employeecrud: DEBUG
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE

management:
  endpoint:
    health:
      show-details: always
  endpoints:
    web:
      exposure:
        include: health,info,metrics

---
spring:
  config:
    activate:
      on-profile: test
  datasource:
    url: jdbc:
    
