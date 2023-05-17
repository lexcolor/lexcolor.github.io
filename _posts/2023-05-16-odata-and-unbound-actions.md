---
title: OData and Unbound Actions
date: 2023-05-17 12:00:00 -500
categories: [BC API] 
tags: [API, AL]
---

According to the ODATA website, OData stands for Open Data Protocol, best practives to build and consume RESTFul APIs, and it is used by every Microsoft Solution, including Business Central.

Being oriented to REST APIs, it means exposing an entity so the end user can perform CRUD operations on that entity. Therefore, BC allows using the OData in API Pages, which in turn are based on a table.

So, whenever I Perform an insertion, the response of the API call is a structure containing the record just created, or modified.

Codeunits, on the other hand, are only accesible through SOAP URLs. Nevertheless, always having in sight the use case for OData, you can also access a codeunit using a OData URL, just with some limitations.

## Present Case

LS Central has the ability to run disconnected from the main database, being a POS system, is a critical ability. This means the architecture of the system is such, that I can have the ERP Database running in the office headquarters (HO in LS language), and a separate database running either at store level, or at terminal level.

There are standardized ways of keeping the information up to date, meaning inventory, customer balances, customer data, etc.

In this particular scenario, a POS command is required, which can show in the POS the balance of the customer. But being the terminal database a different database than the HO server or ERP server, either I'll have outdated data, or I have to reach another database. So, a web service is up to the task here, and unbound action.

## Solution

I have to expose a codeunit containing the endpoint where I'll recover the customer balance, like this:

``` AL
codeunit 50905 "IncomingActions"
{
    trigger OnRun()
    begin
    end;

    procedure GetCustomerBalance(custCode: code[20]): Decimal
    var
        cust: Record Customer;
        errm: Label 'Customer %1 not found...';
    begin
        cust.Reset();
        cust.SetRange("No.", custCode);
        if not cust.FindFirst() then
            Error(errm, custCode);

        cust.CalcFields(Balance);

        exit(cust.Balance);
    end;
}
```

Such codeunit is exposed as a web service, hence, I have a SOAP Address, where the exposed name is the same as the codeunit, IncomingActions; but I don't have an OData address.

If I go to postman and try the address

http://localhoist:7048/BC190/ODataV4/IncomingActions_GetCustomerBalance?Company=c7c44f43-dde8-ec11-86af-780cb8ecf733

As found in [this post by Arend-Jan Kauffmann](https://www.kauffmann.nl/2020/03/05/codeunit-apis-in-business-central/)

The company section is best explained in the mentioned post, so, i won't go into those details, except by saying that this particular company ID works for me, but you will have to find out your own ID. That is also explained in the link.

What I will repeat, is that I can sent as a responsde any simple data type, in this case, a decimal value.

I can't, however, return a JsonObject, and that is because, as I mentioned earlier, OData is meant to interact with your existing entities, so if you would like to return structured data, you should have that data in a table. Luckily for me, I need only the balance, and I'll get an error in case something goes wrong. Enough for my case.

(this post is still incomplete)