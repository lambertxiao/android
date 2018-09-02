#  #Android抓取正方系统课程——实现自己的课程表

上一篇博客讲解了如何使用http协议模拟登陆正方系统，今天继续实现如何抓取课程表并显示在Android界面上，效果如图：
![](http://i.imgur.com/CkjbIpw.png)

由于偷懒，在界面上没下太多功夫，看得过去就行了哈哈哈。

在我们第一次点击 **专业推荐课表查询** 的时候，浏览器干了这么件事。

	Request URL:http://210.38.162.117/(4nqfqf55wv4suyi0aewkdgiy)/tjkbcx.aspx?xh=131110142&xm=%D0%A4%C9%D9%D0%C7&gnmkdm=N121601
	Request Method:GET
	Host:210.38.162.117
	Referer:http://210.38.162.117/(4nqfqf55wv4suyi0aewkdgiy)/xs_main.aspx?xh=131110142

它向服务器发起了一次get，请求的路径是

	http://210.38.162.117/(4nqfqf55wv4suyi0aewkdgiy)/tjkbcx.aspx?xh=131110142&xm=%D0%A4%C9%D9%D0%C7&gnmkdm=N121601

其实和查成绩的路径很类似，
xh是学号,xm是Url编码过的你的名字，gnmkdm在查询课表的时候为N121601。
	
	/**
     * 专业课表查询的url
     */
    String queryTimeTableUrl = "http://210.38.162.117/(sxaiuuqeeivmbu555ha1bd55)/tjkbcx.aspx?xh={xh}&xm={name}&gnmkdm=N121605";

	String REFERER="http://210.38.162.117/(sxaiuuqeeivmbu555ha1bd55)/xs_main.aspx?xh=131110142";
	
	String HOST="210.38.162.117";
	/**
     * 第一次查询的时候只需访问链接
     */
	public void queryTimeTableFirstTime() throws UnsupportedEncodingException {
        queryTimeTableUrl = queryTimeTableUrl.replace("{xh}", account).replace("{name}", URLEncoder.encode(name, "gb2312"));
        final Request request = new Request.Builder()
                .url(queryTimeTableUrl)
				//必须带上登陆成功是返回的链接和Host
                .addHeader("Referer", REFERER)
                .addHeader("Host", HOST)
                .build();
        okHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Request request, IOException e) {
                showToast("课表查询失败");
            }

            @Override
            public void onResponse(Response response) throws IOException {
                if (response.isSuccessful()) {
                    BufferedReader br = new BufferedReader(new InputStreamReader(response.body().byteStream(), "gb2312"));
                    StringBuilder sb = new StringBuilder();
                    String line;
                    while ((line = br.readLine()) != null) {
                        sb.append(line);
                    }
                    parseTimeTableHtml(sb.toString());
                }
            }
        });
    }

之后要做的是就是解析html页面了，我简单抽取了部分html代码，如下：

