# PowerApps_How to Build a Multi-Level Linked Dropdown Menu 
Requirement: The business unit needs a system to automate the processing of internal product requests that will be used for product demonstration/internal use, etc. for non-sales purposes.
Processes such as product selection / request delivery / request approval / request tracking / product delivery / statistics need to be automated.

Thanks to the Azure work environment, we will use Power Apps + Power Automate to build this system

Front-end solutions: Building front-end requirements pages with Power Apps

# 1. Page Selection
In addition to the information entered on the requirements page itself, there is some information that the applicant needs to select from a qualifying list, such as product information:
As shown in the picture below

![menu dropdown](https://github.com/CptLNERV/PowerApps_EDWARD-InternalProductsRequestPlatform/assets/20716430/416f1f53-9006-464f-ac25-fbb70c8fee73) 

After linking the Sharepoint list as a database, we use the information in the database as an option

Level 1 options
```
ForAll(Distinct(ProductListForAPPS,type), {Result: ThisRecord.Value})
```
Level 2 options
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

Level 3 options
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
        // request number logic mr+ year month day yyyyMMdd + applicant's last name initial + first name initial + unique Left(Text(Now()*1),4)
    //1. Year Month Day yyyyyMMdd Year(Today())&Month(Today())&Day(Today())
    //2. Applicant Surname Initial + First Name Initial Left(Office365Users.UserProfile(User().Email).GivenName,1)&Left(Office365Users.UserProfile(User().Email).Surname,1)
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
//After completing the action, the space needs to be reset.
Reset(TextInput8);
Reset(Dropdown_Product);
Reset(TextInput3);
Reset(Checkbox1);
Reset(Dropdown_Type);
Reset(Dropdown_Series)

//Don't need to show the success window for now
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

    // request number logic mr+ year month day yyyyMMdd + applicant's last name initial + first name initial + unique Left(Text(Now()*1),4)
    //1. Year Month Day yyyyMMdd Year(Today())&Month(Today())&Day(Today())
    //2. Applicant Surname Initial + First Name Initial Left(Office365Users.UserProfile(User().Email).GivenName,1)&Left(Office365Users.UserProfile(User().Email).Surname,1)
    //3. unique Hour(Now())& Minute(Now())
        RequestNumber: "IPR" & Year(Today()) & Month(Today()) & Day(Today()) & Left(Office365Users.UserProfile(User().Email).GivenName,1) & Left(Office365Users.UserProfile(User().Email).Surname,1) & Hour(Now())& Minute(Now()),


        //test阶段直接禁止发送 notification  否则发送true
        'Allow sent notification': true
    }
).RequestNumber); //getrequestn number in varRN


Collect(
    '2-INTERN PRODUCT REQUEST',AddColumns(colProductA,"RequestNumber",varRN)
);


//Clearing data in col product

Navigate(EndPage)
```
