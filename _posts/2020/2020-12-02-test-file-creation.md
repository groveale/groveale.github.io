---
title:  "Automated file creation for migration testing"
categories:
  - Migration
tags:
  - PowerShell
  - Automation
  - dotnet
---

A couple of weeks ago I needed to test migration throughput. And I mean actual throughput. This test needed to be more than just transferring a whole load of iso / large video files. It needed to be at least slightly representative of real world data.

I needed a quick and easy way to create lot's of files that I could then migrate.

Naturally I turned to PowerShell.

## Creating a File

My first thought was to use `New-Item`. Using this `cmdlet` I would be able to create lots of files and pre populate them with string values.

``` powershell
New-Item -Path . -Name "testfile1.txt" -ItemType "file" -Value "This is a text string."
```

This was a good start but I needed a way to control the sizing of the file and I wanted all the files to be different.

## Generating Content

I had a couple of solutions for this. My first approach was to try and generate some `Lorem ipsum`, I mean it's really easy to do that in Word surely it's possible in PowerShell. Wrong. Not natively at least. Incase you haven't generated this in Word. Try the following:

``` word
=lorem(50,5)
```

Mind Blown.

My next attempt was to use a [.net method](https://docs.microsoft.com/en-us/dotnet/api/system.web.security.membership.generatepassword?view=netframework-4.8) to generate passwords. This had the added benefit in that I could specify the length. Thus, controlling the size of my files.

To use the `GeneratePassword(Int32, Int32)` method from PowerShell you first need to first add the `System.Web` assembly to your shell session or script.

```powershell
Add-Type -AssemblyName System.web
[System.Web.Security.Membership]::GeneratePassword(10,2)
```

This would generate a 10 char password with 2 non-alphanumeric values. Pretty cool, however, when you start creating long passwords the method ignores the last parameter and drops in as many non-alphanumeric values as it fancies. This in itself is not a problem but you can get some pretty obscure character combinations that have been known to cause issues for certain cloud services... A story for another day :)

So I went for the next thing I seem to generate a lot of - Guids. I know that a guid is always going to be 36 characters and will only contain alphanumeric characters and hypernyms `-`. Interestingly it's much faster to generate Guids using dotnet functions than it is using PowerShell `cmdlets`. I wonder if this is the case for all PowerShell `cmdlets`?

```powershell
# Fast... almost 100 times faster
$dotnetGuid = [Guid]::NewGuid()
# Slow
$psGuid = New-Guid
```

Timings analysis. I was generating a 36 character password to keep it fair. 

``` shell
Total seconds for 10000 passwords   : 0.0129187
Total seconds for 10000 dotnetGuids : 0.0045163
Total seconds for 10000 psGuids     : 0.2653906

Total seconds for 10000 passwords   : 0.0101992
Total seconds for 10000 dotnetGuids : 0.0019209
Total seconds for 10000 psGuids     : 0.2704631

Total seconds for 10000 passwords   : 0.0115864
Total seconds for 10000 dotnetGuids : 0.0020043
Total seconds for 10000 psGuids     : 0.2690527
```

I know we are dealing with very small amounts of time here but when you we will be generating millions of items, it all adds up.

## Sizing

My final requirement is to control the size fo the file. Each Guid I generate is going to be 36 characters long so 36 bytes in size. So if I have 3 Guids per line that would be 108 bytes and an extra 2 bytes for newline - 110 bytes per line. To size the file I simply need the following calculations:

```powershell
#get number of lines (3 guids in a line(+2 for eol))
$count = [Math]::Floor($sizeOfFiles / (110))
#get any extra
$extraBytes = $sizeOfFiles % 110
```

Using the `[Math]:Floor()` we round down to the nearest whole number. This will give us the number of complete lines we will have in the file. Then using the mod operator `%` to get the number of extra bytes to build the final line which will be incomplete or less than 110 bytes.

## Wrapping it all up

I put this all together in a function, it takes a filepath and size (in bytes) you can use the cool PowerShell string sizing e.g. `1.5MB` which will do the byte conversation for you. This will create a text file full of Guids of any size. Now just get your looping logic sorted and start generating those files.

```powershell
function create-file($path, $size)
{
    #get a random filename in the specified directory
    $string = [System.IO.Path]::GetRandomFileName()
    ## add extension to filename
    $fileName = ($string -replace ".{4}$") + ".txt"
    $fn = [System.IO.Path]::Combine($path, $fileName)
    #set number of iterations (3 guids in a line(+2 for eol))
    $count = [Math]::Truncate($sizeOfFiles / (110))
    #get any extra
    $extraBytes = $sizeOfFiles % 110
    #create a filestream
    $fs = New-Object System.IO.FileStream($fn,[System.IO.FileMode]::CreateNew)
    #create a streamwriter
    $sw = New-Object System.IO.StreamWriter($fs,[System.Text.Encoding]::ASCII,128)
    do{
        ## 3 Guid in a line (108 bytes)
        $line = [Guid]::NewGuid().ToString()
        $line = ($line + $line + $line)
        #Write the line (3 guids)
        $sw.WriteLine($line)
        #decrement the counter
        $count--
    }while($count -gt 0)
    #add the remainder to the file
    $extraGuid = [Guid]::NewGuid().ToString()
    $extraGuid = ($extraGuid + $extraGuid + $extraGuid + "AG")
    $extraLine = $extraGuid.Substring(0,$extraBytes)
    $sw.Write($extraLine)
    #close the streamwriter
    $sw.Close()
    #close the filestream
    $fs.Close()
}
```

Hopefully this will be of use to someone. If not, an interesting insight into my problem solving approach at the very least :)
