---
layout: post
title: Springboot 에서 test code 작성하기 1편 - 통합테스트
tags: [Test]
categories : [Java]
comments: true
---
>#### Springboot 에서 test code 작성하기 시리즈
* [Springboot 에서 test code 작성하기 1편 - 통합테스트(MVC)](https://sehajyang.github.io/java/2019/04/05/springboot-test-code-1.html)
* Springboot 에서 test code 작성하기 2편 - 단위테스트(Service)
* [Springboot 에서 test code 작성하기 3편 - assetThat이 중복되는 테스트와 Spock](https://sehajyang.github.io/java/2019/04/05/springboot-test-code-3.html)

최근 사내 프로젝트에 테스트 코드를 작성할 기회가 있었다.  
컨트롤러는 운영 환경과 비슷하게 테스트 하기 위해 통합 테스트, 서비스는 의존성을 줄이고 해당 서비스의 목표에만 집중하기 위해 단위테스트를 하기로 결정했다.  
목표는 해당 컨트롤러의 메소드가 잘 동작하는지(요청을 잘 보내고 예상한 응답을 잘 받는지), 서비스가 개별적으로 잘 동작하는지 확인하는 것 이다.  

테스트에 관한 글을 찾아보던 중 우아한형제들 기술블로그에서 테스트 메소드의 이름을 한글로 짓는다는 것을 알게 되었다.  
테스트가 실패했을 때 어느 기능에서 오류가 났는지 직관적으로 알 수 있어 좋은 것 같아서 테스트 메소드 명을 한글로 작성했다.  

### 1. MVC 컨트롤러 통합 테스트
목표는 고객 정보를 조회 및 등록하는 메소드를 통합테스트 하는 것 이다.  
컨트롤러 통합 테스트는 클래스 상단에 `@SpringbootTest` 을 선언한다.  
모든 Bean을 올리는 것으로 다른 테스트에 비해 테스트 시간이 오래 소요되지만 운영환경과 비슷하게 테스트 할 수 있다.  
따라서 클래스에 선언한 어노테이션은 다음과 같다.  

| 어노테이션           | 설명             |     
| --------------- | ------------------- |
| @RunWith(SpringRunner.class) | 테스트를 실행하기 위해 참조하는 클래스(SpringRunner)    |      
| @AutoConfigureWebMvc    | MVC와 관련된 Bean을 올린다     |
| @Transactional    | 테스트 후 DB 롤백     |
| @Ignore            | 실제로 실행할 필요가 없기 때문에 Ignore를 선언     |
| @SpringBootTest            | 모든 Bean을 올리는 통합 테스트      | 

~~~java
/**
 * @author sehajyang
 * @date 2019-04-05
 */

@RunWith(SpringRunner.class) 
@AutoConfigureWebMvc 
@Transactional 
@Ignore 
@SpringBootTest 
public class CustomerControllerTest {

    @Autowired
    private MockMvc mvc;

    @Autowired
    private WebApplicationContext webApplicationContext;

    @SpyBean
    private CustomerService customerService;

    @Autowired
    private ObjectMapper objectMapper;

    private MockHttpSession session = new MockHttpSession();

    private MockHttpServletRequest request = new MockHttpServletRequest();

~~~


`@SpyBean`은 해당 객체를 주입받아 사용하다 given을 주면 선언한 해당 기능으로 동작하는 Bean 이다.  

고객정보를 조회 및 등록하는 메소드가 있는 도메인 인 `CustomerController` 가 있다.  
아래 코드는 예시입니다.  
~~~java
@Controller
@RequestMapping("customer")
public class CustomerController {
    @Autowired
    CustomerService customerService;

    @GetMapping("")
    public ModelAndView getCustomerList(HttpSession session, HttpServletRequest request){
        ModelAndView mav = new ModelAndView("customerListPage");

        List<Customer> customerList = customerService.getCustomerList();
        mav.addObject("customerList", customerList);

        return mav;
    }

    @PostMapping("")
    public Object regCustomerData(@RequestBody Customer customer, 
                                    HttpSession session, HttpServletRequest request){
        // 기존에 해당 유저가 있는지 확인후 등록하는 메소드 (이렇게 하면 안됩니다(예시일뿐!))
        int result;                        
        int userExist = shopService.getCustomerByCustno(customer.getCustno(1));

        if(userExist < 1){
            return Constant.RESULT_FAIL;
        }else{
            result = shopService.regCustomerData(customer);
        }
        return (result > 0) ? Constant.RESULT_SUCCESS : Constant.RESULT_FAIL;
    }
}

~~~

코드상엔 없지만 `CustomerController`의 모든 메소드의 session과 header에는 특정 값이 셋팅되어 있어야 한다고 가정한다.  
따라서 위 `getCustomerList` 를 테스트 하는 코드는 다음과 같다.  

~~~java
@Before //테스트 수행시 실행
    public void setUp() {
        //Integration Test
        this.mvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
                .dispatchOptions(true) //HTTP TRACE 요청을 디스패치할지 여부를 설정 (default는 false)
                .alwaysDo(print())
                .build();

        session.setAttribute("Id", "셋팅 값");
        request.addHeader("X-FORWARDED-FOR", "셋팅 값");
    }

    @Test
    public void 고객리스트_조회가_정상적으로_되어야한다() throws Exception {
        List<Customer> customerList = customerService.getCustomerList();

        this.mvc.perform(get("/customer")
                    .session(session)
                    .header("X-FORWARDED-FOR", request.getHeader("X-FORWARDED-FOR"))
                    )
                .andExpect(status().isOk()) // 200 return
                .andExpect(view().name("customerListPage")) // 리턴할 view page name
                .andExpect(model().attribute("customerList", customerList)); // 리턴할 model attribute name
              // redirect 리턴일 경우
              //.andExpect(redirectedUrl("/")) 
              //.andExpect(status().is3xxRedirection())
    }

    @Test
    public void 고객정보_등록이_예외없이_되어야한다() throws Exception {
        Customer customer = new Customer("1","sehajyang"); // Customer(custno, custname)

        given(customerService.getCustomerByCustno(customer.getCustno())).willReturn(1); // getCustomerByCustno 의 result는 1을 리턴

        this.mvc.perform(post("/customer")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(customer)) // JSON으로 만들어준다
                    .session(session)
                    .header("X-FORWARDED-FOR", request.getHeader("X-FORWARDED-FOR"))
                    )
                .andExpect(status().isOk())
                .andExpect(content().string("SUCCESS"))
                .andReturn()
                .getResponse();
    }
~~~
그 밖에도 `param()` 등 으로 파라미터를 넘길수도 있다.  
이렇게 Controller를 통합테스트 를 실행해 전체 플로우를 테스트 할 수 있다.  

`@SessionAttributes`를 사용하는 방법도 있는데 이 경우엔 테스트 코드에서 `requestAttr()` 대신 `sessionAttr()`을 사용할 수 있다.    
이 방법은 [이곳](https://www.petrikainulainen.net/programming/spring-framework/integration-testing-of-spring-mvc-applications-forms/)의 예제로 확인할 수 있다.  

만약    
~~~java
    @PostMapping("")
    public Object regCustomerData(Customer customer, 
                                    HttpSession session, HttpServletRequest request){
~~~
이렇게 DTO에 `@RequestBody`가 없는 도메인을 테스트하려면 조금 복잡해진다.  
나는 회사 코드의 POST 도메인 파라미터 DTO에 일부 `@RequestBody`가 없었고, `@RequestBody`를 추가 할 경우 문제가 생길 수 있었기 때문에 
기존 코드를 변경하지 않고 테스트 하는 방법을 찾아보았다.  

우선 [Spring에서는 권장하지 않는다](https://stackoverflow.com/questions/17143116/integration-testing-posting-an-entire-object-to-spring-mvc-controller/20926205) 그 이유는 아래와 같다.  
![img_1]({{ "/assets/img/blogpost/springboot-testcode1-1.JPG"}})  
대략 이렇게 이해할 수 있다.  
>통합 테스트의 주 목적 중 하나는 MockMvc 모델 객체가 Object 데이터로 바뀌었는지 확인하는 것 이다.  
NewObject(DTO)를 자동으로 변환하면 NewObject(DTO)가 실제 form과 호환되지 않는 문제가 있는데, 이러한 문제가 있는 파라미터는 테스트에서 제외되기 때문이다.  

그래서 `@RequestBody`가 없는 도메인의 DTO 파라미터는 십중팔구 넘어가지 않으며 null이된다.  
이것을 해결하기 위한 방법은 두가지 정도가 있다.  
### HttpClient 사용
~~~java 
@Test
    public void 고객정보_등록이_예외없이_되어야한다() throws Exception {
        Customer customer = new Customer("1","sehajyang"); // Customer(custno, custname)

        given(customerService.getCustomerByCustno(customer.getCustno())).willReturn(1); // getCustomerByCustno 의 result는 1을 리턴

        this.mvc.perform(post("/customer")
                    .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                    .content(EntityUtils.toString(new UrlEncodedFormEntity(Arrays.asList(
                            new BasicNameValuePair("custno", "1"),
                            new BasicNameValuePair("custname", "sehajyang")
                    )))))
                    .andExpect(status().isOk())
                    .andExpect(content().string("SUCCESS"))
                    .andReturn()
                    .getResponse();
    }
~~~

### buildUrlEncodedFormEntity() 이용
~~~java

@Test
    public void 고객정보_등록이_예외없이_되어야한다() throws Exception {
        Customer customer = new Customer("1","sehajyang"); // Customer(custno, custname)

        given(customerService.getCustomerByCustno(customer.getCustno())).willReturn(1); // getCustomerByCustno 의 result는 1을 리턴

        this.mvc.perform(post("/customer")
                    .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                    .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                                .content(buildUrlEncodedFormEntity(
                                "custno","1",
                                "custname","sehajyang" 
                    ))))
                    .andExpect(status().isOk())
                    .andExpect(content().string("SUCCESS"))
                    .andReturn()
                    .getResponse();
    }

private String buildUrlEncodedFormEntity(String... params) {
    if( (params.length % 2) > 0 ) {
       throw new IllegalArgumentException("Need to give an even number of parameters");
    }
    StringBuilder result = new StringBuilder();
    for (int i=0; i<params.length; i+=2) {
        if( i > 0 ) {
            result.append('&');
        }
        try {
            result.
            append(URLEncoder.encode(params[i], StandardCharsets.UTF_8.name())).
            append('=').
            append(URLEncoder.encode(params[i+1], StandardCharsets.UTF_8.name()));
        }
        catch (UnsupportedEncodingException e) {
            throw new RuntimeException(e);
        }
    }
    return result.toString();
 }
 ~~~
 
[Testing Form posts through MockMVC](https://stackoverflow.com/questions/36568518/testing-form-posts-through-mockmvc)에서 더 알아볼 수 있다. 

---

위 시리즈는 3부로 나뉘어 통합테스트, 단위테스트, spock를 사용한 테스트로 작성 될 예정입니다.  
많이 미숙하지만 지속적으로 알아가고 있습니다.  
혹 틀린 부분 혹은 궁금한 점이 있다면 댓글 부탁드리겠습니다 읽어주셔서 감사합니다!🙏  


참고자료
* [Mokito 공식 문서](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/mock/mockito)  
* [Mokito github wiki](https://github.com/mockito/mockito/wiki)  
* [Yun 님의 Test Guide](https://github.com/cheese10yun/spring-guide/blob/master/docs/test-guide.md)  




