# PowerApps_EDWARD-InternalProductsRequestPlatform


# 1. Page Product


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
