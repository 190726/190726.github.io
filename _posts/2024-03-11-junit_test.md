---
title: "[java] junit and mockito"
excerpt: "자바 테스트 기법"

categories:
  - java-core
tags:
  - [java]

permalink: /java-core/jnit-mockito/

toc: true
toc_sticky: true

date: 2024-03-11
last_modified_at: 2024-03-11
---

## maven pom.xml 설정
```xml
        <!--mockito 코어-->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>3.12.4</version>
            <scope>test</scope>
        </dependency>
        <!--junit5-->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.6.2</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>5.6.2</version>
            <scope>test</scope>
        </dependency>
        <!--Mockito Extension(모키토 초기화 작업등 자동화)-->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-junit-jupiter</artifactId>
            <version>4.0.0</version>
            <scope>test</scope>
        </dependency>
        <!-- assertj (fluent api)-->
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>3.25.1</version>
            <scope>test</scope>
        </dependency>
```

## target code to Test

```java
private final ProductRepository productRepository;
    private final ProductGateway productGateway;

    public ProductService(ProductRepository productRepository, ProductGateway productGateway) {
        this.productRepository = productRepository;
        this.productGateway = productGateway;
    }

    public Product order(String productNumber, int orderQuantity){

         Product product = productRepository.findById(productNumber);
         Product orderedProduct = product.deductQuantity(orderQuantity); //재고 차감

         productGateway.exchange(orderedProduct);
         productRepository.update(orderedProduct);

         return orderedProduct;
    }

    public void orderHealthCheck(String code){
        if(code.equals("TEST")){
            Product orderedProduct = new Product("999", 10);
            productGateway.exchange(orderedProduct);
            productRepository.update(orderedProduct);
        }else{
            throw new IllegalArgumentException("Invalid arguments for Health Check");
        }
    }

    public List<Product> sampleProducts() {
        return Arrays.asList(new Product("001", 100)
                , new Product("002", 200)
                , new Product("003", 300));
    }
``

## test code
```java
@ExtendWith(MockitoExtension.class)
public class MockTest{

    @Mock //목 객체 생성
    ProductRepository productRepository;

    @Mock
    ProductGateway productGateway;

    @InjectMocks //목 객체를 주입 받을 객체
    ProductService productService;

    @Captor //gateway 로 전달된 product 캡처
    ArgumentCaptor<Product> exchangedProduct;

    @BeforeEach
    public void setup() {
        /*Extension 을 사용 하여 아래 생략*/
        // MockitoAnnotations.initMocks(this);
    }

    @Test
    @DisplayName("기본 테스트")
    void test(){
        //given
        Product product = new Product("001", 10);
        given(productRepository.findById("001")).willReturn(product); //BDD style
        //when
        Product orderedProduct = productService.order("001", 3);
        //then
        assertThat(orderedProduct)
                .extracting("productNumber", "quantity")
                .containsExactlyInAnyOrder("001",7);
        //verify() ->  BDD style
        then(productGateway).should(times(1)).exchange(exchangedProduct.capture());
        Product value = exchangedProduct.getValue();// 메서드 내 호출되는 메서드에 전달 파라미터 검증
        assertThat(value.getQuantity()).isEqualTo(7);
    }

    @Test
    @DisplayName("오류 처리 테스트")
    void exceptionTest(){
        /*
         * given
         */
        String productNumber = "001";
        int quantity = 5;
        int orderQuantity = 10;

        Product product = new Product(productNumber, quantity);
        given(productRepository.findById(productNumber)).willReturn(product); //BDD style
        /*
         * when
         */
        Assertions.assertThrows(IllegalStateException.class, () -> productService.order(productNumber, orderQuantity));
        assertThatExceptionOfType(IllegalStateException.class)
                .isThrownBy(() -> productService.order(productNumber, orderQuantity))
                //.havingCause()
                .withMessage("재고 부족!");
        assertThatThrownBy(() -> productService.order(productNumber, orderQuantity))
                .isInstanceOf(IllegalStateException.class)
                .hasMessageContaining("재고 부족");
    }


    @Test
    @DisplayName("willThrow 테스트")
    void exceptionAtGateway(){
        //given
        Product product = new Product("001", 10);
        Product exceptionProduct = new Product("001", 0);
        given(productRepository.findById("001")).willReturn(product); //BDD style
        willThrow(new RuntimeException()).given(productGateway).exchange(eq(exceptionProduct));
        //when
        try{
            productService.order("001", 10);
            fail("runtimeException");
        }catch(RuntimeException e){
        }
        //then
        then(productRepository).should(never()).update(any());
    }

    @Test
    @DisplayName("순서대로 호출되었는지, 내부 호출된 메서드 파라미터 검증하기")
    void voidTest(){
        productService.orderHealthCheck("TEST");

        InOrder inOrder = inOrder(productGateway, productRepository);
        //순서대로 호출되었는지 검증
        inOrder.verify(productGateway).exchange(any());
        inOrder.verify(productRepository).update(any());

        //void 메서드 내부에서 호출된 파라미터 검증
        then(productGateway).should(times(1)).exchange(exchangedProduct.capture());
        Product capturedProduct = exchangedProduct.getValue();
        assertThat(capturedProduct)
                .extracting("productNumber", "quantity")
                .containsExactlyInAnyOrder("999",10);
    }

