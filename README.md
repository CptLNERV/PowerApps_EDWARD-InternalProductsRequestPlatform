# PowerApps_EDWARD-InternalProductsRequestPlatform

需求：业务部门需要一个系统来自动化处理内部产品需求，这些产品将用在产品展示/内部使用 等等非销售用途。
需要自动话解决产品选择/请求发送/请求审批/请求追踪/产品交付/数据统计 等流程

得利于Azure工作环境，我们将使用Power Apps+Power Automate来搭建这一系统

前端方案:使用Power Apps搭建前端需求页面
# 1. Page Selection



![menu dropdown](https://github.com/CptLNERV/PowerApps_EDWARD-InternalProductsRequestPlatform/assets/20716430/416f1f53-9006-464f-ac25-fbb70c8fee73) 



```
ForAll(Distinct(ProductListForAPPS,type), {Result: ThisRecord.Value})
```
```
SortByColumns
(
    ForAll(Distinct(
        Filter(
            ProductListForAPPS,type = Dropdown_Type.Selected.Result
            )
            ,'More Specific series names'
            ), {Result: ThisRecord.Value})
    ,"Result",
    SortOrder.Descending
)
```



```
ForAll(Distinct(
    SortByColumns(
        Filter(
            ProductListForAPPS,
            'More Specific series names' = Dropdown_Series.Selected.Result,
            IsLDU in ProductName_OPPO
            ).ProductName_OPPO,"ProductName_OPPO",SortOrder.Descending),
            ProductName_OPPO
        ), {Result: ThisRecord.Value})
```

Add the selected result to the gallery
```
//Clear(colProductA);
Collect(
    colProductA,
    {
        // request number逻辑 mr+ 年月日 yyyyMMdd + 申请人姓首字母+ 名首字母 + unique Left(Text(Now()*1),4)
    //1. 年月日 yyyyMMdd Year(Today())&Month(Today())&Day(Today()
    //2. 申请人姓首字母+ 名首字母 Left(Office365Users.UserProfile(User().Email).GivenName,1)&Left(Office365Users.UserProfile(User().Email).Surname,1)
    //3. unique Left(Text(Now()*1),4)  Left(Text(Now()*1),4)
    /*RequestNumber:"IPR"&Year(Today())&Month(Today())&Day(Today())& Left(Office365Users.UserProfile(User().Email).GivenName,1)
    &Left(Office365Users.UserProfile(User().Email).Surname,1)&Left(Text(Now()*24),4),*/
        
        
        ProductName:If(TextInput3.Text="Jiegeniubi",TextInput4.Text, Dropdown_Product.Selected.Result),
        
        Quantity: TextInput8.Text,
        Person: TextInput3.Text,
        Title: Left(Text(Now()*24),4),
        Colorimportant:Checkbox1.Value,
        Index: CountRows(colProductA)+1
        
    }
);
//完成action之后，需要重设空格
Reset(TextInput8);
Reset(Dropdown_Product);
Reset(TextInput3);
Reset(Checkbox1);
Reset(Dropdown_Type);
Reset(Dropdown_Series)

//暂时不要需要显示成功窗口
//Set(varShowSuccess,true)
```




```
Set(varRN,
Patch('INTERN PRODUCT REQUEST', Defaults('INTERN PRODUCT REQUEST'),
    {
        Title:Year(Today()) & Month(Today()) & Day(Today())& Hour(Now())& Minute(Now()),
        'Reqeust date': Today(),
        'type request':Radio2.SelectedText.Value,
        'Department of resquester ': Office365Users.UserProfile(User().Email).Department,
        'Author (Author0)': User().Email,
        Requester: User().FullName,
        'Expected date': DatePicker1.SelectedDate,
        Propose:TextInput2.Text,
        ETA:DatePicker1.SelectedDate,
        Recipient:TextInput5.Text,
        ContactAddress:TextInput6.Text,
        NeedShip:varNeedShip,
        Comments:TextInput6_1.Text,

    // request number逻辑 mr+ 年月日 yyyyMMdd + 申请人姓首字母+ 名首字母 + unique Left(Text(Now()*1),4)
    //1. 年月日 yyyyMMdd Year(Today())&Month(Today())&Day(Today()
    //2. 申请人姓首字母+ 名首字母 Left(Office365Users.UserProfile(User().Email).GivenName,1)&Left(Office365Users.UserProfile(User().Email).Surname,1)
    //3. unique Hour(Now())& Minute(Now())
        RequestNumber: "IPR" & Year(Today()) & Month(Today()) & Day(Today()) & Left(Office365Users.UserProfile(User().Email).GivenName,1) & Left(Office365Users.UserProfile(User().Email).Surname,1) & Hour(Now())& Minute(Now()),


        //test阶段直接禁止发送 notification  否则发送true
        'Allow sent notification': true
    }
).RequestNumber); //getrequestn number in varRN


Collect(
    '2-INTERN PRODUCT REQUEST',AddColumns(colProductA,"RequestNumber",varRN)
);


//清楚 col product中的数据

Navigate(EndPage)
```

# 2. Page2 
