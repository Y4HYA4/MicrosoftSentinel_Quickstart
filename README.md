#  Sentinel PoC Quickstart

I had a great time bringing value with Sentinel, and here I will try to assist you in getting started with integrating this product into your business. 
Sentinel, in my experience, is excellent for increasing your employees' overall awareness asides from assisting in decision making, providing advanced insights and protect your entreprise from threats.

<img src="assets/mainBan.png"  >


> Maybe, a good introduction to the product before the technical ?

#  Microsoft Sentinel Technical Procedure
This technical documentation will walk you through the entire process, from setting up Sentinel to operating it and using its core features.
## Description

We'll go over the preliminary steps for configuring the product.

- Initialize the product on your entreprise Microsoft Azure tenant.
- Ingestion of the required **connectors**.
- Perform **analytics queries** to get alerted when threats appear.
- Operate and create **workbooks** for insightful reports.

# Connectors
The first step is to import the necessary connectors before running any queries on Sentinel.

### Quickstart Connectors 
- Office
- Azure Activity
- Azure Active Directory

### Search for connectors

**1-** On the left pane, click Connectors

**2-** In the search bar, where you can search for the desired connector

**3-** Once found, give it a click and then open connector page

<img src="assets/connectors1.png"  >


### The Connector page

**1-** The good news is once you ingest some connectors. There will be analytics rules, workbooks as well as queries applied within the connector bundle.

**2-** To be configured, each connector will need its own prerequisite.


**3-** The same is true for the configuration. However, here you can find configurations that are as simple as clicking and others that necessitate the use of APIs.

<img src="assets/mainconnectorspage.png"  >

# Analytics Rules

The Microsoft Team is attempting to include as many Analytics rule templates as possible. However, it is nearly impossible to cover everything, so you should be aware of how to include built-in queries as well as create your own with some knowledge of KQL.


Here i'll go through both cases.

## Built-In Rules

You can select the template provided within the product from the asset below, and it should become active in a matter of seconds.

Once in Sentinel, Click Analytics => Rules Templates.

<img src="assets/analytics1.png"  >


## Custom Analytics Rules
We'll walk through configuring the yaml file below, which includes all of the pipeline's parameters. 

However, for this demonstration, we will conduct it as a security operation.

Here is the YAML for Explicit MFA Deny. 


```yml
id: a22740ec-fc1e-4c91-8de6-c29c6450ad00
name: Explicit MFA Deny
description: |
  'User explicitly denies MFA push, indicating that login was not expected and the account's password may be compromised.'
severity: Medium
requiredDataConnectors:
  - connectorId: AzureActiveDirectory
    dataTypes:
      - SigninLogs
  - connectorId: AzureActiveDirectory
    dataTypes:
      - AADNonInteractiveUserSignInLogs
queryFrequency: 1d
queryPeriod: 1d
triggerOperator: gt
triggerThreshold: 0
status: Available
tactics:
  - CredentialAccess
relevantTechniques:
  - T1110
query: |
  let aadFunc = (tableName:string){
  table(tableName)
  | where ResultType == 500121
  | where Status has "MFA Denied; user declined the authentication" or Status has "MFA denied; Phone App Reported Fraud"
  | extend Type = Type
  | extend timestamp = TimeGenerated, AccountCustomEntity = UserPrincipalName, IPCustomEntity = IPAddress, URLCustomEntity = ClientAppUsed
  };
  let aadSignin = aadFunc("SigninLogs");
  let aadNonInt = aadFunc("AADNonInteractiveUserSignInLogs");
  union isfuzzy=true aadSignin, aadNonInt
entityMappings:
  - entityType: Account
    fieldMappings:
      - identifier: FullName
        columnName: AccountCustomEntity
  - entityType: IP
    fieldMappings:
      - identifier: Address
        columnName: IPCustomEntity
  - entityType: URL
    fieldMappings:
      - identifier: Url
        columnName: URLCustomEntity
version: 1.0.1
kind: Scheduled
```
 
