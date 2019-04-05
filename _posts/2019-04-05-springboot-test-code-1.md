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

고객정보를 조회 및 등록하는 메소드가 있는 `CustomerController` 가 있다.  
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

        if(userExist != 1){
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
    public void 고객정보_등록_willReturn이_1이아니면_예외가_발생해야한다() throws Exception {
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

위 시리즈는 3부로 나뉘어 통합테스트, 단위테스트, spock를 사용한 테스트로 작성 될 예정입니다.  
많이 미숙하지만 지속적으로 알아가고 있습니다.  
혹 틀린 부분 혹은 궁금한 점이 있다면 댓글 부탁드리겠습니다 읽어주셔서 감사합니다!🙏  


참고자료
* [Mokito 공식 문서](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/mock/mockito)  
* [Mokito github wiki](https://github.com/mockito/mockito/wiki)  
* [Yun 님의 Test Guide](https://github.com/cheese10yun/spring-guide/blob/master/docs/test-guide.md)  




