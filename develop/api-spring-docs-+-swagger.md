# API 문서 통합하기 (spring docs + swagger)

MSA 구조로 변경하면서 각 도메인들의 API 문서들을 통합해보기로 했다.



먼저 기존의 API 문서들은 spring docs 로 만들어져 있는데

swagger 와의 차이가 무엇인지 간단하게 보면 아래와 같다.

|    | Spring Rest Docs  | Swagger                     |
| -- | ----------------- | --------------------------- |
| 장점 | 제품코드에 영향 없다.      | API 를 테스트 해 볼수 있는 화면을 제공한다. |
|    | 테스트가 성공해야 문서작성된다. | 적용하기 쉽다.                    |
| 단점 | 적용하기 어렵다.         | 제품코드에 어노테이션 추가해야한다.         |
|    |                   | 제품코드와 동기화가 안될수 있다.          |



하지만 나는 이 둘의 장점을 섞어서 만들기로 했다.



## build.gradle&#x20;

```gradle
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.2.0'
	id 'io.spring.dependency-management' version '1.1.5'
	id 'com.epages.restdocs-api-spec' version '0.19.0'
}

group = 'com.settlement'
version = '0.0.1-SNAPSHOT'

java {
	sourceCompatibility = '21'
}

repositories {
	mavenCentral()
}

bootJar {
	archiveBaseName = 'settlement'
	archiveFileName = 'settlement.jar'
	archiveVersion = '0.0.1'
}

dependencies {
	implementation group: 'org.springframework.boot', name: 'spring-boot-starter-web'
	implementation group: 'org.springframework.boot', name: 'spring-boot-starter-actuator'

	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'

	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'org.springframework.restdocs:spring-restdocs-mockmvc'
	testImplementation 'com.epages:restdocs-api-spec-mockmvc:0.19.0'
	testImplementation 'org.mockito:mockito-core'

	testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

ext {
	snippetsDir = file('build/generated-snippets')
}

openapi3 {
	setServer("http://settlement-api.com")
	title = "Settlement API"
	description = "Settlement API description"
	version = "0.1.0"
	format = "yaml"
	outputFileNamePrefix = "settlement"
}

test {
	useJUnitPlatform()
	outputs.dir snippetsDir
}
```



gradle openapi3 를 실행하면

test 단계를 거쳐 build/generated-snippets 에 snippets 들이 생성된다.



swagger 문서 내 테스트를 하기 위한 url로 setServer("http://settlement-api.com") 를 사용하고

yaml 형식의 파일로 생성된다.

<div align="left">

<figure><img src="../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

</div>



## Test 코드

```java
@ExtendWith(RestDocumentationExtension.class)
@WebMvcTest(controllers = SettlementController.class)
class SettlementControllerTest {

    @MockBean
    private SettlementService settlementService;

    @Autowired
    private MockMvc mockMvc;


    @BeforeEach
    void setUp(WebApplicationContext webApplicationContext, RestDocumentationContextProvider restDocumentation) {
        this.mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext)
                .apply(documentationConfiguration(restDocumentation)).build();
    }


    @Test
    void getSettlementTest() throws Exception {
        LocalDate now = LocalDate.of(2024, 6, 1);
        SettlementSearchParam param = SettlementSearchParam.of(1532591L, now, now, 1, 10);
        List<SettlementDocument> documents =
                List.of(SettlementDocument.builder().orderId(1532591).orderNum("1234567890").paymentSuccessYmd(now)
                        .build());
        Page<SettlementDocument> result = new PageImpl<>(documents, param.getPageable(), 1);

        given(settlementService.getSettlement(any())).willReturn(result);

        mockMvc.perform(get("/settlement").param("orderId", "1532591").param("startDate", "2024-06-01")
                        .param("endDate", "2024-06-01").param("page", "1").param("size", "10")).andExpect(status().isOk())
                .andDo(document("settlement-api", resource(
                        ResourceSnippetParameters.builder().tag("Settlement").summary("정산 데이터 조회")
                                .description("정산이 완료된 데이터만 조회합니다.")
                                .queryParameters(parameterWithName("orderId").description("order id").optional(),
                                        parameterWithName("startDate").description("조회 시작 날짜"),
                                        parameterWithName("endDate").description("조회 마지막 날짜"),
                                        parameterWithName("page").description("페이지").optional(),
                                        parameterWithName("size").description("페이지 사이즈").optional()).responseFields(
                                        fieldWithPath("totalCount").type(SimpleType.NUMBER).description("전체 데이터 개수"),
                                        fieldWithPath("totalPage").type(SimpleType.NUMBER).description("전체 페이지 개수"),
                                        fieldWithPath("contents").description("조회 데이터"),
                                        fieldWithPath("contents[].orderId").type(SimpleType.STRING).description("order Id"),
                                        fieldWithPath("contents[].orderNum").type(SimpleType.STRING).description("order num"),
                                        fieldWithPath("contents[].paymentSuccessYmd").type(SimpleType.STRING)
                                                .description("결제일")).build())));
    }
}
```

