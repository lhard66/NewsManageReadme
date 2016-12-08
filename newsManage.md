### 一、钢厂调价

#### 1.调价

点击"添加并生成文本"按钮时执行的sql：
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
    new SqlParameter("@addtime",adj.AddTime),//当时时间
    new SqlParameter("@adjusttype",adj.AdjustType),//前台设置为0
    new SqlParameter("@addtax",adj.AddTax),//默认为空
    new SqlParameter("@includetax",adj.IncludeTax),//默认为空
    new SqlParameter("@address",adj.Address),//默认为空
    new SqlParameter("@temperature",adj.Temperature),//默认为空
    new SqlParameter("@tolerance",adj.Tolerance)//默认为空
};
return SqlHelper.ExecuteNonQuery(ConnString.conn, CommandType.Text, sql, param);
```
高价品种。点击添加时，执行的sql内容：
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