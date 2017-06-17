---
layout: post
title: "Host ASP.Net WebApps On Linux"
description: ""
tags: [ASP.Net,Linux,OpenSource]
summary: "Host ASP.Net WebApps On Linux"
crawlertitle: "Host ASP.Net WebApps On Linux"
date:   2015-02-17 23:09:47 +0700
categories: posts
author: shawkyz
bg: 'HighlightsvNext.png'
---
تكمن المشكلة في إنشاء موقع إلكتروني على الإنترنت في الإستضافة الخاصة بالموقع والتي تأتيك بين أحد الإختيارات التالية:

إستضافة Windows أم إستضافة Linux

وهل العميل على إستعداد للدفع مقابل هذه الإستضافة، وهذا كان هو التحدى لإي مبرمج يعمل بـ لغة ASP البرمجية لإنشاء وبرمجة المواقع

اليوم وبعد أن أصبح لدينا ASP vNext والذي أصبح منتح مفتوح المصدر بالكامل أصبح لدينا خوادم غير الـ IIS والتي كانت لا تعمل إلّا على خوادم Windows فقط

لم نكن نحلم في يوم من الأيام أن نستطيع عمل ASP.NET WebApp ونقوم بإنشاء إستضافة خاصة به على خادم Linux

الآن أصبح الأمر ممكنًا ومتاح للجميع، فمنذ فترة أعلنت شركة Microsoft عن ASP.NET vNext وأطلقت عليه الآن “ASP.Net 5” ويمكنك تطوير موقع كامل على خادم OSX و Linux ويمكنك إستضافة الموقع عليه

كيف يمكننا فعل ذلك؟ وماذا حدث بالظبط، وما هي الخطوات التي يجب أن اتخذها وما هي الـ vNext ؟

مبدأيًا: الـ ASP vNext جميعها مفتوحة المصدر يمكنك التعديل عليها بالكامل كما تحب وقراءة ومشاهدة الكود المصدري، بالإضافة إلى إرسال إقتراحات لـ شركة مايكروسوفت خاصة بتعديلاتك.

سوف تجد الـ repo على GitHub من هنا:

https://github.com/aspnet/home

يذكر أنّ شركة مايكروسوفت في الآونة الأخيرة بدأت تتحد مع فريق Mono فيمكننا في الفترة القادمة توقع ان اي تحديثات خاصة بالـ .NET سوف نجدها في الحال على Mono ولن نضطر للإنتظار بعد الآن بالإضافة إلى إن Mono كـ IDE سوف تتحسن بصورة كبيرة.

ما تم في في ASP vNext كان ببساطة نقلة تاريخية لعدة أسباب:

١- الـ CSharp Compiler موجود مفتوح المصدر تحت اسم Roslyn.

٢- لسنا بحاجة إلى إستخدام الـ IIS Web Server بعد الآن فقد أصبح هناك  Kestrel وبسببه يمكننا إستضافة المواقع على خوادم تعمل بـ Linux وكذلك OSX.

٣- أصبح الآن كل شئ على Nuget.

٤- بعد أن كان لدينا ملفين في المشروع الواحد، ملف خاص بملفات المشروع والثاني خاص بـ Nuget Packages أصبح الاثنين في ملف واحد تحت اسم Project.json

 

في هذه التدوينة سوف نتكلم عن كيف تبرمج Website بـاستخدام ASP vNext و و ما هي الخطوات اللازمة حتى تستضيف الموقع على Linux Machine ،الامثلة القادمة ستكون مرتبطة بتوزيعات لينكس المبنية على Debian.

مبدأيًا لكي تبرمجASP.Net Website على الويندوز فمن الممكن ان تستعملVisual Studio 2015 و هذا أسهل حل .. أيضًا ، من الممكن ان تستعمل اي محرر أخر.

فلنفترض الآن انك انتهيت من برمجة ال Website بالكامل و تريد استضافته على أحد خوادم Linux فما هي الخطوات اللازمة لذلك؟

اذا كنت تعتقد ان الخطوات القادمة من الصعب تنفيذها فلا تقلق ، فقد جهزنا لك Script سوف يقوم بكل هذه المهام من تلقاء نفسه.

1- تحتاج الى Mono و يكون أعلى من 3.4.1:

اكتب هذه الاوامر في ال Terminal لتحميل و تنصيب Mono 3.8.0

{% highlight c %}
sudo apt-get install build-essential

wget http://download.mono-project.com/sources/mono/mono-3.8.0.tar.bz2

tar -xvf mono-3.8.0.tar.bz2

cd mono-3.8.0/

./configure –prefix=/usr/local

make

sudo make install
{% endhighlight %}
2- نحتاج الى تحميل SSL Certificates خاصة بميكروسوفت:
{% highlight c %}
sudo certmgr -ssl -m https://go.microsoft.com

sudo certmgr -ssl -m https://nugetgallery.blob.core.windows.net

sudo certmgr -ssl -m https://nuget.org

sudo certmgr -ssl -m https://www.myget.org/F/aspnetvnext/

mozroots –import –sync
{% endhighlight %}
3- نحتاج الى KVM و KRE :

*اكتب ال UserName الخاص بـ السيرفر مكان {USER NAME}

(KRE (K Runtime Enviroment) , KVM (K Version Manager
{% highlight c %}
sudo apt-get install curl
curl -sSL https://raw.githubusercontent.com/aspnet/Home/master/kvminstall.sh | sh && source ~/.kre/kvm/kvm.sh
sudo -s
source /home/{USER NAME}/.kre/kvm/kvm.sh
kvm upgrade
{% endhighlight %}
 

4- الآن كل ما نحتاجه لتشغيل Kestrel Web Server اصبح جاهز ، و لكن Kestrel يعتمد على libuv فيجب تنصيبها له:

{% highlight c %}
sudo apt-get install gyp

wget http://dist.libuv.org/dist/v1.0.0-rc2/libuv-v1.0.0-rc2.tar.gz

tar -xvf libuv-v1.0.0-rc2.tar.gz

cd libuv-v1.0.0-rc2/

./gyp_uv.py -f make -Duv_library=shared_library

make -C out

sudo cp out/Debug/lib.target/libuv.so /usr/lib/libuv.so.1.0.0-rc2

sudo ln -s libuv.so.1.0.0-rc2 /usr/lib/libuv.so.1

5- تحميل Template جاهز حتى نتأكد ان كل الخطوات السابقة تمت بطريقة صحيحة :

sudo apt-get install git

git clone http://github.com/aspnet/home

cd home/samples/HelloMvc

kpm restore

kpm build
{% endhighlight %}
من الممكن عمل build من خلال الأمر kpm build ، او اذا كنت تريد ان تشغل الserver نكتب k kestrel

الان لو كتبت k kestrel و رايت الرسالة Started اذا فمن الممكن تعاين الموقع من خلال الbrowser ادخل على

http://localhost:5004

ملحوظة: يتم كتابة الاوامر مثل k kestrel او kpm build بداخل ال Directory الخاصة بالموقع.

*اذا كنت لا تريد القيام بكل هذه الخطوات من الممكن تحميل ال Script:

https://gist.github.com/ShawkyZ/0b3831329a0372e33aef#file-kestrel-configuration

*اذا كنت تريد ان تستضيف ال Site على Azure VM مثلًا ، فمن الممكن ان تنشئ واحدة و تستخدم (Putty) مثلًا لكي تتصل بها و تبدأ بعمل كل الخطوات السابقة و عند الانتهاء لا تنسى ان تضبط اعدادات ال EndPoint الخاصة بال VM.



