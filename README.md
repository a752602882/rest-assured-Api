
# 总结

### 准备
```Java        
    @Before
    public void setUp() throws Exception {
    RestAssured.baseURI = "https://search.jd.com";     //定义了url,使用get("/xxx")就可以不写出完整的地址                
    RestAssured.port = 443;        //https才设置这个端口
    }
 ```
```Java 

   //这俩句话完全相同，
   //（1）post请求带参数
   //（2）基地址+"/greetXML"
   //（3）返回的值要匹配body里的参数
   //（4）then（）  返回一个可验证的响应，让您验证响应
given().parameters("firstName", "John", "lastName", "Doe").when().post("/greetXML").then().body("greeting.firstName", equalTo("John"));
with().parameters("firstName", "John", "lastName", "Doe").when().post("/greetXML").then().assertThat().body("greeting.firstName", equalTo("John"));

     get("http://api.douban.com/v2/book/1220562").then().assertThat().body("code", equalTo(100));
    
``` 
