---
title: perl学习
date: 2017-12-12
author: 杨金坤
tags:
  - 其他
categories:
  - perl
thumbnail: img/thumbnail/7.jpg
blogexcerpt:
    perl编程学习，一些常用的代码。
---
    
## perl shell 头
```
#!/usr/bin/perl

require Module::CoreList;
```


## 数值计算

```
my $pia=3.1415926;
my $r=12.5;

print 2*$pia*$r;
print "\n";
```


## 计算圆周长 输入半径

```
print "输入圆的半径：";
my $pia=3.1415926;
chomp ($r = <STDIN>);
if ($r < 0 ) {
  print "周长:0\n";
} else {
  print "周长:".2*$pia*$r."\n";
}
```


## 多行输入

```
my @char = <STDIN>;
print reverse @char;
```


## 求和计算


```
sub total{
   $sum = 0;
 
   foreach $item (@_){
      $sum += $item;
   }
   return $sum;
 }
my @fred = qw{ 1 3 5 7 9 };
my $fred_total = &total(@fred);
print "The total of \@fred is $fred_total.\n";
print "Enter some numbers on separate lines: ";
my $user_total = &total(<STDIN>);
print "The total of those numbers is $user_total.\n"; 

print &total(1..1000);
```


## 字符排序

```
my @char = <STDIN>;
print sort @char;

my %last_name = qw{fred flintstone Wilma Flintstone Barney Rubblebetty rubble Bamm- Bamm Rubble PEBBLES FLINTSONE};
foreach my$key(sort { lc($last_name{$a}) cmp  lc($last_name{$b}) } keys %last_name){

    print"$key=>$last_name{$key}\n";

}
```


## 位置查找

```
my $char="This is a test.";
chomp ($sub = <STDIN>);
my $ind=0;
while ($ind < length($char)){
	$ind=index($char,$sub,$ind);
	if($ind == -1) {
		last;
	}
	print $ind++." ";
}
print "\n";
```


## 目录操作

```
print "输入需切换目录：";
chomp ($dir = <STDIN>);
chdir( $dir ) or die "无法切换目录到".$dir."\n";
print "你现在所在的目录为 $dir\n";
print `ls -l`;
opendir ($dir, '.');
while ($file = readdir $dir) {
  print "$file\n";
}
closedir $dir;
```


## 查看目录
```
print "输入需切换目录：";
chomp ($dir = <STDIN>);
chdir( $dir ) or die "无法切换目录到".$dir."\n";
print "你现在所在的目录为 $dir\n";
print join("\n",sort glob(".* *"))."\n";
```

## 文件操作-删除

```
unless(@ARGV) {
	die "perl $0 <文件名> \n";
}
foreach(@ARGV) {
unlink ($_);
}
```

## 文件操作-重命名

```
if ( @ARGV != 2 ) {
	die "perl $0 <原文件名>  <新文件名> \n";
}
rename($ARGV[0],$ARGV[1]);
```


## 日期截取

```
#Date:2005-05-31;
#判断当前日期是一个星期的第几天;
#weekday（周一至周五），则输出gettowork;否则，输出goplay;
if(`date`=~/^S/){
print"goplay!\n";
}else{
print"gettowork!\n";
}
```


## 模块

```
my %modules = %{$Module::CoreList::version{5.006}}; 
print join ', ', Module::CoreList->find_modules();
print $Module::CoreList::version{5.006};
print keys %modules;
#print join ', ', Module::CoreList->find_modules(qr/Cwd/);
```


## 目录-全路径拼接

```
use Cwd;
use File::Spec;
use File::Basename;
my $dir = getcwd;
my $fl = "";
foreach(glob "*") {
	 $fl = File::Spec->catfile($dir,$_)."\n";
	 print $fl;
	 $fl = basename $fl;
	 print $fl;
};
```


## 文件链接


```
unless(@ARGV) {
	die "perl $0 <dir> \n";
}
foreach(glob "*") {
	 my $link = readlink "$_";
	 if(defined $link) {
	 	print "$_ -> $link\n";
	}
};

link $ARGV[0],$ARGV[1] or warn "不能创建连接";

symlink $ARGV[0],$ARGV[1] or warn "不能创建连接";
```



## 哈希-输出

```
my @char = <STDIN>;
my $r = join('',@char);
my %name = ("1"=>"fred","2"=>"betty","3"=>"barney","4"=>"dino","5"=>"Wilma","6"=>"pebbles","7"=>"bamm-bamm");
@char = split('\n',$r);
my $size = @char;
my $a=0;
while( $a < $size ){
    print "$name{$char[$a++]"};
}
```