这一部分保存了功课表的内容

	<table id="Table6" class="blacktab" rules="all" border="1" height="132" width="100%">
		<tr>
			<td colspan="2" rowspan="1" width="2%">时间</td>
			<td align="Center" width="14%">星期一</td>
			<td align="Center" width="14%">星期二</td>
			<td align="Center" width="14%">星期三</td>
			<td align="Center" width="14%">星期四</td>
			<td align="Center" width="14%">星期五</td>
			<td align="Center" width="14%">星期六</td>
			<td align="Center" width="14%">星期日</td>
		</tr>
		<tr>
			<td colspan="2">早晨</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
		</tr>
		<tr>
			<td rowspan="4" width="1%">上午</td>
			<td width="1%">第1节</td>
			<td align="Center" rowspan="2" width="7%">计算机组成原理<br>2节/周(1-17)<br>张凤英<br>田师208<br></td>
			<td align="Center" rowspan="2" width="7%">汇编语言程序设计<br>2节/周(1-17)<br>陈生庆<br>田师211<br></td>
			<td align="Center" width="7%">&nbsp;</td>
			<td align="Center" rowspan="2" width="7%">教育学<br>2节/周(1-16)<br>刘奕涛<br>田师208<br></td>
			<td align="Center" rowspan="2" width="7%">汇编语言程序设计<br>2节/双周(1-17)<br>陈生庆<br>锡科407<br><br><br>软件工程<br>2节/单周(1-17)<br>陈世基<br>锡科405<br></td>
			<td align="Center" width="7%">&nbsp;</td>
			<td align="Center" width="7%">&nbsp;</td>
		</tr>
		<tr>
			<td>第2节</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
		</tr>
		<tr>
			<td>第3节</td>
			<td align="Center" rowspan="2">汇编语言程序设计<br>2节/单周(1-17)<br>陈生庆<br>田师211<br></td>
			<td align="Center" rowspan="2">数据库系统原理<br>2节/单周(1-17)<br>张海峰<br>田师201<br></td>
			<td align="Center" rowspan="2">计算机组成原理<br>2节/周(1-17)<br>张凤英<br>田师208<br></td>
			<td align="Center" rowspan="2">软件工程<br>2节/单周(1-17)<br>陈世基<br>锡科505<br></td>
			<td align="Center" rowspan="2">网络基础<br>2节/双周(1-17)<br>吴华光<br>锡科407<br></td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
		</tr>
		<tr>
			<td>第4节</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
		</tr>
		<tr>
			<td rowspan="4" width="1%">下午</td>
			<td>第5节</td>
			<td align="Center" rowspan="2">数据库系统原理<br>2节/双周(1-17)<br>张海峰<br>锡科405<br></td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center" rowspan="2">数据库系统原理<br>2节/周(1-17)<br>张海峰<br>田师208<br></td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
		</tr>
		<tr>
			<td>第6节</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
		</tr>
		<tr>
			<td>第7节</td>
			<td align="Center" rowspan="2">数据结构课程设计<br>2节/周(1-17)<br>巫喜红<br>锡科405<br></td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
		</tr>
		<tr>
			<td>第8节</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
		</tr>
		<tr>
			<td rowspan="3" width="1%">晚上</td>
			<td>第9节</td>
			<td align="Center" rowspan="2">软件工程<br>2节/周(1-17)<br>陈世基<br>田师208<br></td>
			<td align="Center">&nbsp;</td><td align="Center" rowspan="3">网络基础<br>3节/周(1-17)<br>吴华光<br>田师209<br></td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td><td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
		</tr>
		<tr>
			<td>第10节</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
		</tr>
		<tr>
			<td>第11节</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
			<td align="Center">&nbsp;</td>
		</tr>
	</table>

下面的html代码包含了学年（xn）、学期（xq）、年级（nj）、专业（zy）、学院（xy）、推荐课表（kb）

	<TABLE id="Table1" cellSpacing="0" cellPadding="3" width="100%">
		<TR>
			<TD>学&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 年:
				<select name="xn" onchange="__doPostBack('xn','')" language="javascript" id="xn">
					<option selected="selected" value="2015-2016">2015-2016</option>
					<option value="2014-2015">2014-2015</option>
					<option value="2013-2014">2013-2014</option>
					<option value="2012-2013">2012-2013</option>
					<option value="2011-2012">2011-2012</option>
					<option value="2010-2011">2010-2011</option>
					<option value="2009-2010">2009-2010</option>
					<option value="2008-2009">2008-2009</option>
					<option value="2007-2008">2007-2008</option>
					</select></TD>
				<TD>学期:
					<select name="xq" onchange="__doPostBack('xq','')" language="javascript" id="xq">
						<option selected="selected" value="1">1</option>
						<option value="2">2</option>
	
					</select>
				</TD>
				<TD>年&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 级:
					<select name="nj" onchange="__doPostBack('nj','')" language="javascript" id="nj">
						<option value="2015">2015</option>
						<option value="2014">2014</option>
						<option selected="selected" value="2013">2013</option>
						<option value="2012">2012</option>
						<option value="2011">2011</option>
						<option value="2010">2010</option>
						<option value="2009">2009</option>
						<option value="2008">2008</option>
						<option value="2007">2007</option>
						<option value="2006">2006</option>
						<option value="2005">2005</option>
						<option value="2004">2004</option>
	
					</select></TD>
				</TR>
				<TR>
					<TD>学院名称:
						<select name="xy" onchange="__doPostBack('xy','')" language="javascript" id="xy">
							<option value="01">数学学院</option>
							<option value="02">物理与光信息科技学院</option>
							<option value="03">化学与环境学院</option>
							<option value="04">文学院</option>
							<option value="05">外国语学院</option>
							<option value="06">生命科学学院</option>
							<option value="07">政法学院</option>
							<option value="08">地理科学与旅游学院</option>
							<option value="09">经济与管理学院</option>
							<option value="10">电子信息工程学院</option>
							<option selected="selected" value="11">计算机学院</option>
							<option value="12">土木工程学院</option>
							<option value="13">美术学院</option>
							<option value="14">体育学院</option>
							<option value="15">音乐学院</option>
							<option value="16">教育科学学院</option>
							<option value="17">梅师分院</option>
							<option value="18">医学院</option>
	
						</select></TD>
						<TD>专业:
							<select name="zy" onchange="__doPostBack('zy','')" language="javascript" id="zy">
								<option value="1111">计算机科学与技术（师范）</option>
								<option value="1121">计算机科学与技术</option>
								<option value="1122">计算机科学与技术（软件方向）</option>
								<option value="1123">计算机科学与技术（网络方向）</option>
								<option selected="selected" value="1124">软件工程</option>
								<option value="1125">网络工程</option>
								<option value="1126">软件工程(服务外包)</option>
								<option value="1127">物联网工程</option>
								<option value="1131">计算机应用技术（师范）</option>
								<option value="1141">计算机及应用</option>
								<option value="1142">计算机应用技术</option>
								<option value=""></option>
	
							</select></TD>
							<TD>推荐课表:
								<select name="kb" onchange="__doPostBack('kb','')" language="javascript" id="kb">
									<option value=""></option>
									<option value="201311242015-2016121311241">计算机1303班</option>
									<option selected="selected" value="201311242015-2016121311242">计算机1304班</option>
	
								</select>
							</TD>
						</TR>
				</TABLE>


