---
layout: post
title: "How to get files information efficiently with PowerShell"
date:   2018-05-07 00:00:00 +0000
tags: PowerShell
comments: true
image:
  src: "/assets/images/unexpected-speed.jpg"
  title: "isn't always expected"
---

## Objective

Let's suppose you want to **get files information** in `Windows` using `PowerShell`. The reason of using `PowerShell` specifically can vary:
* you want to pass execution results further to some exclusive `PowerShell` commands through the [pipeline](https://docs.microsoft.com/en-us/powershell/scripting/getting-started/fundamental/understanding-the-windows-powershell-pipeline?view=powershell-6);
* you're limited to the usage of certain tools or libraries; or you just want to keep things consistent;
* you want to utilize `PowerShell` remote calls;
* you just want to use a modern and well-maintained tool.

Let's also state the requirements:
* you have a full path to each file; let's also assume that all those files are on one drive to keep things simple; therefore, you also have a base folder where all the files reside (which is, perhaps, more specific than `C:\`, but still has lots of files);
* some of those files can be absent -- you should just ignore them and only return information of the existing ones;
* the base folder (where all the files are) can have nested folders (with files of different extensions) and the wanted files could be in each one of those.

Sounds like a real-life<sup>TM</sup> task, doesn't it?

<!--preview-break-->

## Preconditions

Let's set some preconditions:

```powershell
# get wanted files containing folder location (that's why $fl)
# with hundreds of files
$fl = 'c:\path\to\folder\with\all\the\wanted\files'
# real files location
$rfl = $fl + '\_test\PowerShellTestFiles\folder3_with_files' + `
             '\folder32_with_files\folder321_with_files'

# extension of files to search
$ext = '.txt'
$anyOfExt = '*' + $ext

# real files exist in the filesystem
$realFileNames = ('dummy1' + $ext), ('dummy2' + $ext), ('dummy3' + $ext)
$realFilePathNames = ($rfl + '\dummy1' + $ext), ($rfl + '\dummy2' + $ext), `
                     ($rfl + '\dummy3' + $ext)

# fake files do not exist in the filesystem
$fakeFileNames = 'fakefile1.ext', 'fakefile2.ext', 'fakefile3.ext', `
                 'fakefile4.ext', 'fakefile5.ext', `
                 'Very_long_fake_file_name_5_1.file.ext'
$fakeFilePathNames = ($fl + '\fakefile1.ext'), ($fl + '\fakefile2' + $ext), `
                     ($fl + '\fakefile3.ext'), ($fl + '\fakefile4.ext'), `
                     ($fl + '\fakefile5.ext'), `
                     ($fl + '\Very_long_fake_file_name_5_1.file.ext'), `
                     '/fake/path/to/file6', '\fake\path\to\file7', `
                     '\file8', '/fake/path/to/file9.ext', '\to\file10.ext'

$allFileNames = $realFileNames + $fakeFileNames
$allFilePathNames = $realFilePathNames + $fakeFilePathNames
```
As you can see, I'm about to search three existing files and a handful of non-existing ones.

The actual folder I store my files in has the following stats:

```
Size: 13.95 MB (14,629,112 bytes)
Contains: 962 Files, 57 Folders
```

where most of the files are of the wanted `$ext` extension.

## The intuitive way

As a good power tool, `PowerShell` has a command (or cmdlet to be precise) right for that: `Get-ChildItem`. You can even pass a collection of wanted pathnames. Therefore, the code should be something like (variables like `$allFilePathNames` are defined in the [preconditions](#preconditions)):

```powershell
    Get-ChildItem -Path $allFilePathNames `
                  -Recurse -ErrorAction SilentlyContinue -Force
```

... right? Wrong!

![Wrong](/assets/images/wrong-cut.gif  "Wrong")

The following code:

```powershell
# run each command before measuring it's execution time
# to do all the implicit initial runtime actions in advance

' --- just get FILES ---'

    Get-ChildItem -Path $allFilePathNames `
                  -Recurse -ErrorAction SilentlyContinue -Force

Measure-Command { `
    Get-ChildItem -Path $allFilePathNames `
                  -Recurse -ErrorAction SilentlyContinue -Force
}
```

gives the following output:

`Windows 10` with an HDD and `PowerShell` version 5.1:

```powershell
--- just get FILES ---
-a----       2018-05-07     14:42             18 dummy1.txt
-a----       2018-05-07     14:58             22 dummy2.txt
-a----       2018-05-07     14:58             23 dummy3.txt

Ticks             : 818405014
Days              : 0
Hours             : 0
Milliseconds      : 840
Minutes           : 1
Seconds           : 21
TotalDays         : 0.000947228025462963
TotalHours        : 0.0227334726111111
TotalMilliseconds : 81840.5014
TotalMinutes      : 1.36400835666667
TotalSeconds      : 81.8405014
```

`Windows Server 2012` with an SSD and `PowerShell` version 4.0:

```powershell
--- just get FILES ---
-a---        2018-05-07     14:42         18 dummy1.txt
-a---        2018-05-07     14:58         22 dummy2.txt
-a---        2018-05-07     14:58         23 dummy3.txt

Ticks             : 254360031
Days              : 0
Hours             : 0
Milliseconds      : 436
Minutes           : 0
Seconds           : 25
TotalDays         : 0,000294398184027778
TotalHours        : 0,00706555641666667
TotalMilliseconds : 25436,0031
TotalMinutes      : 0,423933385
TotalSeconds      : 25,4360031
```

which is roughly **82 seconds** on `Windows 10` with an HDD and **25 seconds** on `Windows Server 2012` with an SSD.

## Improvements

So, let's try to do something about it. We can also add an extension filter:

```powershell
# # extension of files to search
# $ext = '.txt'
# $anyOfExt = '*' + $ext

' --- get FILES with filter ---'

    Get-ChildItem -Path $allFilePathNames `
                  -Filter $anyOfExt `
                  -Recurse -ErrorAction SilentlyContinue -Force

Measure-Command { `
    Get-ChildItem -Path $allFilePathNames `
                  -Filter $anyOfExt `
                  -Recurse -ErrorAction SilentlyContinue -Force
}
```

and the results is roughly **14 seconds** on `Windows 10` with an HDD and **15 seconds** on `Windows Server 2012` with an SSD.

That doesn't help us much.

## The fastest way

Luckily, `Get-ChildItem` also has `-Include` parameter, where you could pass wanted file names:

```powershell
' --- get FILES with filter and include ---'

    Get-ChildItem -Path $allFilePathNames `
                  -Filter $anyOfExt `
                  -Include $allFileNames `
                  -Recurse -ErrorAction SilentlyContinue -Force

Measure-Command { `
    Get-ChildItem -Path $allFilePathNames `
                  -Filter $anyOfExt `
                  -Include $allFileNames `
                  -Recurse -ErrorAction SilentlyContinue -Force
}
```

which gives us the best result of roughly **8 MILLIseconds** on `Windows 10` with an HDD and **3 MILLIseconds** on `Windows Server 2012` with an SSD.

## Conclusion

By effectively passing wanted filenames to `Get-ChildItem` twice -- the first time to `-Path` as parts of absolute pathnames and the second time to `-Include` as just filenames -- we have achieved the best command performance.

The usage of `-Include` here is a bit counter-intuitive, as:
* such command improvements look like code duplication;
* `-Include` has being shown (e.g. [here](https://tfl09.blogspot.com/2012/02/get-childitem-and-theinclude-and-filter.html)) to be slower than `-Filter` for some cases, therefore is used with caution.

But such change makes a substantial performance improvement.

I also left the `-Filter` parameter -- though it doesn't give a significant performance impact here, it could lead to much better performance in case of having many files of different extensions stored in the same folder.

Numbers to prove that the last result is the best one and the complete test script are in the section below.

## Possible options

The last version is a viable solution. But what are the other parameter combinations could we pass?

The following table contains the results of executing `Get-ChildItem` with  different parameter sets, where:
* "get FOLDER" means that we pass one value to `-Path` -- the path to the folder with all the wanted files;
* "get FILES" means that we pass an array of values to `-Path` -- the paths to each of the wanted files.

| Command description                | Time (milliseconds)              |
|                                    |`Windows 10`|`Windows Server 2012`|
|                                    | HDD        | SSD                 |
|                                    | `PowerShell`version:             |
|                                    | `5.1`      | `4.0`               |
|:-----------------------------------|:---------- |:--------------------|
| get FOLDER with filter and include | 211.0099   | 255.694             |
| get FILES with filter and include  | **5.0347** | **2.7944**          |
| get FOLDER with include            | 100.1992   | 126.2083            |
| get FILES with include             | 5.0245     | 2.968               |
| get FILES with filter              | 14524.8325 | 14724.6479          |
| just get FILES                     | 81840.5014 | 25436.0031          |

<br/>

As you can see:
* on both "get FILES" and "get FOLDER", applying `-Filter` doesn't make much of a difference -- but that's because the folder mostly contains files of the wanted extension, so IMO it should still be applied to consider the general case of having files of different extensions in the folder;
* recursive search (i.e. "get FOLDER with include" cases) with filenames passed to `-Include` is faster than the "no include" cases by two orders (roughly a hundred times) -- therefore, should be used this way while having only filenames and a base folder path where the files are in;
* if we know absolute pathnames of all the wanted files -- we should pass them to `-Path` as an array ("get FILES with include" cases) and their filenames to `-Include` -- as it executes another hundred times faster compared to the recursive search (i.e. "get FOLDER with include" cases).

Here's the whole test script:

```powershell
# get wanted files containing folder location (that's why $fl)
# with hundreds of files
$fl = 'c:\path\to\folder\with\all\the\wanted\files'
# real files location
$rfl = $fl + '\_test\PowerShellTestFiles\folder3_with_files' + `
             '\folder32_with_files\folder321_with_files'

# extension of files to search
$ext = '.txt'
$anyOfExt = '*' + $ext

# real files exist in the filesystem
$realFileNames = ('dummy1' + $ext), ('dummy2' + $ext), ('dummy3' + $ext)
$realFilePathNames = ($rfl + '\dummy1' + $ext), ($rfl + '\dummy2' + $ext), `
                     ($rfl + '\dummy3' + $ext)

# fake files do not exist in the filesystem
$fakeFileNames = 'fakefile1.ext', 'fakefile2.ext', 'fakefile3.ext', `
                 'fakefile4.ext', 'fakefile5.ext', `
                 'Very_long_fake_file_name_5_1.file.ext'
$fakeFilePathNames = ($fl + '\fakefile1.ext'), ($fl + '\fakefile2' + $ext), `
                     ($fl + '\fakefile3.ext'), ($fl + '\fakefile4.ext'), `
                     ($fl + '\fakefile5.ext'), `
                     ($fl + '\Very_long_fake_file_name_5_1.file.ext'), `
                     '/fake/path/to/file6', '\fake\path\to\file7', `
                     '\file8', '/fake/path/to/file9.ext', '\to\file10.ext'

$allFileNames = $realFileNames + $fakeFileNames
$allFilePathNames = $realFilePathNames + $fakeFilePathNames

# run each command before measuring it's execution time
# to do all the implicit initial runtime actions in advance

' --- get FOLDER with filter and include ---'

    Get-ChildItem -Path $fl `
                  -Filter $anyOfExt `
                  -Include $allFileNames `
                  -Recurse -ErrorAction SilentlyContinue -Force

Measure-Command { `
    Get-ChildItem -Path $fl `
                  -Filter $anyOfExt `
                  -Include $allFileNames `
                  -Recurse -ErrorAction SilentlyContinue -Force
}

' --- get FILES with filter and include ---'

    Get-ChildItem -Path $allFilePathNames `
                  -Filter $anyOfExt `
                  -Include $allFileNames `
                  -Recurse -ErrorAction SilentlyContinue -Force

Measure-Command { `
    Get-ChildItem -Path $allFilePathNames `
                  -Filter $anyOfExt `
                  -Include $allFileNames `
                  -Recurse -ErrorAction SilentlyContinue -Force
}

' --- get FOLDER with include ---'

    Get-ChildItem -Path $fl `
                  -Include $allFileNames `
                  -Recurse -ErrorAction SilentlyContinue -Force

Measure-Command { `
    Get-ChildItem -Path $fl `
                  -Include $allFileNames `
                  -Recurse -ErrorAction SilentlyContinue -Force
}

' --- get FILES with include ---'

    Get-ChildItem -Path $allFilePathNames `
                  -Include $allFileNames `
                  -Recurse -ErrorAction SilentlyContinue -Force

Measure-Command { `
    Get-ChildItem -Path $allFilePathNames `
                  -Include $allFileNames `
                  -Recurse -ErrorAction SilentlyContinue -Force
}

' --- get FILES with filter ---'

    Get-ChildItem -Path $allFilePathNames `
                  -Filter $anyOfExt `
                  -Recurse -ErrorAction SilentlyContinue -Force

Measure-Command { `
    Get-ChildItem -Path $allFilePathNames `
                  -Filter $anyOfExt `
                  -Recurse -ErrorAction SilentlyContinue -Force
}

' --- just get FILES ---'

    Get-ChildItem -Path $allFilePathNames `
                  -Recurse -ErrorAction SilentlyContinue -Force

Measure-Command { `
    Get-ChildItem -Path $allFilePathNames `
                  -Recurse -ErrorAction SilentlyContinue -Force
}
```

and the results:

`Windows 10` with an HDD:

```powershell
--- get FOLDER with filter and include ---
Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       2018-05-07     14:42             18 dummy1.txt        
-a----       2018-05-07     14:58             22 dummy2.txt        
-a----       2018-05-07     14:58             23 dummy3.txt        

Ticks             : 2110099
Days              : 0
Hours             : 0
Milliseconds      : 211
Minutes           : 0
Seconds           : 0
TotalDays         : 2.44224421296296E-06
TotalHours        : 5.86138611111111E-05
TotalMilliseconds : 211.0099
TotalMinutes      : 0.00351683166666667
TotalSeconds      : 0.2110099

--- get FILES with filter and include ---
-a----       2018-05-07     14:42             18 dummy1.txt        
-a----       2018-05-07     14:58             22 dummy2.txt        
-a----       2018-05-07     14:58             23 dummy3.txt        

Ticks             : 50347
Days              : 0
Hours             : 0
Milliseconds      : 5
Minutes           : 0
Seconds           : 0
TotalDays         : 5.82719907407407E-08
TotalHours        : 1.39852777777778E-06
TotalMilliseconds : 5.0347
TotalMinutes      : 8.39116666666667E-05
TotalSeconds      : 0.0050347

--- get FOLDER with include ---
-a----       2018-05-07     14:42             18 dummy1.txt        
-a----       2018-05-07     14:58             22 dummy2.txt        
-a----       2018-05-07     14:58             23 dummy3.txt        

Ticks             : 1001992
Days              : 0
Hours             : 0
Milliseconds      : 100
Minutes           : 0
Seconds           : 0
TotalDays         : 1.15971296296296E-06
TotalHours        : 2.78331111111111E-05
TotalMilliseconds : 100.1992
TotalMinutes      : 0.00166998666666667
TotalSeconds      : 0.1001992

--- get FILES with include ---
-a----       2018-05-07     14:42             18 dummy1.txt        
-a----       2018-05-07     14:58             22 dummy2.txt        
-a----       2018-05-07     14:58             23 dummy3.txt        

Ticks             : 50245
Days              : 0
Hours             : 0
Milliseconds      : 5
Minutes           : 0
Seconds           : 0
TotalDays         : 5.81539351851852E-08
TotalHours        : 1.39569444444444E-06
TotalMilliseconds : 5.0245
TotalMinutes      : 8.37416666666667E-05
TotalSeconds      : 0.0050245

--- get FILES with filter ---
-a----       2018-05-07     14:42             18 dummy1.txt        
-a----       2018-05-07     14:58             22 dummy2.txt        
-a----       2018-05-07     14:58             23 dummy3.txt        

Ticks             : 145248325
Days              : 0
Hours             : 0
Milliseconds      : 524
Minutes           : 0
Seconds           : 14
TotalDays         : 0.000584778153935185
TotalHours        : 0.0140346756944444
TotalMilliseconds : 14524.8325
TotalMinutes      : 0.842080541666667
TotalSeconds      : 14.5248325

--- just get FILES ---
-a----       2018-05-07     14:42             18 dummy1.txt        
-a----       2018-05-07     14:58             22 dummy2.txt        
-a----       2018-05-07     14:58             23 dummy3.txt        

Ticks             : 818405014
Days              : 0
Hours             : 0
Milliseconds      : 840
Minutes           : 1
Seconds           : 21
TotalDays         : 0.000947228025462963
TotalHours        : 0.0227334726111111
TotalMilliseconds : 81840.5014
TotalMinutes      : 1.36400835666667
TotalSeconds      : 81.8405014
```

`Windows Server 2012` with an SSD:

```powershell
--- get FOLDER with filter and include ---
Mode                LastWriteTime     Length Name
----                -------------     ------ ----
-a---        2018-05-07     14:42         18 dummy1.txt
-a---        2018-05-07     14:58         22 dummy2.txt
-a---        2018-05-07     14:58         23 dummy3.txt

Ticks             : 2556940
Days              : 0
Hours             : 0
Milliseconds      : 255
Minutes           : 0
Seconds           : 0
TotalDays         : 2,9594212962963E-06
TotalHours        : 7,10261111111111E-05
TotalMilliseconds : 255,694
TotalMinutes      : 0,00426156666666667
TotalSeconds      : 0,255694

--- get FILES with filter and include ---
-a---        2018-05-07     14:42         18 dummy1.txt
-a---        2018-05-07     14:58         22 dummy2.txt
-a---        2018-05-07     14:58         23 dummy3.txt

Ticks             : 27944
Days              : 0
Hours             : 0
Milliseconds      : 2
Minutes           : 0
Seconds           : 0
TotalDays         : 3,23425925925926E-08
TotalHours        : 7,76222222222222E-07
TotalMilliseconds : 2,7944
TotalMinutes      : 4,65733333333333E-05
TotalSeconds      : 0,0027944

--- get FOLDER with include ---
-a---        2018-05-07     14:42         18 dummy1.txt
-a---        2018-05-07     14:58         22 dummy2.txt
-a---        2018-05-07     14:58         23 dummy3.txt

Ticks             : 1262083
Days              : 0
Hours             : 0
Milliseconds      : 126
Minutes           : 0
Seconds           : 0
TotalDays         : 1,46074421296296E-06
TotalHours        : 3,50578611111111E-05
TotalMilliseconds : 126,2083
TotalMinutes      : 0,00210347166666667
TotalSeconds      : 0,1262083

--- get FILES with include ---
-a---        2018-05-07     14:42         18 dummy1.txt
-a---        2018-05-07     14:58         22 dummy2.txt
-a---        2018-05-07     14:58         23 dummy3.txt

Ticks             : 29680
Days              : 0
Hours             : 0
Milliseconds      : 2
Minutes           : 0
Seconds           : 0
TotalDays         : 3,43518518518518E-08
TotalHours        : 8,24444444444444E-07
TotalMilliseconds : 2,968
TotalMinutes      : 4,94666666666667E-05
TotalSeconds      : 0,002968

--- get FILES with filter ---
-a---        2018-05-07     14:42         18 dummy1.txt
-a---        2018-05-07     14:58         22 dummy2.txt
-a---        2018-05-07     14:58         23 dummy3.txt

Ticks             : 147246479
Days              : 0
Hours             : 0
Milliseconds      : 724
Minutes           : 0
Seconds           : 14
TotalDays         : 0,000170424165509259
TotalHours        : 0,00409017997222222
TotalMilliseconds : 14724,6479
TotalMinutes      : 0,245410798333333
TotalSeconds      : 14,7246479

--- just get FILES ---
-a---        2018-05-07     14:42         18 dummy1.txt
-a---        2018-05-07     14:58         22 dummy2.txt
-a---        2018-05-07     14:58         23 dummy3.txt

Ticks             : 254360031
Days              : 0
Hours             : 0
Milliseconds      : 436
Minutes           : 0
Seconds           : 25
TotalDays         : 0,000294398184027778
TotalHours        : 0,00706555641666667
TotalMilliseconds : 25436,0031
TotalMinutes      : 0,423933385
TotalSeconds      : 25,4360031
```
