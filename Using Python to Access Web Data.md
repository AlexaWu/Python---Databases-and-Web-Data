# Summary & Key Skills

- Use regular expressions to extract data from strings
- Understand the protocols web browsers use to retrieve documents and web apps
- Retrieve data from websites and APIs using Python
- Work with XML (eXtensible Markup Language) data

`Json`   `Xml`   `Python Programming`   `Web Scraping`


# Extracting Data With Regular Expressions 

_**Finding Numbers in a Haystack**_

In this assignment you will read through and parse a file with text and numbers. You will extract all the numbers in the file and compute the sum of the numbers.

_**Data Files**_

We provide two files for this assignment. One is a sample file where we give you the sum for your testing and the other is the actual data you need to process for the assignment.

> Sample data: http://py4e-data.dr-chuck.net/regex_sum_42.txt (There are 90 values with a sum=445833)\
 Actual data: http://py4e-data.dr-chuck.net/regex_sum_1037114.txt (There are 88 values and the sum ends with 873)

_**Data Format**_

The file contains much of the text from the introduction of the textbook except that random numbers are inserted throughout the text. Here is a sample of the output you might see:


> Why should you learn to write programs? 7746\
12 1929 8827\
Writing programs (or programming) is a very creative \
7 and rewarding activity.  You can write programs for \
many reasons, ranging from making your living to solving\
8837 a difficult data analysis problem to having fun to helping 128\
someone else solve a problem.  This book assumes that \
everyone needs to know how to program ...


The sum for the sample text above is **27486**. The numbers can appear anywhere in the line. There can be any number of numbers in each line (including none).


_**Handling The Data**_

The basic outline of this problem is to read the file, look for integers using the re.findall(), looking for a regular expression of '[0-9]+' and then converting the extracted strings to integers and summing up the integers.


###  My work:
#### Python code:
```javascript
import re
handle = open('regex_sum_1037114.txt')
file = handle.read()
numbers = re.findall('[0-9]+',file)
sum = 0

for text in numbers:
    text = int(text)
    sum = sum + text

print (sum)
```

#### Sum: 357873



# Understanding the Request / Response Cycle

_**Exploring the HyperText Transport Protocol**_

You are to retrieve the following document using the HTTP protocol in a way that you can examine the HTTP Response headers.

http://data.pr4e.org/intro-short.txt
There are three ways that you might retrieve this web page and look at the response headers:

**Preferred:** Modify the socket1.py program to retrieve the above URL and print out the headers and data. Make sure to change the code to retrieve the above URL - the values are different for each URL.
Open the URL in a web browser with a developer console or FireBug and manually examine the headers that are returned.

###  My work:
#### Python code:

```javascript
import socket

mysock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
mysock.connect(('data.pr4e.org', 80))
cmd = 'GET http://data.pr4e.org/intro-short.txt HTTP/1.0\r\n\r\n'.encode()
mysock.send(cmd)

while True:
    data = mysock.recv(512)
    if len(data) < 1:
        break
    print(data.decode(),end='')

mysock.close()
```

# Scraping HTML Data with BeautifulSoup

**Scraping Numbers from HTML using BeautifulSoup** In this assignment you will write a Python program similar to http://www.py4e.com/code3/urllink2.py. The program will use **urllib** to read the HTML from the data files below, and parse the data, extracting numbers and compute the sum of the numbers in the file.

We provide two files for this assignment. One is a sample file where we give you the sum for your testing and the other is the actual data you need to process for the assignment.

> Sample data: http://py4e-data.dr-chuck.net/comments_42.html (Sum=2553)\
 Actual data: http://py4e-data.dr-chuck.net/comments_1037116.html (Sum ends with 46)

_**Data Format**_

The file is a table of names and comment counts. You can ignore most of the data in the file except for lines like the following:

> <tr><td>Modu</td><td><span class="comments">90</span></td></tr>\
> <tr><td>Kenzie</td><td><span class="comments">88</span></td></tr>\
> <tr><td>Hubert</td><td><span class="comments">87</span></td></tr>

You are to find all the <span> tags in the file and pull out the numbers from the tag and sum the numbers.
    
#### Sample Code
It shows how to find all of a certain kind of tag, loop through the tags and extract the various aspects of the tags.