위 테스트를 기반으로 snippets 가 생성되고 yaml 파일이 만들어진다.

[**공식문서**](https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle/) 를 참고하세요



만들어진 파일은 아래에서 확인이 가능하다.

{% embed url="https://editor.swagger.io/" %}

### **그렇다면 이렇게 만들어진 도메인별 문서들을 어떻게 한곳에서 관리하고 볼 수 있을까?**

먼저 yaml 문서를 만들어 한곳에 모으는 작업이 필요하다.

그 작업을 나는 jenkins 에서 하기로 했는데

test 라는 단계를 만들어서 파일을 만들고 한곳에 모으는 작업을 하기로 했다.

<div data-full-width="false">

<figure><img src="../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

</div>

```groovy
stage('test')
{
    if(env.PROFILE == 'beta') {
        sh """
        #!/bin/bash
       
        export JAVA_HOME=/usr/lib/jvm/java21/
        chmod a+x ./gradlew
        ./gradlew openapi3

        aws s3 cp build/api-spec s3://files/docs/spec --recursive
    """
    }
}
```

실서버가 아닌 테스트 서버에서만 작업을 실행하도록 하고

생성된 문서를 aws 의 파일 저장소인 s3 에 파일을 취합한다.



## swagger-ui

그럼 취합된 문서들을 이쁜 swagger ui 로 만들어주자.

### index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link rel="stylesheet" type="text/css" href="https://unpkg.com/swagger-ui-dist@5.17.14/swagger-ui.css">
</head>
<body>
    <div id="swagger-ui"></div>
    <script src="https://unpkg.com/swagger-ui-dist@5.17.14/swagger-ui-bundle.js"></script>
    <script src="https://unpkg.com/swagger-ui-dist@5.17.14/swagger-ui-standalone-preset.js"></script>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>

    <script>
        window.onload = function () {
            axios.get('config.json')
                .then(function(res) {
                    const config = res.data;
                    console.log(config);
                    window.ui = SwaggerUIBundle({
                        dom_id: "#swagger-ui",
                        deepLinking: true,
                        presets: [
                            SwaggerUIBundle.presets.apis,
                            SwaggerUIStandalonePreset
                        ],
                        plugins: [
                            SwaggerUIBundle.plugins.DownloadUrl
                        ],
                        layout: "StandaloneLayout",
                        urls: config.shop
                    })
                })
        }
    </script>
</body>
</html>
```

swagger-ui 스크립트를 받고

config.json 에서 도메인별로 문서들을 묶어줄 수 있다.

### config.json

```json
{
  "shop": [
    {"url": "spec/settlement.yaml", "name": "settlement-api"},
    {"url": "spec/order.yaml", "name": "order-api"}
  ],
  "internal": [
    {"url": "spec/internal.yaml", "name": "internal-api"}
  ]
}
```





그리고 html 파일을 열어주면

<figure><img src="../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>

깔끔한 외모로 탄생한 문서를 확인할 수 있다.
