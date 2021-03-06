
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

#### then式新语法（建议）
```Java
given().
        param("x", "y").
when().
        get("/lotto").
then().
        statusCode(400).
        body("lotto.lottoId", equalTo(6));
```
#### expect式旧语法
```Java
given().
        param("x", "y").
expect().   //必须接在这里，不能放在when后面，表达上就不是很清晰
        assertThat().
        statusCode(400).
        body("lotto.lottoId", equalTo(6)).
when().
        get("/lotto");
;
```
#### ps:

<br>　　`given() .`</br>
<br>　　　　请求参数,头区域：param,formParam,queryParam,pathParam,contentType...... </br>
<br>　　`when() .  `</br> 
<br>　　　　请求方式,路径参数区域：  post,get("pathParams")......</br>
<br>　　`then() . `</br>
<br>　　　　期望区域：statusCode，body，assertThat()......</br>

#### Json example

```Java        
    @Before
    public void setUp() throws Exception {
    RestAssured.baseURI = "https://api.steampowered.com/IEconDOTA2_570";             
    RestAssured.port = 443;        //https才设置这个端口
    }
    
     given()
                .param("key","BAA464D3B432D062BEA99BA753214681").
     when()
                .get("/GetHeroes/v0001/").
     then()
                .body("result.status", equalTo(200),
                        "result.count", equalTo(113));// 不能拆分为2个body,哪样是错误的
                        
   
        
 ```
#### ps:
*  get()方法使用场景：没有参数的时候
*  given()方法使用场景：有参数的时候



```Java     
  ValidatableResponse resp =     given()
                .param("key","BAA464D3B432D062BEA99BA753214681").
                        when()
                .get("/GetHeroes/v0001/").
                        then();
        //判断返回Json数据的title
        resp.body("result.status", equalTo(200));
        //判断二级属性rating.max的值
        resp.body("result.count", equalTo(113));
 ```
如果要验证json里的很多数据怎么办呢？
>>写无数的的resp.body();
>>使用json模板一次性解决

```Java
      //设置默认的jsonsecema版本
       JsonSchemaFactory jsonSchemaFactory = JsonSchemaFactory.newBuilder().setValidationConfiguration(
       ValidationConfiguration.newBuilder().setDefaultVersion(DRAFTV3).freeze()).freeze();
       
       String url= JDTest.class.getClassLoader().getResource("match-schema.json").getPath();
       get(dotaMatchUrl).then().assertThat().body(matchesJsonSchemaInClasspath(url).using(jsonSchemaFactory));
 ```
 #### ps： 
 *  当然没必要非要在then后面使用，当我们获得http,https返回数据时候,分开判断，这种写法更加明确
 *  String json = given.....
 *  assertThat(json, matchesJsonSchemaInClasspath("greeting-schema.json"));
 
 #### XML栗子
 
 ```Java
      given().param("key","BAA464D3B432D062BEA99BA753214681")
                .when()
                .get("/GetHeroes/v0001")
                .then().assertThat().body("result.count",equalTo(113));
 ```
 * 虽然看上去和json一摸一样，但是xml在一些高级的用法上完全不同,就让我们先迷糊到吧
 
 
 ## Json & Xml   语法糖
 
 #### xml文件资源 
 
  ```Java
 <shopping>
    <category type="groceries">
        <item>Chocolate</item>
        <item>Coffee</item>
    </category>
    <category type="supplies">
        <item>Paper</item>
        <item quantity="4">Pens</item>
    </category>
    <category type="present">
        <item when="Aug 10">Kathryn's Birthday</item>
    </category>
</shopping>
  ```
   
  ```Java
   @Test
    public  void testShoppingXML(){
        //第一种方法
        ValidatableResponse response=given()
                .when()
                        .get("/test/Shopping.xml")
                .then()
                .body("shopping.category.find { it.@type == 'groceries' }.item",hasItems("Chocolate", "Coffee"));


       // 第二种方法
        String  response1  = get("/test/Shopping.xml").asString();
        List<String> groceries = from(response1).getList("shopping.category.find { it.@type == 'groceries' }.item");

       //第三种方法
        get("/test/Shopping.xml").then().body("**.find{it.@type == 'groceries'}",hasItems("Chocolate", "Coffee"));
   }
 
  ```
 * shopping-->category节点，找一个叫groceries的category节点，所以返回的值是category节点，节点下有许多元素
 
 
 #### Json文件资源
  ```Java
  {
  "store":{
   "book":[
        {
            "author":"Nigel Rees",
            "category":"reference",
            "price":8.95,
            "title":"Sayings of the Century"
        },
        {
        ......
        },
        {
        ......
        },
        {
         ......
         }
    ]
  }
}    
   ```
  ```Java    
 ValidatableResponse response=given()
                .when()
                .get("/test/store.json")
                .then()
                .body("store.book.findAll { it.price<10 }.title" +
                        "",hasItems("Chocolate", "Coffee"));
   ```
*  store-->book-->title  这个节点下的price<10
*  Json 语法{} 是用来放键值对    , [ ] 是用来放数组的 store.book[0].price &  store.book[1].price
可能检索出来有多个符合条件的，所以就要用findAll

  ```Java    
  /* 
     对 author属性的值求长度，collect{}和findAll这些方法类似，是一个集合,一个长度的集合
     最后判断这个集合所有元素之和（17，20，58）『sum()』是否大于50
  */
    given().when().get("/test/store.json").then()
               .body("store.book.author.collect {it.length() }.sum()",greaterThan(50));
               
   ```            
   
#### 响应内容返回类型
  ```Java 
   InputStream stream = get("/lotto").asInputStream();  
   byte[] byteArray = get("/lotto").asByteArray();
   String json = get("/lotto").asString();
   ```
   * 可见，响应内容可以通过输入流 ，二进制，字符串等形式返回
   
   
#### extract 提取响应信息
   
  ```Java
{
    "title" : "My Title",
     "_links": {
             "self": { "href": "/title" },
             "next": { "href": "/title?page=2" }
          }
}
//
Response response = 
given().
        param("param_name", "param_value").
when().
        get("/title").
then().
        contentType(JSON).
        body("title", equalTo("My Title")).
extract().
        response(); 

String nextTitleLink = response.path("_links.next.href");
String headerValue = response.header("headerName");
   ```
* extract 是提取的意思 ,提取响应的所有信息，用于以后的操作，比如说：返回的值是下一个请求的地址等...... 