This query will require **Azure Active Directory** connector for it to perform.

Go to Analytics but instead of templates, click **Create** then Scheduled query rule. 

<img src="assets/Custom-analytics-rule-1.png" >

As a best practice, refer to the pipeline and include a fit description so your team understands the role of this query.

Choose the severity indicated. And then choose your tactics. After that, click Next to set the rule.

<img src="assets/Custom-analytics-rule-2.png"  >

**1-** As for the query, once it is created, enter it in this field and wait for the product to reload.

**2-** Don't forget about the Entity mapping.

<img src="assets/Custom-analytics-rule-3.png"  >

Sentinel is constantly improving and incorporating artificial intelligence. When you include such queries, the product will be able to identify the correct entity and map it automatically.

This is extremely powerful as it allows you to not only find out what’s going on in your environment but also to perform SecOps quick.

Once Entities are set click Next to incidents Settings.

<img src="assets/Custom-analytics-rule-4.png"  >

**1-** What is most important here is to **enable incident creation** so that you can be notified via your product home page if any threats are triggering.

**2-** The second section is to organize your queries. This is helpful when you have tons of rules that make sense as groups. I'll leave the decision to you.

<img src="assets/Custom-analytics-rule-5.png"  >

**1-** You can receive an automated response that will take you to a logic app where you can decide what action to take when this rule is triggered.

In this case, I prefer sending an email to my manager about an analytics with a high severity and posting the threat to our team channel.

**2-** When you're finished, you'll see a summary of the entire rule. Continue with the creation by clicking **Create.**

<img src="assets/Custom-analytics-rule-6.png"  >

# Workbooks

## Built-In Workbooks

Microsoft provides a set of templates along the chosen connector that you can take advantage of directly and use.

**1-** On the left pane, Click **Workbooks.**

**2-** Click **Templates**.

**3-** Search for the desired workbook.

**4-** Click **save**, click to My workbooks besides the template and you should have it.

<img src="assets/workbooks/workbook-templates.png"  >


## Custom Workbooks
In addition to the templates, you can make your own. **JSON** and **ARM Templates** formats are supported by workbooks.

1- Go to **Workbooks**

2- Instead of Templates, Click **Add Workbook**

<img src="assets/workbooks/create-workbook.png"  >

<br>

Click on **Edit**

<img src="assets/workbooks/create-workbook-1.png"  >

<br>

**1-** Click on the **code <>**  <br>

**2-** This will print you the editor where you can decide the type.     
- For this demo we will use JSON, copy my [workbook](https://github.com/Azure/Azure-Sentinel/blob/master/Workbooks/SentinelCosts.json) and paste it on your environment.

**3-** Once done, click **Apply**   <br>

**4-** Click **Done Editing** 

**5-** Then, the **Save Icon**.

<img src="assets/workbooks/create-workbook-3.png"  >

Continue by entering the name of your workbook, the subscription, the RG, and the location.

Finally Click **Apply**
<img src="assets/workbooks/create-workbook-4.png"  >


That's how it's done. You can keep looking through the Microsoft repo for open source workbooks.

# Workbook Showcase

This section is where you can marvel at the fantastic insights generated by this feature.

## Azure Activity
<img src="assets/workbooks/Results/Azure-Sign-Ins.png"  >

<br>

## Azure Active Directory Audit Insights
<img src="assets/workbooks/Results/AAD-Audit.png"  >

<br>

## All In One Insights
<img src="assets/workbooks/Results/All-In-One.png"  >

<br>

## Multi-Factor Authentification Insights

<img src="assets/workbooks/Results/MFA1.png"  >

<br>

<img src="assets/workbooks/Results/MFA2.png"  >

<br>

<img src="assets/workbooks/Results/MFA3.png"  >

<br>

## Conditionnal Access Insights
<img src="assets/workbooks/Results/CA.png"  >

## Closure
That's it; your connectors and analytics should now be properly preconfigured. You also can add workbooks and create your own.


