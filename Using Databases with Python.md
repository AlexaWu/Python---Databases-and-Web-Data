# Counting Organizations

This application will read the mailbox data (mbox.txt) and count the number of email messages per organization (i.e. domain name of the email address) using a database with the following schema to maintain the counts.

>CREATE TABLE Counts (org TEXT, count INTEGER)

Run the program on **mbox.txt** (http://www.py4e.com/code3/mbox.txt).

Because the sample code is using an **UPDATE** statement and committing the results to the database as each record is read in the loop, it might take as long as a few minutes to process all the data. The commit insists on completely writing all the data to disk every time it is called.

*The program can be speeded up greatly by moving the commit operation outside of the loop. In any database program, there is a balance between the number of operations you execute between commits and the importance of not losing the results of operations that have not yet been committed.*

### Python code
```Javascript
import sqlite3

conn = sqlite3.connect('emaildb.sqlite')
cur = conn.cursor()

cur.execute('DROP TABLE IF EXISTS Counts')

cur.execute('''
CREATE TABLE Counts (org TEXT, count INTEGER)''')

fname = input('Enter file name: ')
if (len(fname) < 1): fname = 'mbox-short.txt'
fh = open(fname)

for line in fh:
    if not line.startswith('From: '): continue
    pieces = line.split()
    email = pieces[1]
    (emailname, org) = email.split("@")

    cur.execute('SELECT count FROM Counts WHERE org = ? ', (org,))
    row = cur.fetchone()

    if row is None:
        cur.execute('''INSERT INTO Counts (org, count)
                VALUES (?, 1)''', (org,))
    else:
        cur.execute('UPDATE Counts SET count = count + 1 WHERE org = ?',
                    (org,))
    conn.commit()

# https://www.sqlite.org/lang_select.html
sqlstr = 'SELECT org, count FROM Counts ORDER BY count DESC LIMIT 10'

for row in cur.execute(sqlstr):
    print(str(row[0]), row[1])

cur.close()
```

### SQLite
![](https://github.com/AlexaWu/Python/blob/main/screenshots/emaildb.PNG)

---

# Multi-Table Database - Tracks

_**Musical Track Database**_

This application will read an iTunes export file in XML and produce a properly normalized database.

If you run the program multiple times in testing or with different files, make sure to empty out the data before each run.

- You can use this code for the application: http://www.py4e.com/code3/tracks.zip. The ZIP file contains the Library.xml file to be used.

- To export your own Library.xml from iTunes: File -> Library -> Export Library. Make sure it is in the correct folder. iTUnes might change UI and/or export format any time.
  
### Python code
```Javascript
import xml.etree.ElementTree as ET
import sqlite3

conn = sqlite3.connect('trackdb.sqlite')
cur = conn.cursor()

# Make some fresh tables using executescript()
cur.executescript('''
DROP TABLE IF EXISTS Artist;
DROP TABLE IF EXISTS Genre;
DROP TABLE IF EXISTS Album;
DROP TABLE IF EXISTS Track;

CREATE TABLE Artist (
    id  INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
    name    TEXT UNIQUE
);

CREATE TABLE Genre(
    id INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
    name    TEXT UNIQUE
);

CREATE TABLE Album (
    id  INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
    artist_id  INTEGER,
    title   TEXT UNIQUE
);

CREATE TABLE Track (
    id  INTEGER NOT NULL PRIMARY KEY
        AUTOINCREMENT UNIQUE,
    title TEXT  UNIQUE,
    album_id  INTEGER,
    genre_id  INTEGER,
    len INTEGER, rating INTEGER, count INTEGER
);
''')


fname = input('Enter file name: ')
if ( len(fname) < 1 ) : fname = 'Library.xml'

# <key>Track ID</key><integer>369</integer>
# <key>Name</key><string>Another One Bites The Dust</string>
# <key>Artist</key><string>Queen</string>
def lookup(d, key):
    found = False

    for child in d:
        if found : return child.text
        if child.tag == 'key' and child.text == key :
            found = True
    return None

stuff = ET.parse(fname)
all = stuff.findall('dict/dict/dict')
print('Dict count:', len(all))

for entry in all:
    if ( lookup(entry, 'Track ID') is None ) : continue

    name = lookup(entry, 'Name')
    artist = lookup(entry, 'Artist')
    genre = lookup(entry, 'Genre')
    album = lookup(entry, 'Album')
    count = lookup(entry, 'Play Count')
    rating = lookup(entry, 'Rating')
    length = lookup(entry, 'Total Time')

    if name is None or artist is None or album is None or genre is None:
        continue

    print(name, artist, genre, album, count, rating, length)

    cur.execute('''INSERT OR IGNORE INTO Artist (name)
        VALUES ( ? )''', ( artist, ) )
    cur.execute('SELECT id FROM Artist WHERE name = ? ', (artist, ))
    artist_id = cur.fetchone()[0]

    cur.execute('''INSERT OR IGNORE INTO Genre (name)
        VALUES ( ? )''', ( genre, ) )
    cur.execute('SELECT id FROM Genre WHERE name = ? ', (genre, ))
    genre_id = cur.fetchone()[0]

    cur.execute('''INSERT OR IGNORE INTO Album (title, artist_id)
        VALUES ( ?, ? )''', ( album, artist_id ) )
    cur.execute('SELECT id FROM Album WHERE title = ? ', (album, ))
    album_id = cur.fetchone()[0]

    cur.execute('''INSERT OR REPLACE INTO Track
        (title, album_id, genre_id, len, rating, count)
        VALUES ( ?, ?, ?, ?, ?, ? )''',
        ( name, album_id, genre_id, length, rating, count ) )

    conn.commit()
```

### SQLite
![](https://github.com/AlexaWu/Python/blob/main/screenshots/tracks-browse.PNG)

### Test trackdb.sqlite
```Javascript
SELECT Track.title, Artist.name, Album.title, Genre.name 
    FROM Track JOIN Genre JOIN Album JOIN Artist 
    ON Track.genre_id = Genre.ID and Track.album_id = Album.id 
        AND Album.artist_id = Artist.id
    ORDER BY Artist.name LIMIT 3
```    
![](https://github.com/AlexaWu/Python/blob/main/screenshots/tracks.PNG)

---

# Many Students in Many Courses

_**Instructions**_

This application will read roster data in JSON format, parse the file, and then produce an SQLite database that contains a User, Course, and Member table and populate the tables from the data file.

Download this file (https://www.py4e.com/tools/sql-intro/roster_data.php?PHPSESSID=2f6cf52fc6bce10a6969a93a55f9cecf) and save it as roster_data.json. Move the downloaded file into the **same folder** as the below roster.py program.

Read the above JSON data.

### Python code
```Javascript
import json
import sqlite3

conn = sqlite3.connect('rosterdb.sqlite')
cur = conn.cursor()

# Do some setup
cur.executescript('''
DROP TABLE IF EXISTS User;
DROP TABLE IF EXISTS Member;
DROP TABLE IF EXISTS Course;

CREATE TABLE User (
    id     INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
    name   TEXT UNIQUE
);

CREATE TABLE Course (
    id     INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT UNIQUE,
    title  TEXT UNIQUE
);

CREATE TABLE Member (
    user_id     INTEGER,
    course_id   INTEGER,
    role        INTEGER,
    PRIMARY KEY (user_id, course_id)
)
''')

fname = input('Enter file name: ')
if len(fname) < 1:
    fname = 'roster_data_sample.json'

# [
#   [ "Charley", "si110", 1 ],
#   [ "Mea", "si110", 0 ],

str_data = open(fname).read()
json_data = json.loads(str_data)

for entry in json_data:

    name = entry[0];
    title = entry[1];
    role_id = entry[2];

    print((name, title))

    cur.execute('''INSERT OR IGNORE INTO User (name)
        VALUES ( ? )''', ( name, ) )
    cur.execute('SELECT id FROM User WHERE name = ? ', (name, ))
    user_id = cur.fetchone()[0]

    cur.execute('''INSERT OR IGNORE INTO Course (title)
        VALUES ( ? )''', ( title, ) )
    cur.execute('SELECT id FROM Course WHERE title = ? ', (title, ))
    course_id = cur.fetchone()[0]

    cur.execute('''INSERT OR REPLACE INTO Member
        (user_id, course_id, role) VALUES ( ?, ? ,? )''',
        ( user_id, course_id, role_id) )

    conn.commit()
```
### Test trackdb.sqlite

Run the following SQL command:
```Javascript
SELECT User.name,Course.title, Member.role FROM 
    User JOIN Member JOIN Course 
    ON User.id = Member.user_id AND Member.course_id = Course.id
    ORDER BY User.name DESC, Course.title DESC, Member.role DESC LIMIT 2;
```    
The output should look as follows:
>Zuriel|si110|0\
Zi|si206|0

Once that query gives the correct data, run this query:
```Javascript
SELECT 'XYZZY' || hex(User.name || Course.title || Member.role ) AS X FROM 
    User JOIN Member JOIN Course 
    ON User.id = Member.user_id AND Member.course_id = Course.id
    ORDER BY X LIMIT 1;
```
Then you could get one row with a string: XYZZY4161646974736933363330

### SQLite
![](https://github.com/AlexaWu/Python/blob/main/screenshots/rosterdb.PNG)

---

# Databases and Visualization (Google GeoCoding API & Google Maps)

Use the Google GeoCoding API to retrieve data and then use Google Maps to visualize the data.

![](https://github.com/AlexaWu/Python/blob/main/screenshots/Databases%20and%20Visualization%20flow%20chart.PNG)

Using the Google geocoding API to clean up some user-entered geographic locations of university names and then placing the data on a Google Map.

_**Retrieving GEOData**_

Download the code from http://www.py4e.com/code3/geodata.zip - then unzip the file.

Then run the geoload.py to lookup all of the entries in where.data and produce the geodata.sqlite. 

Then run geodump.py to read the database and produce where.js.

Run the programs. Then open where.html to visualize the map.

_**More explanation about the codes**_

Note: Windows has difficulty in displaying UTF-8 characters in the console so for each command window you open, you may need to type the following command before running this code:

    chcp 65001

http://stackoverflow.com/questions/388490/unicode-characters-in-windows-command-line-how

The first problem to solve is that the Google geocoding API is rate limited to a fixed number of requests per day. So if you have a lot of data you might need to stop and restart the lookup process several times.  

So we break the problem into two phases.

In the first phase we take our input data in the file (where.data) and read it one line at a time, and retrieve the geocoded response and store it in a database (geodata.sqlite).
Before we use the geocoding API, we simply check to see if we already have the data for that particular line of input.

You can re-start the process at any time by removing the file geodata.sqlite

Run the geoload.py program.   This program will read the input lines in where.data and for each line check to see if it is already in the database and if we don't have the data for the location, call the geocoding API to retrieve the data and store it in the database.

As of December 2016, the Google Geocoding APIs changed dramatically. They moved some functionality that we use from the Geocoding API into the Places API.  Also all the Google Geo-related APIs require an API key. To complete this assignment without a Google account, without an API key, or from a country that blocks access to Google, you can use a subset of that data which is available at:

    http://py4e-data.dr-chuck.net/json

To use this, simply leave the api_key set to False in geoload.py. This URL only has a subset of the data but it has no rate limit so it is good for testing.

If you want to try this with the API key, follow the instructions at:

    https://developers.google.com/maps/documentation/geocoding/intro

and put the API key in the code.

_**sample run**_

Here is a sample run after there is already some data in the database:
```javascript
Mac: python3 geoload.py
Win: geoload.py

Found in database  Northeastern University

Found in database  University of Hong Kong, Illinois Institute of Technology, Bradley University

Found in database  Technion

Found in database  Viswakarma Institute, Pune, India

Found in database  UMD

Found in database  Tufts University

Resolving Monash University
Retrieving http://py4e-data.dr-chuck.net/json?key=42&address=Monash+University
Retrieved 2063 characters {    "results" : [
{u'status': u'OK', u'results': ... }

Resolving Kokshetau Institute of Economics and Management
Retrieving http://py4e-data.dr-chuck.net/json?key=42&address=Kokshetau+Institute+of+Economics+and+Management
Retrieved 1749 characters {    "results" : [
{u'status': u'OK', u'results': ... }
```
The first five locations are already in the database and so they are skipped.  The program scans to the point where it finds un-retrieved locations and starts retrieving them.

The geoload.py can be stopped at any time, and there is a counter that you can use to limit the number of calls to the geocoding API for each run.

Once you have some data loaded into geodata.sqlite, you can visualize the data using the (geodump.py) program.  This program reads the database and writes tile file (where.js)
with the location, latitude, and longitude in the form of executable JavaScript code.

A run of the geodump.py program is as follows:
```javascript
Mac: python3 geodump.py
Win: geodump.py

Northeastern University, 360 Huntington Avenue, Boston, MA 02115, USA 42.3396998 -71.08975
Bradley University, 1501 West Bradley Avenue, Peoria, IL 61625, USA 40.6963857 -89.6160811
...
Technion, Viazman 87, Kesalsaba, 32000, Israel 32.7775 35.0216667
Monash University Clayton Campus, Wellington Road, Clayton VIC 3800, Australia -37.9152113 145.134682
Kokshetau, Kazakhstan 53.2833333 69.3833333
...
12 records written to where.js
Open where.html to view the data in a browser
```
The file (where.html) consists of HTML and JavaScript to visualize a Google Map.  It reads the most recent data in where.js to get the data to be visualized.  

Here is the format of the where.js file:
```javascript
myData = [
[42.3396998,-71.08975, 'Northeastern University, 360 Huntington Avenue, Boston, MA 02115, USA'],
[40.6963857,-89.6160811, 'Bradley University, 1501 West Bradley Avenue, Peoria, IL 61625, USA'],
[32.7775,35.0216667, 'Technion, Viazman 87, Kesalsaba, 32000, Israel'],
   ...
];
```
This is a JavaScript list of lists.  

Simply open where.html in a browser to see the locations.  You can hover over each map pin to find the location that the gecoding API returned for the user-entered input.  If you cannot see any data when you open the where.html file, you might want to check the JavaScript or developer console for your browser.


