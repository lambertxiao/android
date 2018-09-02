# Android实现模拟登陆正方系统查成绩 

相信有很多小伙伴的学校在使用的都是正方系统作为教务系统，在平时我们会图方便利用各种微信公众号、超级课程表之类的东西查成绩，今天就跟大家示范一下如何自己编码模拟登陆正方系统并获得我们需要的信息。

本次教程需要备有如下知识：了解http协议、抓包、Java网络请求的写法、解析html页面的开源库Jsoup、以及方便网络操作的OkHttp（这个如果不熟悉可以用Android原生的HttpUrlConnection，但这里推荐使用OkHttp，因为写起来确实很方便啊，哈哈哈哈）

好，话不多说，马上开始！！！

![](http://i.imgur.com/NuJjuSF.png)

以我们学习的教务系统为例子，通常我们访问的链接如下：

	http://210.38.162.117

然后正方系统就会很诡异将我们访问的链接变成如下形式：

	http://210.38.162.117/(sxaiuuqeeivmbu555ha1bd55)/default2.aspx

中间有着24个随机的字符：iwpzdwm0xckdu055dwq5jb45

在这个了地方我纠结了很久，我本以为需要先通过访问

**http://210.38.162.117**

再获得变化后的

**http://210.38.162.117/(sxaiuuqeeivmbu555ha1bd55)/default2.aspx**

，然后再次发起链接，但经过多次实验后发现，特么这24个随机字符其实是可以固定写死的Orz，可就是说我们一开始就可以通过访问第二条链接实现登陆。

	String loginUrl = "http://210.38.162.117/(sxaiuuqeeivmbu555ha1bd55)/default2.aspx";

现在我们就要通过抓包的形式获得登陆时浏览器向服务器提供的数据，怎么抓包可以详情百度，这里简单地说一下，通常现在的浏览器都自带了抓包功能（个人喜欢用UC浏览器哈哈）

在浏览器中右键->审查元素（不同浏览器可能名字不同）

![](http://i.imgur.com/8MnfqnO.png)

抓包页面如图所示，不同浏览器的抓包功能也不一，有的好有的坏。

在登陆前先打开审查元素功能，输入用户名、密码后登陆

![](http://i.imgur.com/GdJXTcb.png)

登陆后，点击那个default2.aspx会看到旁边的一些请求头、回复头、表单数据。（看不懂的童鞋赶紧去恶补一下Http协议吧）

	__VIEWSTATE:dDwyODE2NTM0OTg7Oz4EdrhGRK2tAP2OLlvKpVtfzXS17g%3D%3D
	txtUserName:131110142
	TextBox2:******
	txtSecretCode:
	RadioButtonList1:%D1%A7%C9%FA
	Button1:
	lbLanguage:
	hidPdrs:
	hidsc:
这个就是我们表单的数据，实际上我们在向服务器发送请求时，只需要传过去这些数据就行了，

__VIEWSTATE	是一个固定的字符串，是可以写死的。（不同学校的系统不知会不会有所变化）

txtUserName	是学号

TextBox2	是密码，这里被我打了马赛克哈哈哈

txtSecretCode	是验证码

RadioButtonList1	后面那个字符串是学生，在提交的时候经过了Url编码

剩下的几个字段默认都是空，也不必在意它是什么了。

知道了访问所需要的字段，现在开始编码，还是用简单快速的OkHttp。

	String loginUrl = "http://210.38.162.117/(sxaiuuqeeivmbu555ha1bd55)/default2.aspx";

	public void login() throws UnsupportedEncodingException {
		//构建表单数据
        FormEncodingBuilder builder = new FormEncodingBuilder();
        builder.add("__VIEWSTATE", "dDwyODE2NTM0OTg7Oz4EdrhGRK2tAP2OLlvKpVtfzXS17g==");
        builder.add("txtUserName", "131110142");
        builder.add("TextBox2", "******");
        builder.add("txtSecretCode", "");
        builder.add("lbLanguage", "");
        builder.add("RadioButtonList1", URLEncoder.encode("学生", "gbk"));
        builder.add("Button1", "");
        builder.add("hidPdrs", "");
        builder.add("hidsc", "");
        RequestBody requestBody = builder.build();
		//发送post请求
        final Request request = new Request.Builder().url(loginUrl).post(requestBody).build();
        okHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Request request, IOException e) {

            }

			//获得相应后会执行的方法
            @Override
            public void onResponse(Response response) throws IOException {
                if (response.isSuccessful()) {
                    String urlString = response.request().urlString();
        			//登陆查询
                    loginScore(urlString);
                }
            }
        });
    }

