---
layout: post
title: "ASP.Net Web Application Security"
description: "" 
tags: [ASP.Net,Security]
summary: "ASP.Net Web Application Security"
crawlertitle: "ASP.Net Web Application Security"
date:   2016-04-16 23:09:47 +0700
categories: posts
author: shawkyz
bg: 'logo-aspnet.jpg'
---
في التدوينة دي هتكلم  بصورة عامة عن بعض الخطوات اللي ممكن تعملها عشان تحمي ال Web App بتاعك.

في العادي ممكن تطبقها على اي Web App بس هقترح Libraries او tools او Best Practices ممكن تستعملها لو شغال بال ASP.Net.

## [Cross Site Scripting](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS))

فيه طرق كتير معروفة عشان تـPrevent ال XSS بانواعها ، هتعتمد يا اما على ال Input Validation 
انك بتشوف هل ال User مثلًا حاول يكتب tag معينة زي script tag فتروح تمسحها، او انك تـ Html Encode ال output.

- مبدأيًا ASP.Net بتوفرلك Input Validation لل Malicious Characters ، و ده حلو ، بس المشكلة انه بيعتمد على Black List.
بمعنى فيه زي list كده ببعض ال malicious payloads اللي ممكن تسببلك مشاكل ، و ده اكيد مش هينفع معاك ، حتى لو الليست فيها 10000 payload ، ممكن حد يعديها.
فالحل لو عايز تعمل  Input Validation استعمل Whitelist ، يعني هتسمح بـ Inputs معينة بس و الباقي هتشيله ، ممكن تستعمل AntiXss Library ممكن تنزلها من Nuget 
{% highlight c %}
PM> Install-Package AntiXSS
{% endhighlight %}
[هتلاقي ال documentation بتاعها من هنا](https://msdn.microsoft.com/en-us/library/system.web.security.antixss.antixssencoder(v=vs.110).aspx)

- برضه AntiXss ممكن تHtml Encodeالoutput بتاعك.

- في حالة انك مثلا عايز تعمل Html Live Editor لأي سبب كان ، أكيد ده مش هيبقى مخاطره اكتر انك تعمله ، بس لو اضطريت هتحتاج تـSanitize ال Html ده و تعتمد على Whitelist.
فيه Library  كنت استعملتها قبل كده بسيطة جدًا ، و فيها ال WhiteList و ممكن تعدل عليها ، و كمان مكتوبلها Unit Tests على كل ال payloads اللي مكتوبة في الXSS Cheat Sheet من OWASP ، 
 [ده اللينك بتاعها](https://github.com/mganss/HtmlSanitizer)

- استعمل ال Header X-XSS-Protection 

- Flag ال Cookies بتاعتك بـ HTTPOnly عشان في حالة لو فيه فعلًا XSS الattacker ميقدرش يوصل للـ Cookies.

<br/>

## [Cross Site Request Forgery](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF))

- من اول VS 12 لما تعمل ASP.Net Web Forms App في ال Master Page بيكون فيه Prevention لل CSRF ، 
لو شغال على project 
[قديم ارجع للينك ده](http://software-security.sans.org/developer-how-to/developer-guide-csrf)
ده هيحميك لو هتستعمل ال Master Page دي و هتستعمل ال ASP.Net View States 
بس فرضًا لو كنت كتبت قبل كده صفحة او Web Service مثلا و بتاخد parameter بـ Get Request و معملتش اي check ، حاجة مثلا زي
{% highlight c %}
GET SITE/deleteuser.aspx?id=1
{% endhighlight %}
يبقى اكيد اللي كان مكتوب في ال Master Page مش هيبقى ليه أي تأثير في حالتك ، فهتضطر تـimplement انت ال check بنفسك ، ممكن ترجع للـ [reference ده](https://www.owasp.org/index.php/Anti_CSRF_Tokens_ASP.NET)


- ال ASP.Net MVC بت prevent ال CSRF بانها بتحط 2 tokens واحد في ال HTML Form و التاني بيبقى  الcookie و ده لحد كده كويس.
بس برضه هتقابلك نفس المشكلة لو عندك مثلا Web Service و هتبعتلها Ajax Request من الClient فهتحتاج الtokens دي تضيفها لل Request بتاعتك [ممكن تشوف اللينك ده](http://www.asp.net/web-api/overview/security/preventing-cross-site-request-forgery-csrf-attacks)


<br/>

## [ClickJacking](https://www.owasp.org/index.php/Clickjacking)

قبل ما تفكر في الحل اسال نفسك ، اي هي الصفحات اللي هتتاخد كـ iframe ؟ 
هل كلها هتستعمل كـ iframe ؟ طب هتستعمل برة ال domain بتاعي ؟
عامة كده كده لازم الصفحات اللي ليها علاقة ال settings بتاعت ال user مثلا او بيحصل فيها اي عمليات حساسة ، لازم تخليها متسمحش انها تتاخد كـ iframe برة ال domain بتاعك. عشان ت Prevent ال Click Jacking هتضيف ال **x-frame-options header** و هتـ set ال attribute يا اما

* Deny
* ALLOW-FROM
+ SAMEORIGIN

لو **DENY** هتمنع ان الصفحة تتـrender في اي iframe.

او **SAMEORIGIN** و هتتrender بس في حالة ان ال parent هيكون من نفس ال domain 

او **ALLOW-FROM** و بتحدد انت ال top level browing context يكون من domain معين ،

 تفاصيل اكتر [هتلاقيها هنا](https://blogs.msdn.microsoft.com/ieinternals/2010/03/30/combating-clickjacking-with-x-frame-options/)


<br/>

## [Unrestricted File Upload](https://www.owasp.org/index.php/Unrestricted_File_Upload)

اكيد مش هينفع تسمح  للـ user انه يـ Upload اي file عندك ، فلازم يكون عندك Whitelist بانواع ال files اللي هتسمح بيها بس. 
مثلا انت هتسمح بـ : txt, doc, pdf 
فممكن تعمل 2 Checks 

* الاول تشوف ال File Type اللي جاي و لو مش موجود في ال Whitelist بتاعتك متقبلوش.
* التاني هتشوف ال File Signature ، كل File Type بيكون ليه FIle Signature معينة بتكون في اول 20 byte منه 
[ممكن تشوفهم من هنا](https://en.wikipedia.org/wiki/List_of_file_signatures)
فبرضه هتعمل Whitelist بال File Signatures اللي هتقبلها و تcheck لو كان ال File اللي جاي ده ال signature بتاعته موجودة في ال Whitelist ولا لأ بانك هتقرا اول 20 byte من ال File ، ده عشان مثلًا لو ال user غير ال extension بتاع ال exe file لـ txt مثلا ، في العادي دي مش مشكلة طلاما هو مش executable و محدش هيعمله execute، بس فرضًا لو كان فيه اي flow عندك تخليه يغير اسم ال file مثلا فساعتها ممكن تبقى مشكلة.
عامة  "Better Safe Than Sorry".

<br/>

## [SQL Injections](https://www.owasp.org/index.php/SQL_Injection)

- اسهل طريقة عشان تـ prevent ال SQL Injection انك تستعمل Parameterized Queries ، يعني لو انت شغال بال ADO.Net مثلًا و هتستعمل SqlCommand Class و ضيف ال Parameters بتاعتك من ال SqlCommand Object ، زي مثلا command.Pramaters.Add(..) بدل ما تكتب ال query بتاعك و concat بالparameters على طول.

- و لو بتستعمل ORM زي NHibernate او Entity FrameWork بيكون فيهم Implemented Mechanisms عشان تـ Prevent ال Sql Injections.

* اياك تstore ال passwords كـ Plain Text ، لازم تـ Encrypt و تـ Salt الHash اللي طلع بعد كده تـ Store

<br/>

## [Privilege Escalation](https://www.owasp.org/index.php/Testing_for_Privilege_escalation_(OTG-AUTHZ-003))

لو عندك مثلا 2 Roles للـ Users بتوعك (Admin,User) فلازم تتأكد ان ال User العادي ميقدرش ينفذ نفس العمليات اللي يقدر ينفذها ال Admin.
أسهل طريقة عشان تtest بيها الموضوع انك تحاول تـ Execute Function المفروض انها تشتغل بصلاحيات الـ Admin بس.
عشان تـ Prevent الـ Privilege Escalations الحل الوحيد انك تتأكد ان كل User تحت Role ميقدرش ينفذ اي عمليات تانية برة ال roles scope بتاعته ، فلازم تعمل permission check عالFunctions اللي هتتنفذ لكل rule منهم. غير كده اكتب Unit Tests و جرب Test Cases زي مثلا انك تـ Execute Function معينة و انت معندكش الPermission.


<br/>

## [Indirect Object Reference](https://www.owasp.org/index.php/Testing_for_Insecure_Direct_Object_References_(OTG-AUTHZ-004))
عشان تـ Prevent ال IDOR لازم تتأكد من كذا حاجة:

1. ال Ids عندي مينفعش انها تتـguess ، خلي الـ Ids بتاعتك GUID مثلًأ.
2. اتاكد ان كل User تحت Role معينة ميقدرش يعمل اي حاجة برة ال permission set اللي انت محددها في ال Role دي.


<br/>

## [Open Redirection](https://www.owasp.org/index.php/Unvalidated_Redirects_and_Forwards_Cheat_Sheet)
ابسط طريقة انك تـCheck لو كان ال Url اللي عايز يروحله Local Url ولا لأ ، لو لأ رجعه عال Index بتاعتك مثلًا او تخليه يـ Confirm انه هيـredirect برة ال domain بتاعك.
ممكن برضه تاخد [اللينك ده كـ reference](http://www.asp.net/mvc/overview/security/preventing-open-redirection-attacks)


<br/>

### مصادر ممكن تقرأ منها:

- [كتاب OWASP Top 10 For .Net من Troy Hunt](http://www.troyhunt.com/2011/12/free-ebook-owasp-top-10-for-net.html)
- [OWASP TOP 10 2013 Reference](https://www.owasp.org/index.php/Top_10_2013-Top_10)


<br/>

> حاجة اخيرة: لو بتستعمل اي third-party libraries دايمًا خليك up to date ، عشان ممكن مثلا لو بتستعمل JS Library معينة ، ممكن يكون هي نفسها فيها مثلًا XSS Vuln ساعتها انت برضه ممكن يبقى عندك نفس المشكلة.