## 重复打印

```
print "打印字符指定次数\n输入字符：";
my $char = <STDIN>;
print "输入需要打印的次数：";
chomp ($cnt = <STDIN>);
print $char x $cnt;
```


## 计算乘积

```
print "两个数值的乘积\n输入数字：";
chomp ($num1 = <STDIN>);
print "输入数字：";
chomp ($num2 = <STDIN>);
print "$num1 * $num2 = ".$num1*$num2."\n";
```



## 根据输入匹配姓名

```
my @fname=("flintstone","rubble","flintstone");
my @name=("fred","barney","wilma");
my $size = @name;
my $a=0;
while( $a < $size ){
	  #print $fname[$a];
    if( $name[$a] eq "fred" ){
    # 布尔表达式为true时执行
    print $fname[$a]."\n";
    }
    $a=$a+1;
}
```


## 哈希 value 输出

```
print "given name?\n";
my $gname=<STDIN>;
my %name=("fred\n","flintstone","barney\n","rubble","wilma\n","flintstone");
print "family name :".($name{$gname} or die "family name :can not find ".$gname )."\n" ;
```

## 字符计数

```
print "type word like this :fred,barney,dino,wilma,fred";
my $var = <STDIN>;  
my $count = ($var =~ s/fred/fred/g);  
print $count;  
```


## 字符计数

```
print "type word like this :fred,barney,dino,wilma,fred!\n";
chomp ($var = <STDIN>);
my @array = split(',', $var);
my %count;
grep { ++$count{ $_ } < 2; } @array;
for $char ( keys %count )
{
     print("$char => $count{$char}\n");
}
```



## 文本替换

```
open(FILE,"<myfile.txt") or die "Can't open myfile: $!";
open(OFILE,">myfile.txt.out") or die "Can't open myfile: $!";
foreach(<FILE>) {
  $_ =~ s/fred/Larry/;
  print OFILE $_ ;
}
```




```
open(FILE,"<myfile.txt") or die "Can't open myfile: $!";
open(OFILE,">myfile.txt.out") or die "Can't open myfile: $!";
foreach(<FILE>) {
  $_ =~ s/fred/<replace>/i;
  $_ =~ s/wilma/fred/i;
  $_ =~ s/<replace>/wilma/i;
  print OFILE $_ ;
}
```


## 随机试验


```
chomp($guess = <STDIN>);
while( $guess != "" and $guess != "exit" and $guess != "quit") {
	if ($guess > int(1 + rand 100) ) {
		print "too high\n";
	} elsif ($guess < int(1 + rand 100)) {
		print "too low\n";
	} else {
		print "corect\n";
		last;
	}
	chomp($guess = <STDIN>);
}
```

## 简单正则

```
print grep /wilma/ && /fred/, <FILE>; # 同时包含wilma 和 fred 
my $what = "fred|barney";
print grep /($what){3}/,<FILE>;
print grep /\b[a-z]+a\b/,<FILE>; # 单词以a结尾
print grep /^([^A-Z]+)?[A-Z]([^A-Z]+)?$/, <FILE>; #只含一个大写字母

my @char = ("x","x ","xx  ");
print grep(s/\s+$/_\n/,<STDIN>); # 空白结尾
```



## 格式化输出

```
format myformat =
12345678901234567890
@>>>>>>>>>>>>>>>>>>>
$name
.

$~ = "myformat";

while(1)
{
   chomp ($name = <STDIN>);
   write;
}
```




```
format myformat =
@>>>>>>>>>>>>>>>>>>>
$name
.

$~ = "myformat";
@char = <STDIN>;
foreach(@char) {
  $name=$_;
  write;
}
```



## 倒序打印

```
open(FILE,"<myfile.txt") or die "Can't open myfile: $!";
foreach (reverse <FILE>){
	print "$_";
}
```



## 平均数

```
sub average{
   # 获取所有传入的参数
   $n = scalar(@_);
   $sum = 0;
 
   foreach $item (@_){
      $sum += $item;
   }
   $sum / $n;
}
```

## 大于平均数

```
sub above_average{
   # 获取所有传入的参数
   $avg = average(@_);
   $above = "";
   foreach $item (@_){
   	  if ( $item > $avg) {
   	    $above = "$above $item"
   	  }
   }
   return $above;
}
```




```
my @fred = &above_average(1..10);
print "\@fred is @fred\n";
print "(Should be 6 7 8 9 10)\n";
my @barney = &above_average(100, 1..10);
print "\@barney is @barney\n";
print "(Should be just 100)\n";
```


