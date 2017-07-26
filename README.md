
# 总结

### 准备
```Java        
    @Before
    public void setUp() throws Exception {
    RestAssured.baseURI = "https://search.jd.com";//定义了url,使用get("/xxx")就可以不写出完整的地址                
    RestAssured.port = 443;        //https才设置这个端口
    }
 ```
```Java 

   //这俩句话完全相同，
   //（1）post请求带参数
   //（2）基地址+"/greetXML"
   //（3）返回的值要匹配body里的参数
   //（4）then（）  返回一个可验证的响应，让您验证响应
given().parameters("firstName", "John", "lastName", "Doe").when().
           post("/greetXML").then().body("greeting.firstName", equalTo("John"));
with().parameters("firstName", "John", "lastName", "Doe").when().
           post("/greetXML").then().assertThat().body("greeting.firstName", equalTo("John"));
```
### Get

```Java
//get(String path, Object... pathParams) ==given.get(String path, Object... pathParams)
//Restassured.get方法 实际调用的是Restassured.given.get() (Restassured.given返回的RequestSpecification对象)
get("http://api.douban.com/v2/book/1220562").then().assertThat().body("code", equalTo(100));
//等效于
given.get("http://api.douban.com/v2/book/1220562").then().assertThat().body("code", equalTo(100))

``` 
#### ps:
 * get()请求  调用given.get()
 * given()返回RequestSpecification对象
 * given().get()调用RequestSpecification.get()
 * given().param() RequestSpecification.param()
 
### given语法详解

#### then式旧语法
```Java
given().
        param("x", "y").
when().
        get("/lotto").
then().
        statusCode(400).
        body("lotto.lottoId", equalTo(6));
```
#### expect式新语法（建议）
```Java
given().
        param("x", "y").
when().
        get("/lotto").
expect().
        assertThat().
        statusCode(400).
        body("lotto.lottoId", equalTo(6));
```
#### ps:

<br>　　`given() .`</br>
<br>　　　　请求参数,头区域：param,formParam,queryParam,pathParam,contentType...... </br>
<br>　　`when() .  `</br> 
<br>　　　　请求方式,路径参数区域：  post,get("pathParams")......</br>
<br>　　`expect . `</br>
<br>　　　　期望区域：statusCode，body，assertThat()......</br>