    @Test
    void listTest(){
        List<Product> products = productService.sampleProducts();
        assertThat(products).hasSize(3)
                .extracting("productNumber", "quantity")
                .containsExactlyInAnyOrder(tuple("001", 100), tuple("002", 200), tuple("003", 300));
    }

    @Test
    void assertJTest(){
        Product testedProduct = new Product("123", 100);
        Product testedProduct1 = new Product("001", 50);
        Product testedProduct2 = new Product("002", 30);
        //기본 검증
        assertThat(testedProduct.getProductNumber()).startsWith("1");
        assertThat(testedProduct.getProductNumber()).contains("2");
        assertThat(testedProduct.getProductNumber()).endsWith("3");

        //리스트 검증
        List<Product> lists = Arrays.asList(testedProduct, testedProduct1, testedProduct2);
        assertThat(lists).allMatch(p -> p.getProductNumber().length()==3)
                .anyMatch(p -> p.getQuantity() >= 100)
                .noneMatch(p -> p.getProductNumber().equals("999"));
        assertThat(lists).map(Product::getProductNumber).contains("123", "001");
    }

    @Test
    void numberTest() {
        BigDecimal num1 = new BigDecimal("3.14");
        System.out.println(num1.toString());
        assertThat(num1).isGreaterThan(new BigDecimal("3"));

        int a = 10;
        assertThat(a).isPositive().isGreaterThan(9).isLessThan(11);
    }

    @Test
    void calendarTest(){
        assertThat(LocalDate.of(2024, 3, 31)).hasYear(2024).hasMonth(Month.MARCH).hasDayOfMonth(31);
        assertThat(LocalTime.of(23, 17, 59, 5)).hasHour(23)
                .hasMinute(17)
                .hasSecond(59)
                .hasNano(5);

        assertThat(LocalDateTime.of(2000, 12, 31, 23, 17, 59, 5)).hasYear(2000)
                .hasMonth(Month.DECEMBER)
                .hasMonthValue(12)
                .hasDayOfMonth(31)
                .hasHour(23)
                .hasMinute(17)
                .hasSecond(59)
                .hasNano(5);
    }
}
```

## spring mvc 
```java
@ActiveProfiles("test")
@WebMvcTest(EmpController.class)
class EmpControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Autowired
    ObjectMapper objectMapper;

    @MockBean
    EmpRepository empRepository;

    @MockBean
    DeptRepository deptRepository;

    @DisplayName("EMP 등록 하기")
    @Test
    void createRequest() throws Exception {

        when(empRepository.save(any())).thenReturn(Emp.builder().empNo("9999").build());

        EmpRequest empRequest = new EmpRequest("9999", "이상국",44, "A001");

        System.out.println(empRepository);
        mockMvc.perform(post("/emp")
                    .content(objectMapper.writeValueAsString(empRequest))
                    .contentType(MediaType.APPLICATION_JSON))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.empNo").value("9999"));
    }

    @Test
    void find() throws Exception {
        EmpResponse empResponse1 = new EmpResponse("0001", "이상국", "인사팀");
        EmpResponse empResponse2 = new EmpResponse("0002", "김상국", "인사팀");
        when(empRepository.findByAgeAndRegion(any(), any()))
                .thenReturn(List.of(empResponse1, empResponse2));

        mockMvc.perform(get("/emp/where")
                        .param("age", "42")
                        .param("region", "서울"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$").isArray())
                .andExpect(jsonPath("$[0].empId").value("0001"));
    }
}
```

## pareameterize test
1. pom.xml 에 의존성 추가
```xml
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter-params</artifactId>
  <version>5.6.2</version>
  <scope>test</scope>
</dependency>
```

```java
@ParameterizedTest
    @ValueSource(ints = {2, 4, 6})
    void parameterizedTest(int index){
        Assertions.assertThat(index % 2 == 0).isTrue();
    }

    @ParameterizedTest
    @CsvSource(value = {"1:true", "2:true", "3:false"}, delimiter = ':')
    void csvSourceTest(int index, boolean expected){
        System.out.println(index);
        System.out.println(expected);
    }

    @ParameterizedTest
    @CsvFileSource(resources = "/file.csv") //경로는 /resources 부터 시작
    void csvFileSource(String name, int amount){
        System.out.println(name);
        System.out.println(amount);
    }

    @ParameterizedTest
    @MethodSource("generateData")
    void methodSourceTest(List<Integer> list1, List<String> list2, LocalDate date){
        System.out.println(list1);
        System.out.println(date);
        System.out.println(list2);
    }

    static Stream<Arguments> generateData(){
        return Stream.of(
             Arguments.of(Arrays.asList(1, 2, 3), Arrays.asList("a", "b", "c"), LocalDate.now())
            ,Arguments.of(Arrays.asList(4, 5, 6), Arrays.asList("d", "e", "f"), LocalDate.of(2024,3,11))
        );
    }
```