上面代码在获取响应的时候，通过response.request().urlString()拿到了

	http://210.38.162.117/(sxaiuuqeeivmbu555ha1bd55)/xs_main.aspx?xh=131110142	

这样一条链接，这是干嘛的呢？首先你可以在浏览器的地址栏中看一看，在你登陆成功后是不是地址栏内容就变成了如上面相似的连接。

该链接作用很大，我们要执行后续操作如抓成绩，抓课程表都需要先通过它。

好了，先将它放着，我们继续点击信息查询中的成绩查询，
![](http://i.imgur.com/jJ0JuW9.png)

我们会发现这样一条链接

	String loginScoreUrl = "http://210.38.162.117/(sxaiuuqeeivmbu555ha1bd55)/xscjcx.aspx?xh=131110142&xm=%D0%A4%C9%D9%D0%C7&gnmkdm=N121605";
  
该链接是登陆成绩查询的链接，它带了三个参数：

xh	学号

xm	学名（就是你的名字经过Url编码后得到的字符串）

nmkdm	没看出是什么，但它是固定不变的

知道了这三个参数，我们继续走，再次发起访问，记得此时访问的时候必须要将

	http://210.38.162.117/(sxaiuuqeeivmbu555ha1bd55)/xs_main.aspx?xh=131110142
添加到请求头里面，否则访问会失败，失败，败（三遍了），同时要加上Host请求头
	
	private void loginScore(String urlString) {
        Request request = new Request.Builder()
                .url(loginScoreUrl)
                .addHeader("Referer", urlString)
                .addHeader("Host", "210.38.162.117")
                .build();

        okHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Request request, IOException e) {
            }

            @Override
            public void onResponse(Response response) throws IOException {
                if (response.isSuccessful()) {
            		//此时登陆成功后我们就可以开始查成绩了
                    searchScore();
                }
            }
        });
    }

还是类似的代码，我们需要向服务器提交几个数据

__EVENTTARGET	默认为空

__EVENTARGUMENT	也为空

__VIEWSTATE	这个东西丧尽天良了，竟然辣么长辣么长，不加还不行，有学.net的同学能否告诉我这究竟是什么鬼

ddlXN	学年

ddlXQ	学期

ddl_kcxz	不知是啥，为空

btn_xq	查询的是按的哪一项目

学期成绩是	%D1%A7%C6%DA%B3%C9%BC%A8

学年成绩是	%D1%A7%C4%EA%B3%C9%BC%A8

历年成绩是	%C0%FA%C4%EA%B3%C9%BC%A8