借助jsoup解析

	public static List<TimeTableBean> parseTimeTableHtml(String html) {
        List<TimeTableBean> timeTableBeens = new ArrayList<>();
        //先解析
        Document document = Jsoup.parse(html);
        //先解析出年级、学院、专业、班级代号
        getParm(document);
        //再解析__VIEWSTATE
        Element viewstateTag = document.getElementsByTag("input").get(2);
        __VIEWSTATE = viewstateTag.val();
        //---------------------------------
        Element element = document.getElementById("Table6");
        //得到所有的行
        Elements trs = element.getElementsByTag("tr");
        for (int i = 0; i < trs.size(); i++) {
            //只需要处理行号为2（第一节）、4（第三节课）、6（第五节课）、8（第七节课）、10（第九节课）
            if (i == 2 || i == 4 || i == 6 || i == 8 || i == 10) {
                //得到显示第一节课的那一行
                Element e1 = trs.get(i);
                //得到所有的列
                Elements tds = e1.getElementsByTag("td");
                if (i==2||i==6||i==10){
                    //先清除显示上午、下午、晚上的一行
                    tds.remove(0);
                }
                tds.remove(0);
                for (int j = 0; j < tds.size(); j++) {
                    String msg = tds.get(j).text();
                    //判断是否有课程
                    if (msg.length() == 1) {
                        //没有课程就跳过
                        Log.i("xsx", "msg为空跳过");
                        continue;
                    }
                    //处理一个td里的字符，如：计算机组成原理 2节/周(1-17) 张凤英 田师208
                    String strings[] = msg.split(" ");
                    //多少节
                    int duringNum = Integer.parseInt(strings[1].substring(0, 1));
                    int startNum = i-1;
                    int endNum = startNum + duringNum - 1;
                    String day = "";
                    switch (j) {
                        case 0:
                            day = "星期一";
                            break;
                        case 1:
                            day = "星期二";
                            break;
                        case 2:
                            day = "星期三";
                            break;
                        case 3:
                            day = "星期四";
                            break;
                        case 4:
                            day = "星期五";
                            break;
                        case 5:
                            day = "星期六";
                            break;
                        case 6:
                            day = "星期天";
                            break;
                    }
                    timeTableBeens.add(new TimeTableBean(strings[0], startNum, endNum, day, strings[3]));
                }
            }
        }
        return timeTableBeens;
    }
	/**
     * 是否是第一次查询课表
     */
	static boolean isFirstTime = true;
	/**
     * 第一次查询课表的时候需要查询
     */
    static String nj;
    static String xy;
    static String zy;
    static String kb;
    /**
     * 每次都有查询出来，为下次查询做准备
     */
    static String __VIEWSTATE;
    

    /**
     * 得到年级、学院、专业、班级代号等参数
     *
     * @param document
     */
    private static void getParm(Document document) {
        if (isFirstTime) {
            Elements elements1 = document.getElementById("nj").getAllElements();
            for (Element e : elements1) {
                Element element = e.getElementsByAttribute("selected").get(0);
                if (element != null) {
                    nj = element.val();
                    break;
                }
            }
            Elements elements2 = document.getElementById("xy").getAllElements();
            for (Element e : elements2) {
                Element element = e.getElementsByAttribute("selected").get(0);
                if (element != null) {
                    xy = element.val();
                    break;
                }
            }
            Elements elements3 = document.getElementById("zy").getAllElements();
            for (Element e : elements3) {
                Element element = e.getElementsByAttribute("selected").get(0);
                if (element != null) {
                    zy = element.val();
                    break;
                }
            }
            Elements elements4 = document.getElementById("kb").getAllElements();
            for (Element e : elements4) {
                Element element = e.getElementsByAttribute("selected").get(0);
                if (element != null) {
                    kb = element.val();
                    break;
                }
            }
            isFirstTime = false;
            Log.i("xsx", "nj=" + nj + ",xy=" + xy + ",zy=" + zy + ",kb=" + kb);
            Log.i("xsx", "第一次查询");
        }
    }