> ...\
 #Retrieve all of the anchor tags\
 tags = soup('a')\
 for tag in tags:\
    #Look at the parts of a tag\
    print 'TAG:',tag\
    print 'URL:',tag.get('href', None)\
   print 'Contents:',tag.contents[0]\
   print 'Attrs:',tag.attrs

You need to adjust this code to look for **span** tags and pull out the text content of the span tag, convert them to integers and add them up to complete the assignment.

#### Sample Execution
>$ python3 solution.py\
Enter - http://py4e-data.dr-chuck.net/comments_42.html \
Count 50\
Sum 2...

###  My work:
#### Python code:
```javascript
# To run this, download the BeautifulSoup zip file
# http://www.py4e.com/code3/bs4.zip
# and unzip it in the same directory as this file

from urllib.request import urlopen
from bs4 import BeautifulSoup
import ssl

# Ignore SSL certificate errors
ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

url = input('Enter - ')
html = urlopen(url, context=ctx).read()
soup = BeautifulSoup(html, "html.parser")

# Retrieve all of the anchor tags
tags = soup('span')
number = list()

for tag in tags:
    number.append(int(tag.string))

print(sum(number))
```
#### Sum: 2446

# Following Links in HTML Using BeautifulSoup

#### Following Links in Python

In this assignment you will write a Python program that expands on http://www.py4e.com/code3/urllinks.py. The program will use **urllib** to read the HTML from the data files below, extract the href= vaues from the anchor tags, scan for a tag that is in a particular position relative to the first name in the list, follow that link and repeat the process a number of times and report the last name you find.

We provide two files for this assignment. One is a sample file where we give you the name for your testing and the other is the actual data you need to process for the assignment

>Sample problem: Start at http://py4e-data.dr-chuck.net/known_by_Fikret.html \
Find the link at position **3** (the first name is 1). Follow that link. Repeat this process **4** times. The answer is the last name that you retrieve.\
Sequence of names: Fikret Montgomery Mhairade Butchi Anayah\
Last name in sequence: Anayah\
Actual problem: Start at: http://py4e-data.dr-chuck.net/known_by_Jonatan.html \
Find the link at position **18** (the first name is 1). Follow that link. Repeat this process **7** times. The answer is the last name that you retrieve.\
Hint: The first character of the name of the last page that you will load is: I

#### Strategy
The web pages tweak the height between the links and hide the page after a few seconds to make it difficult for you to do the assignment without writing a Python program. But frankly with a little effort and patience you can overcome these attempts to make it a little harder to complete the assignment without writing a Python program. But that is not the point. The point is to write a clever Python program to solve the program.

#### Sample execution

> $ python3 solution.py\
Enter URL: http://py4e-data.dr-chuck.net/known_by_Fikret.html \
Enter count: 4\
Enter position: 3\
Retrieving: http://py4e-data.dr-chuck.net/known_by_Fikret.html \
Retrieving: http://py4e-data.dr-chuck.net/known_by_Montgomery.html \
Retrieving: http://py4e-data.dr-chuck.net/known_by_Mhairade.html \
Retrieving: http://py4e-data.dr-chuck.net/known_by_Butchi.html \
Retrieving: http://py4e-data.dr-chuck.net/known_by_Anayah.html \

The answer to the assignment for this execution is "Anayah".

#### Python code:
```javascript
# To run this, download the BeautifulSoup zip file
# http://www.py4e.com/code3/bs4.zip
# and unzip it in the same directory as this file

import urllib.request, urllib.parse, urllib.error
from bs4 import BeautifulSoup
import ssl

# Ignore SSL certificate errors
ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

url = input('Enter - ')
html = urllib.request.urlopen(url, context=ctx).read()
soup = BeautifulSoup(html, 'html.parser')

position = int(input('position: '))
count = int(input('count: '))

for i in range(count):
    html = urllib.request.urlopen(url, context=ctx).read()
    soup = BeautifulSoup(html, 'html.parser')
    tags = soup('a')
    num = 0

    for tag in tags:
        num = num +1

        if num > position:
            break
        url = tag.get('href', None)
        name = tag.contents[0]

print(name)
```

#### Name: Ismaeel 

# Extracting Data from XML