我在尝试的时候发现有点bug，就是无论查询学年成绩还是历年成绩，它都提示要学年学期一起输入。这里待解决。

	private void searchScore() throws UnsupportedEncodingException {
        FormEncodingBuilder builder = new FormEncodingBuilder();
        RequestBody requestBody =
                builder.add("__EVENTTARGET", "")
                        .add("__EVENTARGUMENT", "")
                        .add("__VIEWSTATE", "dDw1OTcxMjMxNzI7dDxwPGw8U29ydEV4cHJlcztzZmRjYms7ZGczO2R5YnlzY2o7U29ydERpcmU7eGg7c3RyX3RhYl9iamc7Y2pjeF9sc2I7enhjamN4eHM7PjtsPGtjbWM7XGU7YmpnO1xlO2FzYzsxMzExMTAxNDI7emZfY3hjanRqXzEzMTExMDE0Mjs7MTs+PjtsPGk8MT47PjtsPHQ8O2w8aTw0PjtpPDEwPjtpPDE5PjtpPDI0PjtpPDMyPjtpPDM0PjtpPDM2PjtpPDM4PjtpPDQwPjtpPDQyPjtpPDQ0PjtpPDQ2PjtpPDQ4PjtpPDUwPjs+O2w8dDx0PDt0PGk8MTc+O0A8XGU7MjAwMS0yMDAyOzIwMDItMjAwMzsyMDAzLTIwMDQ7MjAwNC0yMDA1OzIwMDUtMjAwNjsyMDA2LTIwMDc7MjAwNy0yMDA4OzIwMDgtMjAwOTsyMDA5LTIwMTA7MjAxMC0yMDExOzIwMTEtMjAxMjsyMDEyLTIwMTM7MjAxMy0yMDE0OzIwMTQtMjAxNTsyMDE1LTIwMTY7MjAxNi0yMDE3Oz47QDxcZTsyMDAxLTIwMDI7MjAwMi0yMDAzOzIwMDMtMjAwNDsyMDA0LTIwMDU7MjAwNS0yMDA2OzIwMDYtMjAwNzsyMDA3LTIwMDg7MjAwOC0yMDA5OzIwMDktMjAxMDsyMDEwLTIwMTE7MjAxMS0yMDEyOzIwMTItMjAxMzsyMDEzLTIwMTQ7MjAxNC0yMDE1OzIwMTUtMjAxNjsyMDE2LTIwMTc7Pj47Pjs7Pjt0PHQ8cDxwPGw8RGF0YVRleHRGaWVsZDtEYXRhVmFsdWVGaWVsZDs+O2w8a2N4em1jO2tjeHpkbTs+Pjs+O3Q8aTw3PjtAPOWFrOWFseW/heS/rjvkuJPkuJrlv4Xkv6475LiT5Lia6ZmQ6YCJO+S4k+S4muS7u+mAiTvlhazlhbHpmZDpgIk75YWs5YWx5Lu76YCJO1xlOz47QDwwMTswMjswMzswNDswNTswNjtcZTs+Pjs+Ozs+O3Q8cDxwPGw8VmlzaWJsZTs+O2w8bzxmPjs+Pjs+Ozs+O3Q8cDxwPGw8VmlzaWJsZTs+O2w8bzxmPjs+Pjs+Ozs+O3Q8cDxwPGw8VGV4dDs+O2w8XGU7Pj47Pjs7Pjt0PHA8cDxsPFRleHQ7VmlzaWJsZTs+O2w85a2m5Y+377yaMTMxMTEwMTQyO288dD47Pj47Pjs7Pjt0PHA8cDxsPFRleHQ7VmlzaWJsZTs+O2w85aeT5ZCN77ya6IKW5bCR5pifO288dD47Pj47Pjs7Pjt0PHA8cDxsPFRleHQ7VmlzaWJsZTs+O2w85a2m6Zmi77ya6K6h566X5py65a2m6ZmiO288dD47Pj47Pjs7Pjt0PHA8cDxsPFRleHQ7VmlzaWJsZTs+O2w85LiT5Lia77yaO288dD47Pj47Pjs7Pjt0PHA8cDxsPFRleHQ7VmlzaWJsZTs+O2w86L2v5Lu25bel56iLO288dD47Pj47Pjs7Pjt0PHA8cDxsPFRleHQ7PjtsPOS4k+S4muaWueWQkTo7Pj47Pjs7Pjt0PHA8cDxsPFRleHQ7VmlzaWJsZTs+O2w86KGM5pS/54+t77ya6K6h566X5py6MTMwNOePrTtvPHQ+Oz4+Oz47Oz47dDw7bDxpPDE+O2k8Mz47aTw1PjtpPDc+O2k8OT47aTwxMT47aTwxMz47aTwxNT47aTwxNz47aTwxOD47aTwxOT47aTwyMT47aTwyMz47aTwyNT47aTwyNz47aTwyOT47aTwzMT47aTwzMz47aTwzNT47aTwzNj47PjtsPHQ8cDxwPGw8VmlzaWJsZTs+O2w8bzxmPjs+Pjs+Ozs+O3Q8QDA8cDxwPGw8VmlzaWJsZTs+O2w8bzxmPjs+PjtwPGw8c3R5bGU7PjtsPERJU1BMQVk6bm9uZTs+Pj47Ozs7Ozs7Ozs7Pjs7Pjt0PDtsPGk8MTM+Oz47bDx0PEAwPDs7Ozs7Ozs7Ozs+Ozs+Oz4+O3Q8cDxwPGw8VGV4dDtWaXNpYmxlOz47bDzoh7Pku4rmnKrpgJrov4for77nqIvmiJDnu6nvvJo7bzx0Pjs+Pjs+Ozs+O3Q8QDA8cDxwPGw8UGFnZUNvdW50O18hSXRlbUNvdW50O18hRGF0YVNvdXJjZUl0ZW1Db3VudDtEYXRhS2V5czs+O2w8aTwxPjtpPDA+O2k8MD47bDw+Oz4+O3A8bDxzdHlsZTs+O2w8RElTUExBWTpibG9jazs+Pj47Ozs7Ozs7Ozs7Pjs7Pjt0PEAwPHA8cDxsPFZpc2libGU7PjtsPG88Zj47Pj47cDxsPHN0eWxlOz47bDxESVNQTEFZOm5vbmU7Pj4+Ozs7Ozs7Ozs7Oz47Oz47dDxAMDxwPHA8bDxWaXNpYmxlOz47bDxvPGY+Oz4+O3A8bDxzdHlsZTs+O2w8RElTUExBWTpub25lOz4+Pjs7Ozs7Ozs7Ozs+Ozs+O3Q8QDA8Ozs7Ozs7Ozs7Oz47Oz47dDxAMDxwPHA8bDxWaXNpYmxlOz47bDxvPGY+Oz4+O3A8bDxzdHlsZTs+O2w8RElTUExBWTpub25lOz4+Pjs7Ozs7Ozs7Ozs+Ozs+O3Q8QDA8cDxwPGw8VmlzaWJsZTs+O2w8bzxmPjs+PjtwPGw8c3R5bGU7PjtsPERJU1BMQVk6bm9uZTs+Pj47Ozs7Ozs7Ozs7Pjs7Pjt0PEAwPHA8cDxsPFZpc2libGU7PjtsPG88Zj47Pj47Pjs7Ozs7Ozs7Ozs+Ozs+O3Q8QDA8cDxwPGw8VmlzaWJsZTs+O2w8bzxmPjs+PjtwPGw8c3R5bGU7PjtsPERJU1BMQVk6bm9uZTs+Pj47Ozs7Ozs7Ozs7Pjs7Pjt0PEAwPHA8cDxsPFZpc2libGU7PjtsPG88Zj47Pj47cDxsPHN0eWxlOz47bDxESVNQTEFZOm5vbmU7Pj4+Ozs7Ozs7Ozs7Oz47Oz47dDxAMDw7QDA8OztAMDxwPGw8SGVhZGVyVGV4dDs+O2w85Yib5paw5YaF5a65Oz4+Ozs7Oz47QDA8cDxsPEhlYWRlclRleHQ7PjtsPOWIm+aWsOWtpuWIhjs+Pjs7Ozs+O0AwPHA8bDxIZWFkZXJUZXh0Oz47bDzliJvmlrDmrKHmlbA7Pj47Ozs7Pjs7Oz47Ozs7Ozs7Ozs+Ozs+O3Q8cDxwPGw8VGV4dDtWaXNpYmxlOz47bDzmnKzkuJPkuJrlhbE4N+S6ujtvPGY+Oz4+Oz47Oz47dDxwPHA8bDxWaXNpYmxlOz47bDxvPGY+Oz4+Oz47Oz47dDxwPHA8bDxWaXNpYmxlOz47bDxvPGY+Oz4+Oz47Oz47dDxwPHA8bDxWaXNpYmxlOz47bDxvPGY+Oz4+Oz47Oz47dDxwPHA8bDxUZXh0Oz47bDxKWVU7Pj47Pjs7Pjt0PHA8cDxsPEltYWdlVXJsOz47bDwuL2V4Y2VsLzEzMTExMDE0Mi5qcGc7Pj47Pjs7Pjs+Pjt0PDtsPGk8Mz47PjtsPHQ8QDA8Ozs7Ozs7Ozs7Oz47Oz47Pj47Pj47Pj47PlsDeYWhU3K64A48vZ2GUe5jfjlZ")
                        .add("hidLanguage", "")
                        .add("ddlXN", "2014-2015")
                        .add("ddlXQ", "1")
                        .add("ddl_kcxz", "")
                        .add("btn_xq", URLEncoder.encode("历年成绩","gb2312")).build();
        final Request request = new Request.Builder().url(loginScoreUrl)
                .addHeader("Referer", loginScoreUrl)
                .addHeader("Host", "210.38.162.117")
                .post(requestBody).build();
        okHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Request request, IOException e) {

            }

            @Override
            public void onResponse(Response response) throws IOException {
               
                BufferedReader br = new BufferedReader(new InputStreamReader(response.body().byteStream(), "gb2312"));
                StringBuilder sb=new StringBuilder();
                String line;
                while ((line = br.readLine()) != null) {
                    sb.append(line);
                }
				parseHtml(sb.toString());
            }
        });
    }