接下来，实现查询指定学年学期的课程表，这里需要向服务器发送post的请求，请求的路径还是之前那一条，另外要带上一些参数。

这里__VIEWSTATE这个参数是一个随机产生的字符串，请求的时候需要带上，请求成功后会生成新的字符串，所以每次请求后我们都要将它保存，为下一次请求做准备。


	/**
     * 指定查询某一学期的课程表
     * @param date
     * @throws UnsupportedEncodingException
     */
    public void queryTimeTable(String date) throws UnsupportedEncodingException {
        String url = "http://210.38.162.117/(sxaiuuqeeivmbu555ha1bd55)/tjkbcx.aspx?xh={xh}&xm={name}&gnmkdm=N121601";
        String xn = date.substring(0, 9);
        String xq = date.substring(11, 12);
        url = url.replace("{xh}", account).replace("{name}", URLEncoder.encode(name, "gb2312"));
        FormEncodingBuilder builder = new FormEncodingBuilder();

        RequestBody requestBody = builder
                .add("__EVENTTARGET", "xq")
                .add("__EVENTARGUMENT", "")
                .add("__VIEWSTATE", __VIEWSTATE)
                .add("xn", xn)  //学年
                .add("xq", xq)  //学期
                .add("nj", nj)  //年级
                .add("xy", xy)  //学院
                .add("zy", zy)  //专业代号
                .add("kb", kb) //班级代号
                .build();

        final Request request = new Request.Builder().url(url)
                .addHeader("Referer", url)
                .addHeader("Host", HOST)
                .post(requestBody)
                .build();


        okHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Request request, IOException e) {

            }

            @Override
            public void onResponse(Response response) throws IOException {
                log("onResponse");
                if (response.isSuccessful()) {
                    BufferedReader br = new BufferedReader(new InputStreamReader(response.body().byteStream(), "gb2312"));
                    StringBuilder sb = new StringBuilder();
                    String line;
                    while ((line = br.readLine()) != null) {
                        sb.append(line);
                    }
                    parseTimeTableHtml(sb.toString());
                }
            }
        });


    }



首先定义实体类
	
	/**
	 * 课程表中一节课
	 * Created by XSX on 2016/2/21.
	 */
	public class TimeTableBean implements Serializable{
	    /**
	     * 课程名
	     */
	    private String cName;
	    /**第几节开始
	     *
	     */
	    private int startNum;
	    /**
	     * 第几节结束
	     */
	    private int endNum;
	    /**
	     * 星期几
	     */
	    private String day;
	    /**
	     * 哪间教室
	     */
	    private String classroom;
	
	    public TimeTableBean() {
	    }
	
	    public TimeTableBean(String cName, int startNum, int endNum, String day, String classroom) {
	        this.cName = cName;
	        this.startNum = startNum;
	        this.endNum = endNum;
	        this.day = day;
	        this.classroom = classroom;
	    }
	
	
	    public String getcName() {
	        return cName;
	    }
	
	    public void setcName(String cName) {
	        this.cName = cName;
	    }
	
	    public int getStartNum() {
	        return startNum;
	    }
	
	    public void setStartNum(int startNum) {
	        this.startNum = startNum;
	    }
	
	    public int getEndNum() {
	        return endNum;
	    }
	
	    public void setEndNum(int endNum) {
	        this.endNum = endNum;
	    }
	
	    public String getDay() {
	        return day;
	    }
	
	    public void setDay(String day) {
	        this.day = day;
	    }
	
	    public String getClassroom() {
	        return classroom;
	    }
	
	    public void setClassroom(String classroom) {
	        this.classroom = classroom;
	    }
	}

