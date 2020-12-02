# fw_data_store
A python library for database handling with Create-Read-Delete functions. 

(Kindly note : The unit test for this library is saved as unit_test.ipynb and was done in a python notebook file (.ipynb) which can be accessed through Jupyter Notebooks, Google colab or Github itself. Thank you.)

###### Language written in: Python 
###### OS used: Windows
###### package location : data_store

data_store provides basic database handling functions like, Create,Read and Delete. The data is stored as key-value pairs in a JSON format. 

## Create()

Data is added to the database with the Create() function that inputs a **key,value and a 'Time-To-Live'**, beyond which the key will no longer be available for Read() or Delete()
operations.  Example:

```
dataparser = data_store()

#dataparser.Create(key,value,time-to-live)

dataparser.Create('Dwight',100,60)
```
which creates:
>{'Dwight': {'value': 100, 'ttl': 1606880260}}

'ttl' stores the time at which the data point was created and then adds the user-defined Time-to-Live , to calculate the time beyond which the key is dead. 
If a TTL was not defined by the user, the ttl is set to None and the key will be valid indefinitely. 

```
dataparser = data_store()
dataparser.Create('Dwight',100,20)
dataparser.Create('Jim',500)
```
which saves the data as:

> {'Dwight': {'value': 100, 'ttl': 1606882423},
>'Jim': {'value': 500, 'ttl': None}}
 
 ## Read()
 
 Reading values through a given key is done with the Read(). Read() accepts a given key and returns a value if the key is present. 
 
 ```
 dataparser.Read('Jim')
 ```
 
 Output:
 
 >'500'
 
 ```
 dataparser.Read('Dwight')
 ```
 
 > Exception: Key's Time-To-Live has expired. Unable to read.
 
 ## Delete()
 
 Deleting values is done using Delete(). Delete() accepts a given key and pops a datapoint if the key is present in the database. 
 
```
dataparser.Delete('Jim')
dataparser.data
```
Output:
>Key-value pair deleted

>{'Dwight': {'value': 100, 'ttl': 1606883628}}


## Other Functions:

1) **DisplayAll()**

returns the saved data in the form of key:value pairs. 

```
dataparser = data_store()
dataparser.Create('Dwight',100,20)
dataparser.Create('Jim',500)
dataparser.Create('Michael',1000)

dataparser.DisplayAll()
```
Output:
>{'Dwight': 100, 'Jim': 500, 'Michael': 1000}


2) **ClearAll()**

Clears database.json and the local variables from any saved values. 

```
dataparser.ClearAll()
dataparser.DisplayAll()

```
>Data Cleared

>Exception: Unable to display. Database is empty

## Program Structure

Upon instantiating the class, the datastore is created as 'database.json' in a user-defined address. If no address was provided, it defaults to the current working address. 

```
class data_store:

    def __init__(self, file_path=os.getcwd()):
        self.filepath = file_path + '/database.json'

        self.filelock = threading.Lock()
        self.datalock = threading.Lock()
```
## Packages used

The program utilizes the following standard python libraries:

```
import json, os, sys, time, threading
```

##  Requirements Fulfilled

The program fulfills the following requirements:

1) **Thread-Safety**

The program accommodates multi-threading and provides a thread-safe handling of the files and objects, using threading locks.
The thread locks are acquired and released sequentially before and after every operation. 
```
self.filelock = threading.Lock()
self.datalock = threading.Lock()
```

2) **Single client process access to data store**

The use of file locks permits the use of the file opener to only one client proceess at a time. 


3) **Limited file size**

The file size is limited to < 1GB. This is done through a class methoud checkfilesize()

```
 def checkfilesize(self):

        # Checks if database size exceeds 1 GB size.

        self.filelock.acquire()

        if os.path.getsize(self.filepath) <= 1e+9:
            self.filelock.release()
            return True
        else:
            self.filelock.release()
            return False
```

If the function returns a False value, an error is raised and does not permit adding more data to the database.

## Exceptions Handled 

The program raises an exception and throws a friendly and read-able error message if the package is not used appropriately. 
Some instances where an exception will be raised include:

1) Inputting a key that is not a string, greater than 32 characters in length or is NULL.
2) Inputting a value that is not a JSON Object or is greater than 16 KB.
3) Creating new data when the data file is > 1 GB.
4) Other Corner cases.

A demonstration of the exception handling is done in the unit_test python notebook. JSON format was selected for the database format as it shows faster read/write speeds in comparison to other traditional formats such as pickle and csv.  