在完成查询后，我们从response中获得整个html页面，然后通过jsoup解析它，获得我们要的成绩。

	
    private List<Course> parseHtml(String html) {
        List<Course> courses=new ArrayList<>();
        Document document=Jsoup.parse(html);
        //得到存放成绩的表格
        Element element=document.getElementsByClass("datelist").get(0);
        //得到所有的行
        Elements trs=element.getElementsByTag("tr");
        for (int i=1;i<trs.size();i++){
            //第一行是列名这里不需要
            Element e=trs.get(i);
            //得到一行中的所有列
            Elements tds=e.getElementsByTag("td");
            String cYear=tds.get(0).text();
            String cTerm=tds.get(1).text();
            String cId=tds.get(2).text();
            String cName=tds.get(3).text();
            String cState=tds.get(4).text();
            String credit=tds.get(6).text();
            String gpa=tds.get(7).text();
            String score=tds.get(8).text();
            String college=tds.get(12).text();
            Course course=new Course(cId,cName,college,credit,cState,cTerm,cYear,gpa,score);
            courses.add(course);
        }
        return courses;
    }

	public class Course implements Serializable{
	    /**
	     * 课程id
	     */
	    private String cId;
	    /**
	     * 课程名
	     */
	    private String cName;
	    /**
	     * 课程属性
	     */
	    private String cState;
	    /**
	     * 学分
	     */
	    private String credit;
	    /**
	     * 绩点
	     */
	    private String gpa;
	    /**
	     * 分数
	     */
	    private String score;
	    /**
	     * 学年
	     */
	    private String cYear;
	    /**
	     * 学期
	     */
	    private String cTerm;
	    /**
	     * 开课学院
	     */
	    private String college;
	
	
	    public Course() {
	    }
	
	    public Course(String cId, String cName, String college, String credit, String cState, String cTerm, String cYear, String gpa, String score) {
	        this.cId = cId;
	        this.cName = cName;
	        this.college = college;
	        this.credit = credit;
	        this.cState = cState;
	        this.cTerm = cTerm;
	        this.cYear = cYear;
	        this.gpa = gpa;
	        this.score = score;
	    }
	
	    public String getcId() {
	        return cId;
	    }
	
	    public void setcId(String cId) {
	        this.cId = cId;
	    }
	
	    public String getcName() {
	        return cName;
	    }
	
	    public void setcName(String cName) {
	        this.cName = cName;
	    }
	
	    public String getCollege() {
	        return college;
	    }
	
	    public void setCollege(String college) {
	        this.college = college;
	    }
	
	    public String getCredit() {
	        return credit;
	    }
	
	    public void setCredit(String credit) {
	        this.credit = credit;
	    }
	
	    public String getcState() {
	        return cState;
	    }
	
	    public void setcState(String cState) {
	        this.cState = cState;
	    }
	
	    public String getcTerm() {
	        return cTerm;
	    }
	
	    public void setcTerm(String cTerm) {
	        this.cTerm = cTerm;
	    }
	
	    public String getcYear() {
	        return cYear;
	    }
	
	    public void setcYear(String cYear) {
	        this.cYear = cYear;
	    }
	
	    public String getGpa() {
	        return gpa;
	    }
	
	    public void setGpa(String gpa) {
	        this.gpa = gpa;
	    }
	
	    public String getScore() {
	        return score;
	    }
	
	    public void setScore(String score) {
	        this.score = score;
	    }
	}

至此，就解析到的数据传入了一个List集合，然后把它弄成什么样子显示在Android客户端上就由你自己决定了。

![](http://i.imgur.com/jFpQ0aj.png)
![](http://i.imgur.com/zEUYDA6.png)

附上自己写好的一个Demo，哈哈哈。