视图方面，简单说下处理的思路，每一节课都是一个TextView，然后课程表的布局是一个RelativeLayout，通过是星期几的课可以确定左边距，通过确定第几节课开始可以确定上边距，最后根据课程的长度确定TextView的高度。

	/**
	 * 课程表视图
	 * Created by XSX on 2016/2/21.
	 */
	public class TimeTableView extends LinearLayout {
	
	    /**
	     * 显示周几的LinearLayout
	     */
	    private LinearLayout dayLinearLayout;
	    /**
	     * 下方内容的LinearLayout
	     */
	    private LinearLayout contentLinearLayout;
	    /**
	     * 下方左边显示节数的LinearLayout
	     */
	    private LinearLayout numberLinearLayout;
	    /**
	     * 课程表格的LinearLayout
	     */
	    private RelativeLayout tableLinearLayout;
	    /**
	     * 一个格子的宽度
	     */
	    private static final int GRID_WIDTH = 60;
	    /**
	     * 一个格子的高度
	     */
	    private static final int GRID_HEIGHT = 70;
	    /**
	     * 侧边宽度
	     */
	    private static final int SIDE_WIDTH = 30;
	    /**
	     * 侧边高度
	     */
	    private static final int SIDE_HEIGHT = 40;
	    /**
	     * 一周的课程
	     */
	    private List<Course> courses;
	    /**
	     * 一周的天数+一个用来显示几月份的格子，总共8个格子
	     */
	    private static final int DAY_COUNT = 7;
	    /**
	     * 一天最多11节课
	     */
	    private static final int COURSE_COUNT = 11;
	
	
	    private static final int[] bg=new int[]{
	            R.drawable.course_bg1,
	            R.drawable.course_bg2,
	            R.drawable.course_bg3,
	            R.drawable.course_bg4,
	            R.drawable.course_bg5,
	            R.drawable.course_bg6,
	            R.drawable.course_bg7,
	            R.drawable.course_bg8
	    };
	
	    private static final int textColor=Color.parseColor("#ffffff");
	    private static final int otherTextColor=Color.parseColor("#8ec6f4");
	
	    public TimeTableView(Context context) {
	        this(context, null);
	    }
	
	    public TimeTableView(Context context, AttributeSet attrs) {
	        this(context, attrs, 0);
	    }
	
	    public TimeTableView(Context context, AttributeSet attrs, int defStyleAttr) {
	        super(context, attrs, defStyleAttr);
	    }
	
	    /**
	     * 外界如果没有设置课程的话界面是空的
	     *
	     * @param courses
	     */
	    public void setCourses(List<Course> courses) {
	        this.courses = courses;
	        initTimeTable();
	    }
	
	    /**
	     * 初始化课程表
	     */
	    private void initTimeTable() {
	        //-----------------初始化最上方一栏-----------------------
	        dayLinearLayout = new LinearLayout(getContext());
	        //设置宽高
	        dayLinearLayout.setLayoutParams(new LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, dp2px(SIDE_HEIGHT)));
	        //设置方向
	        dayLinearLayout.setOrientation(HORIZONTAL);
	        //添加显示周几的textView
	        for (int i = 0; i <= DAY_COUNT; i++) {
	            if (i == 0) {
	                //先添加一个显示当前月数的TextView
	                TextView tv0 = new TextView(getContext());
	                //设置宽高
	                tv0.setWidth(dp2px(SIDE_WIDTH));
	                tv0.setHeight(dp2px(SIDE_HEIGHT));
	                tv0.setText("2月");
	                tv0.setTextColor(otherTextColor);
	                tv0.setBackgroundResource(R.drawable.tv_border);
	                tv0.setGravity(Gravity.CENTER);
	                dayLinearLayout.addView(tv0);
	            }
	            //添加显示周几的textView
	            else {
	                TextView tv = new TextView(getContext());
	                tv.setWidth(dp2px(GRID_WIDTH));
	                tv.setHeight(dp2px(SIDE_HEIGHT));
	                tv.setText("星期" + i);
	                tv.setTextColor(otherTextColor);
	                tv.setBackgroundResource(R.drawable.tv_border);
	                tv.setGravity(Gravity.CENTER);
	                dayLinearLayout.addView(tv);
	            }
	        }
	        this.addView(dayLinearLayout);
	
	        //-------------------初始化左边一栏-------------------
	        contentLinearLayout = new LinearLayout(getContext());
	        contentLinearLayout.setLayoutParams(new LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT));
	        contentLinearLayout.setOrientation(HORIZONTAL);
	
	        numberLinearLayout = new LinearLayout(getContext());
	        numberLinearLayout.setLayoutParams(new LayoutParams(dp2px(SIDE_WIDTH), ViewGroup.LayoutParams.WRAP_CONTENT));
	        numberLinearLayout.setOrientation(VERTICAL);
	
	        //添加显示节数的TextView
	        for (int i = 1; i <= COURSE_COUNT; i++) {
	            TextView tv = new TextView(getContext());
	            tv.setWidth(dp2px(SIDE_WIDTH));
	            tv.setHeight(dp2px(GRID_HEIGHT));
	            tv.setText(i + "");
	            tv.setTextColor(otherTextColor);
	            tv.setGravity(Gravity.CENTER);
	            tv.setBackgroundResource(R.drawable.tv_border);
	            numberLinearLayout.addView(tv);
	        }
	        contentLinearLayout.addView(numberLinearLayout);
	
	        //--------------初始化课程内容-----------------------
	        tableLinearLayout = new RelativeLayout(getContext());
	        tableLinearLayout.setLayoutParams(new LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT));
	        for (int i = 0; i < courses.size(); i++) {
	            Course course=courses.get(i);
	            TextView tv=new TextView(getContext());
	            tv.setWidth(dp2px(GRID_WIDTH));
	            //tv.setHeight(dp2px(GRID_HEIGHT));
	            tv.setGravity(Gravity.CENTER);
	            tv.setText(course.getcName()+"@"+course.getClassroom());
	            tv.setTextColor(textColor);
	            tv.setBackgroundResource(bg[new Random().nextInt(7)]);
	            //tv.setBackgroundResource(R.drawable.course_bg1);
	            //设置top和left
	            int left = 0;
	            String day=course.getDay();
	            //判断星期几决定left
	            switch (day){
	                case "星期一":
	                    left=0;
	                    break;
	                case "星期二":
	                    left=GRID_WIDTH;
	                    break;
	                case "星期三":
	                    left=GRID_WIDTH*2;
	                    break;
	                case "星期四":
	                    left=GRID_WIDTH*3;
	                    break;
	                case "星期五":
	                    left=GRID_WIDTH*4;
	                    break;
	                case "星期六":
	                    left=GRID_WIDTH*5;
	                    break;
	                case "星期天":
	                    left=GRID_WIDTH*6;
	                    break;
	            }
	            //tv.setLeft(dp2px(left));
	            //决定top
	            int startNum=course.getStartNum();
	            int endNum=course.getEndNum();
	            int top=0,height=0;
	            switch (startNum){
	                case 1:
	                    top=0;
	                    break;
	                case 3:
	                    top=GRID_HEIGHT*2;
	                    break;
	                case 5:
	                    top=GRID_HEIGHT*4;
	                    break;
	                case 7:
	                    top=GRID_HEIGHT*6;
	                    break;
	                case 9:
	                    top=GRID_HEIGHT*8;
	                    break;
	            }
	            //tv.setTop(dp2px(top));
	            //决定height
	            switch (endNum){
	                //有2节课，有3节课的
	                case 2:
	                case 4:
	                case 6:
	                case 8:
	                case 10:
	                    height=GRID_HEIGHT*2;
	                    break;
	                case 3:
	                case 7:
	                case 11:
	                    height=GRID_HEIGHT*3;
	                    break;
	            }
	            Log.i("xsx","left="+left+",top="+top+",height="+height);
	            RelativeLayout.LayoutParams lp = new RelativeLayout.LayoutParams(RelativeLayout.LayoutParams.WRAP_CONTENT, RelativeLayout.LayoutParams.WRAP_CONTENT);
	            //lp.setMargins(left, top, left+GRID_WIDTH, top+height);
	            lp.leftMargin=dp2px(left);
	            lp.topMargin=dp2px(top);
	//            lp.height=height;
	            tv.setLayoutParams(lp);
	            tv.setHeight(dp2px(height));
	            tableLinearLayout.addView(tv);
	        }
	        contentLinearLayout.addView(tableLinearLayout);
	        this.addView(contentLinearLayout);
	    }
	
	    @Override
	    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
	    }
	
	    private int dp2px(int dpValue) {
	        DisplayMetrics metrics = getContext().getResources().getDisplayMetrics();
	        float density = metrics.density;
	        return (int) (density * dpValue + 0.5f);
	    }
	}
