### 一、钢厂调价

#### 1.调价

* 点击"添加并生成文本"按钮时执行的sql：
```
string sql = 
"insert into adjustprice
(
    steelname,steeltype,steelparam,quality,price,weight,
    remark,range,adjust,adjusttime,addtime,adjusttype,
    addtax,includetax,address,temperature,tolerance
) 
values
(
    @steelname,@steeltype,@steelparam,@quality,@price,
    @weight,@remark,@range,@adjust,@adjusttime,@addtime,
    @adjusttype,@addtax,@includetax,@address,@temperature,@tolerance
)";

SqlParameter[] param = new SqlParameter[] { 
    new SqlParameter("@steelname",adj.SteelName),//钢厂
    new SqlParameter("@steeltype",adj.SteelType),//调价情况：品种
    new SqlParameter("@steelparam",adj.SteelParam),//调价情况：调价规格
    new SqlParameter("@quality",adj.Quality),//调价情况：材质
    new SqlParameter("@price",adj.Price),//调价情况：出厂价格
    new SqlParameter("@weight",adj.Weight),//此外写死，为空""
    new SqlParameter("@remark",adj.Remark),//调价情况：备注
    new SqlParameter("@range",adj.Range),//调价情况：调整幅度
    new SqlParameter("@adjust",adj.Adjust),//此处写死，为0
    new SqlParameter("@adjusttime",adj.AdjustTime),//调价时间
    new SqlParameter("@addtime",adj.AddTime),//当前时间
    new SqlParameter("@adjusttype",adj.AdjustType),//前台设置为0
    new SqlParameter("@addtax",adj.AddTax),//默认为空
    new SqlParameter("@includetax",adj.IncludeTax),//默认为空
    new SqlParameter("@address",adj.Address),//默认为空
    new SqlParameter("@temperature",adj.Temperature),//默认为空
    new SqlParameter("@tolerance",adj.Tolerance)//默认为空
};
return SqlHelper.ExecuteNonQuery(ConnString.conn, CommandType.Text, sql, param);
```
* 调价品种。点击添加时，执行的sql内容：
```
string sql = 
"insert into steeltypes
(
    typename,showorder,status,pid,itemid
) 
values
(
@typename,@showorder,@status,@pid,@itemid
);select @@identity; ";

SqlParameter[] param = new SqlParameter[] { 
    new SqlParameter("@typename",st.TypeName),//此二级品种名
    new SqlParameter("@showorder",st.ShowOrder),//排序，此外为0
    new SqlParameter("@status",st.Status),//状态，此处为0
    new SqlParameter("@pid",st.Pid),//父品种id，即界面中的下拉框。
    new SqlParameter("@itemid",st.ItemID)//为空
};
return Convert.ToInt32(SqlHelper.ExecuteScalar(ConnString.conn, CommandType.Text, sql, param));
```
* 点击“调价品种”时会根据“钢厂、调价品种和调价时间”，查询数据库。    
* 其中“钢厂和调价品种”是在界面中通过Dom操作拿到，调价时间是后台两次查询数据库。
```
string sql = 
"select 
    id,steelname,steeltype,adjusttype,steelparam,
    quality,price,weight,remark,range,adjust,adjusttime,
    addtime 
from adjustprice 
where 
    steelname=@steelname  
and 
    adjusttype=@adjusttype 
and 
    adjusttime=@adjusttime ";

```
* 拿到数据后，后台进行循环拼接字符串操作，后在前台显示。
*注：此写法会消耗大量后台资源，极不建议采用！*
```
if (list.Count > 0)
{
    for (int i = 0; i < list.Count; i++)
    {
        sb.Append("<tr align='center' id='trAdjust" + (i + 1) + "'>");
        sb.Append("<td>" + DropDown(typeArr, "" + list[i].SteelType + "") + "</td>");
        sb.Append("<td><input type='text' value='" + list[i].Quality + "'/></td>");
        sb.Append("<td><input type='text' value='" + list[i].SteelParam + "'/></td>");
        sb.Append("<td><input type='text' value='" + list[i].Price + "' style='width:55px'  /></td>");
        sb.Append("<td><input type='text'  value='" + list[i].Adjust + "' style='width:55px'  /></td>");
        sb.Append("<td><input type='text' value='" + list[i].Remark + "'/></td>");
        sb.Append("<td><a href='#' onclick='removeAdjust(" + (i + 1) + ")'>-</a></td>");
        sb.Append("</tr>");
    }
}
else
{
    sb.Append("<tr align='center' id='trAdjust1'>");
    sb.Append("<td>" + DropDown(typeArr, "") + "</td>");
    sb.Append("<td><input type='text' /></td>");
    sb.Append("<td><input type='text' /></td>");
    sb.Append("<td><input type='text'  style='width:55px'  /></td>");
    sb.Append("<td><input type='text'   style='width:55px'  /></td>");
    sb.Append("<td><input type='text' /></td>");
    sb.Append("<td></td>");
    sb.Append("</tr>");
}
```