In this assignment you will write a Python program somewhat similar to http://www.py4e.com/code3/geoxml.py. The program will prompt for a URL, read the XML data from that URL using **urllib** and then parse and extract the comment counts from the XML data, compute the sum of the numbers in the file.

We provide two files for this assignment. One is a sample file where we give you the sum for your testing and the other is the actual data you need to process for the assignment.

>Sample data: http://py4e-data.dr-chuck.net/comments_42.xml (Sum=2553)\
Actual data: http://py4e-data.dr-chuck.net/comments_1037118.xml (Sum ends with 32)

You do not need to save these files to your folder since your program will read the data directly from the URL. Note: Each student will have a distinct data url for the assignment - so only use your own data url for analysis.

#### Data Format and Approach
The data consists of a number of names and comment counts in XML as follows:

><comment>\
  <name>Matthias</name>\
  <count>97</count>\
</comment>

You are to look through all the <comment> tags and find the <count> values sum the numbers. The closest sample code that shows how to parse XML is geoxml.py. But since the nesting of the elements in our data is different than the data we are parsing in that sample code you will have to make real changes to the code.
 
To make the code a little simpler, you can use an XPath selector string to look through the entire tree of XML for any tag named 'count' with the following line of code:

>counts = tree.findall('.//count')

Take a look at the Python ElementTree documentation and look for the supported XPath syntax for details. You could also work from the top of the XML down to the comments node and then loop through the child nodes of the comments node.

#### Sample Execution

>$ python3 solution.py\
Enter location: http://py4e-data.dr-chuck.net/comments_42.xml \
Retrieving http://py4e-data.dr-chuck.net/comments_42.xml \
Retrieved 4189 characters\
Count: 50\
Sum: 2...

#### Python code:
```javascript
import urllib.request, urllib.parse, urllib.error
import xml.etree.ElementTree as ET
import ssl

# Ignore SSL certificate errors
ctx = ssl.create_default_context()
ctx.check_hostname = False
ctx.verify_mode = ssl.CERT_NONE

api_key = False
# If you have a Google Places API key, enter it here
# api_key = 'AIzaSy___IDByT70'
# https://developers.google.com/maps/documentation/geocoding/intro

if api_key is False:
    api_key = 42
    serviceurl = 'http://py4e-data.dr-chuck.net/xml?'
else :
    serviceurl = 'https://maps.googleapis.com/maps/api/geocode/xml?'

address = input('Enter location: ')
source = urllib.request.urlopen(address)

data = source.read()
#print('Retrieved', len(data), 'characters')
#print(data.decode())
tree = ET.fromstring(data)

counts = tree.findall('.//count')
list = [int(count.text) for count in counts]

print(sum(list))

```
#### Sum: 2232

# Extracting Data from JSON

In this assignment you will write a Python program somewhat similar to http://www.py4e.com/code3/json2.py. The program will prompt for a URL, read the JSON data from that URL using **urllib** and then parse and extract the comment counts from the JSON data, compute the sum of the numbers in the file and enter the sum below:
We provide two files for this assignment. One is a sample file where we give you the sum for your testing and the other is the actual data you need to process for the assignment.

>Sample data: http://py4e-data.dr-chuck.net/comments_42.json (Sum=2553)\
Actual data: http://py4e-data.dr-chuck.net/comments_1037119.json (Sum ends with 59)

You do not need to save these files to your folder since your program will read the data directly from the URL. Note: Each student will have a distinct data url for the assignment - so only use your own data url for analysis.

#### Data Format
The data consists of a number of names and comment counts in JSON as follows:

>{\
  comments: [\
    {\
      name: "Matthias"\
      count: 97\
    },\
    {\
      name: "Geomer"\
      count: 97\
    }\
    ...\
  ]\
}

The closest sample code that shows how to parse JSON and extract a list is json2.py. You might also want to look at geoxml.py to see how to prompt for a URL and retrieve data from a URL.

#### Sample Execution

>$ python3 solution.py\
Enter location: http://py4e-data.dr-chuck.net/comments_42.json \
Retrieving http://py4e-data.dr-chuck.net/comments_42.json \
Retrieved 2733 characters\
Count: 50\
Sum: 2...

#### Python code:
```javascript


```
