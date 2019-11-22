读取PDF文档原始字符串STRING

` TCPDF_PARSER ` 将字符串转为下面的格式

` TCPDF_PARSER `返回数组有两个元素，`xref`、和`data`
```
$response = [
    "xref" => [
        "xref" =>[],
        "trailer" =>[],
    ],
    "data" => [],
]
```
`xref`的子元素`xref.trailer`，`size`代表PDF文档有23个间接对象，`root`根对象是`1_0`
```
array:2 [▼
  "xref" => array:22 [▶]
  "trailer" => array:4 [▼
    "size" => 23
    "root" => "1_0"
    "info" => "3_0"
    "id" => array:2 [▼
      0 => "32F313AC7D6BBA39D2A8C4B61F5ECF09"
      1 => "32F313AC7D6BBA39D2A8C4B61F5ECF09"
    ]
  ]
]
```
`xref`的子元素`xref.xref`，一共23个元素，代表PDF文档有23个间接对象，下标`1_0`表示第1个间接对象，修改次数为0，在原始字符串STRING中，`1 0 obj`表示第一个对象开始，下标`1_0`对应的值17，表示从原始字符串STRING截取对象时，STRING下标为17的地方开始就是`1 0 obj`。简而言之，`xref.xref`就是告诉我们，有多少个对象，怎么截取每个对象对应的数据。`xref.xref`好比内存地址，根据内存地址去找变量。
```
array:2 [▼
  "xref" => array:22 [▼
    "1_0" => 17
    "2_0" => 66
    ......
    "21_0" => 273270
    "22_0" => 273531
  ]
  "trailer" => array:4 [▶]
]
```
`data`就是23个子元素的数据，有书签对象，目录对象，content对象，重点看content对象，里面就是要解析显示的文本数据。对应下面`5_0`元素。`Filter`表示要解码，解码方式为`FlateDecode`，`Length`长度为`numeric`9022。重点是`stream`数据流，乱码部分就是他对应的值，这是原始数据，我们要分析的重点。
```
array:22 [▼
  "1_0" => array:1 [▶]
  "2_0" => array:1 [▶]
  "3_0" => array:1 [▶]
  "4_0" => array:1 [▶]
  "5_0" => array:3 [▶]
  ......
  "22_0" => array:1 [▶]
]
```
```
  "5_0" => array:3 [▼
    0 => array:3 [▼
      0 => "<<"
      1 => array:4 [▼
        0 => array:3 [▼
          0 => "/"
          1 => "Filter"
          2 => 534
        ]
        1 => array:3 [▼
          0 => "/"
          1 => "FlateDecode"
          2 => 546
        ]
        2 => array:3 [▼
          0 => "/"
          1 => "Length"
          2 => 553
        ]
        3 => array:3 [▼
          0 => "numeric"
          1 => "9022"
          2 => 558
        ]
      ]
      2 => 560
    ]
    1 => array:4 [▼
      0 => "stream"
      1 => b"""
        x£┼]IÅ\x1D╣æ¥7á ­.\x03T\eV\x16¸\x05\x10\x04╝ÑÌ,\x18\x036¼┴\x1C\x1A>╚mu╗\x0Fû▄ì2<■¸ÄÓÆ/ÖI2ÿ»J×±@-Ue\x06?\x06â\x11\x1FâAµÒ±ùþƒ~°°²¾ß¦╗ÃÒ¾¾Ã´ ³ÚOç´\x1E?|¨Ù\x1F\x1E ▶
```

`Smalot\PdfParser\Document.php:parseContent($content)`将乱码字符串还原为PDF对象，就是下面这样的格式
```
5 0 obj
<< /Length 44 >>
stream
BT
/F1 24 Tf
100 100 Td (Hello World) Tj
ET
endstream
endobj
```
`smalot\pdfparser\src\Smalot\PdfParser\PDFObject.php:getText(Page $page = null)`将数据分割为单个字、词，格式如下
```
/**
* $commands
* array:1 [▼
   0 => array:3 [▼
       "t" => "<"
       "o" => ""
       "c" => "190B0B764B263E7C"
       ]
   ]
*/
```
`smalot\pdfparser\src\Smalot\PdfParser\Font.php:decodeText($commands)`
将`190B0B764B263E7C`这样的数据装维十六进制，最后在字典表找出对应的汉字返回