#### 2.调价列表
* 页面加载时查询数据库SQL语句：
*注：查询数据库时，只传入了一个参数：pid=0*

```
List<SteelType> list = new List<SteelType>();
string sql = "
select 
    id,typename,showorder,status,pid,itemid 
from steeltypes 
where 
    pid=@pid //默认写为：0
order by id ";
//以下代码是拿到数据后的赋值操作
using (SqlDataReader reader = SqlHelper.ExecuteReader(ConnString.conn, CommandType.Text, sql, new SqlParameter("@pid", pid)))
{
    while (reader.Read())
    {
        SteelType st = new SteelType();
        st.TypeName = reader["typename"].ToString();
        st.ShowOrder = Convert.ToInt32(reader["showorder"]);
        st.Status = Convert.ToInt32(reader["status"]);
        st.Pid = Convert.ToInt32(reader["pid"]);
        st.ItemID = reader["itemid"].ToString();
        st.ID = Convert.ToInt32(reader["id"]);
        list.Add(st);
    }
}
return list;
```

#### 3.调价新闻

* 页面加载时查询数据库，是通过存储过程，传入的参数为：itemid
页面加载时表现层调用代码：
```
DataSet ds = NewsCenter.BLL.News.GetAllNewsByPage(itemid, pagesize, page, ref count);//后三个参数，都为分页。
```
调用存储过程代码：
```
ds = NewsCenter.DBUtility.SqlHelper.ExecuteDataSet(ConnString.connReadonly, CommandType.StoredProcedure, "[dbo].[list_page]", parm);
```

#### 4.钢厂名称

* 钢厂名称页面显示时，调用的数据如下：

id | steelname | showorder | status
---|---|---|---
ID | 钢厂名 | 排序 | 状态

#### 5.品种类别

* 品种类别页面加载时，调用的数据如下：

id | typename | showorder | status | pid | itemid 
---|---|---|---|---|---
ID | 钢厂名 | 排序 | 状态 | 父ID | 项目ID

#### 6.数据导入

* 点击“导入数据”时，插入到表中的数据：

steelname|steeltype|steelparam|quality|price|weight|remark|range|adjust|adjusttime|addtime|adjusttype|addtax|includetax|address|temperature|tolerance
---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---


#### 7.调价
* 根据“钢厂(steelname)、品种(steeltype)、开始日期(adjusttime)、结束日期(endTime)、AdjustType(默认值为0)”，调取数据库数据。
代码如下：
```
return AdjustPriceDAL.GetAdjustPriceDraw(steelname, steeltype, adjusttime, endTime, adjustType);
```
* DAL层SQL语句
```
string sql = "
select 
    id,steelname,steeltype,adjusttype,steelparam,quality,price,
    weight,remark,range,adjust,adjusttime,addtime,address,temperature,
    tolerance,addtax,includetax 
from adjustprice 
where 
adjusttype=@adjusttype and 1=1 ";

//此处为处理开始时间和结束时间
param.Add(new SqlParameter("@adjusttype", adjustType));
if (!string.IsNullOrEmpty(adjusttime))
{
    sql += " and adjusttime>=@adjusttime ";
    param.Add(new SqlParameter("@adjusttime", adjusttime));
}
if (!string.IsNullOrEmpty(endTime))
{
    sql += "  and  adjusttime<=@endtime";
    param.Add(new SqlParameter("@endtime", endTime));
}
```
查询数据库，返回的数据有：
```
IList<AdjustPrice> list = new List<AdjustPrice>();
using (SqlDataReader reader = SqlHelper.ExecuteReader(ConnString.conn, CommandType.Text, sql, param.ToArray()))
{
    while (reader.Read())
    {        
        AdjustPrice ad = new AdjustPrice();
        ad.AddTime = reader["addtime"].ToString();
        ad.Adjust = Convert.ToInt32(reader["adjust"]);
        ad.AdjustTime = reader["adjusttime"].ToString();
        ad.ID = Convert.ToInt32(reader["id"]);
        ad.Price = Convert.ToString(reader["price"]);
        ad.Quality = Convert.ToString(reader["quality"]);
        ad.Range = Convert.ToString(reader["range"]);
        ad.Remark = reader["remark"].ToString();
        ad.SteelName = reader["steelname"].ToString();
        ad.SteelParam = reader["steelparam"].ToString();
        ad.SteelType = reader["steeltype"].ToString();
        ad.AdjustType = Convert.ToInt32(reader["adjusttype"]);
        ad.Weight = Convert.ToString(reader["weight"]);
        ad.Tolerance = reader["tolerance"].ToString();
        ad.Address = reader["address"].ToString();
        ad.Temperature = Convert.ToString(reader["temperature"]);
        ad.AddTax = Convert.ToString(reader["addtax"]);
        ad.IncludeTax = Convert.ToString(reader["includetax"]);

        list.Add(ad);
        
    }
}
return list;
```