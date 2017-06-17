---
layout: post
title: "حفظ و استعادة Backup ف MSSQL Database بال #C"
description: ""
category: 
tags: [db,mssql,C#]
summary: "حفظ و استعادة Backup ف MSSQL Database بال #C"
crawlertitle: "حفظ و استعادة Backup ف MSSQL Database بال #C"
date:   2014-02-23 23:09:47 +0700
categories: posts
author: shawkyz
bg: 'database-backup-ic59flemi_emresupcin.png'
---
ازاي تاخد Database Backup من البرنامج بتاعك و ازاي تعمل Restore

مدبأيا ازاي نعمل Backup لـ Database MSSQL اصلا ؟

لو عندنا مثلا Database اسمها TestDB و عايز اعملها Backup على ال C:\TestDB.bak

هنكتب الامر ده ف ال SQL
{% highlight c %}
BACKUP DATABASE TestDB TO DISK ='C:\TestDB.bak'
{% endhighlight %}
طيب نعملها ازاي ف ال App بتاعنا

ممكن ببساطة نضيف folderBrowserDialog عشان بس نحدد مكان ال Folder اللي الـ User عايز يحفظ فيه ال BackUp

و ناخد بعد كده ال path اللي اختاره و نشتغل عليه احنا

مثال بسيط

انا عامل كلاس اسمه Database فيه كل ال Functions الخاصة بالتعامل مع ال Database بتاعتي

و واخد من الكلاس ده object اسمه db و فيه Function اسمها BackUpDB و بتاخد parameter هو string path

دلوقتي لما ال user يدوس على button بتاع ال Browse اللي هيفتحله ال folderBrowserDialog

ده هيكون code ال button click
{% highlight c %}
private void button6_Click(object sender, EventArgs e)
{
folderBrowserDialog1.ShowDialog();
string path = folderBrowserDialog1.SelectedPath;
if(path!="")
{
 
db.BackUpDB(path);
MessageBox.Show("تم الحفظ بنجاح");
}
else { MessageBox.Show("خطأ في الحفظ"); }
}
{% endhighlight %}
ببساطة باخد منه ال path و بديه لل BackUpDB Function

ايه اللي بيحصل ف ال BackUpDB Function بقى
{% highlight c %}
public void BackUpDB(string path)
{
con.Open();
SqlCommand cm = new SqlCommand("USE master", con);
cm.ExecuteNonQuery();
SqlCommand cmd = new SqlCommand("BACKUP DATABASE TestDB TO DISK = '" + path + "\\TestDB" +DateTime.Now.ToShortDateString().Replace('/','-') + ".bak'", con);
 
cmd.ExecuteNonQuery();
con.Close();
}
{% endhighlight %}
طيب دلوقتي انا عايز اعمل restore

ال restore هيجيلك exception غبي كده هيقولك ان فيه connection لل database مفتوحة و انها بتستعمل دلوقتي .. طب عشان نخلص من الحوارات دي نعمل ايه ؟

هنعمل 4 حاجات

1- هنشتغل على Master بدل ال Database بتاعتنا بمعنى اني هعمل Use master

2- هخلي ال database بتاعتي Single_User

3-هعمل Restore With Replace بأمر زي ده كده
{% highlight c %}
RESTORE DATABASE TestDB FROM DISK=’C:\TestDB.bak’ With Replace
{% endhighlight %}
4- هرجع ال database بتاعتي تاني Multi_User

طيب نعملها ازاي ف ال App بتاعنا

هنعمل المرادي SaveFileDialog عشان نحدد ال path اللي عايز يحفظ فيه ال Backup و ناخد ال path ده و نعمل restore منه

مثال بسيط

الكلاس بتاع ال Database في Function اسمها RestoreDB بتاخد string path برضه

دلوقتي لما ال user يدوس على button عشان يفتح ال SaveFileDialog هاخد ال FileName من ال SaveFileDialog ده و هبعته للـFunction
{% highlight c %}
private void button7_Click(object sender, EventArgs e)
{
openFileDialog2.Filter = "Backup File (*.bak)|*.bak";
openFileDialog2.FileName = null;
openFileDialog2.ShowDialog();
string path = openFileDialog2.FileName;
if (path != "")
{
try
{
db.RestoreDB(path);
MessageBox.Show("تم ");
Application.Restart();
}
catch { MessageBox.Show("خطأ"); }
}
else
{
MessageBox.Show("خطأ في الاسترجاع");
}
}
{% endhighlight %}
و ده اللي بيحصل جوة ال Function بتاعت RestoreDB
{% highlight c %}
public void RestoreDB(string path)
{
 
con.Open();
SqlCommand cm = new SqlCommand("USE master", con);
cm.ExecuteNonQuery();
string Alter1 = @"ALTER DATABASE [TestDB] SET Single_User WITH Rollback Immediate";
SqlCommand Alter1Cmd = new SqlCommand(Alter1, con);
Alter1Cmd.ExecuteNonQuery();
try
{
SqlCommand cmd = new SqlCommand("RESTORE DATABASE TestDB FROM DISK='" + path + "' With Replace", con);
cmd.ExecuteNonQuery();
}
catch { }
string Alter2 = @"ALTER DATABASE [TestDB] SET Multi_User";
SqlCommand Alter2Cmd = new SqlCommand(Alter2, con);
Alter2Cmd.ExecuteNonQuery();
con.Close();
}
{% endhighlight %}