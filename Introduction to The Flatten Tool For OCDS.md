# Introduction to The Flatten Tool For OCDS
Recently I was playing around with  some open contracting data which was my first time working with data in json format. If you have ever had to work with open contracting data or generally any multi-deeply nested json file, you’ll relate to my trauma in trying to flatten out the file using pandas, json and json_normalize in jupyter notebook. Even with that I still did not get my desired output. I had to further write a number of loops and functions. For a beginner to intermediate level data tinkerer like myself, this experience can get very frustrating. Csv files have been the norm of most data projects I’ve worked on. On few occasions did I deal with spreadsheets which was pretty easy to convert to csv format on the click of a button or command in the terminal.

CSV formatted files are flat i.e. the data has rows and columns just like a table so it’s easier to sort, filter or group. However, data in JSON format is based on a key/value pair relationship. And it gets much complicated as the level of nesting goes deep. Think of it as a tree with many branches where each branch has multiple branches and these multiple branches also have branches which also have branches and so on. To be able to do some data wrangling we need to flatten out all of these branches into a straight long branch. 

Don’t even think about it! Trying to totally flatten out the deep nests into a table format would require multiple lines of code I doubt you’re prepared to write when you can’t wait to get your hands dirty on some data.

As most of the data collected from or published to the web is in json format, it is inevitable you will come across a json file to work with in the course of your data journey. JSON files have their tremendous use cases but data analysis is definitely not one of them.

The Flatten tool saves you all the time and effort it takes in writing those multiple lines of code to convert json formatted data into csv’s and vice versa.  

## Prerequisites
Before you start this tutorial, I assume that
- You know a thing or two about the command line/terminal and have probably written a few commands
- You have worked with open contracting data or any data in either one of the formats ie json or csv
- You have Jupyter Notebook installed
- You know some python and are familiar with using its libraries working with data

## Environment Setup/Downloading The Tool
My environment set up was done in Ubuntu 18.04 hence the steps below might not apply in other versions or operating systems. The flatten tool runs on python 3.5 or later and a virtual environment. Virtual environments allow you to create and manage multiple projects with each relying on different versions of python to execute.

First, check your python version  by running ```python -V``` in your terminal. If you’re not using Python 3, to create a virtual environment and install the flatten tool run the following:  

```sudo pip3 install virtualenv```  
```cd ~```  
```mkdir Virtualenvs```  
```cd Virtualenvs```  
```git clone https://github.com/OpenDataServices/flatten-tool.git```  
```cd flatten-tool```  
```virtualenv flatten_env```  
```source flatten_env/bin/activate```  
```pip install -r requirements_dev.txt```  


If you are using Python 3, then you should already have the venv module from the standard library installed. Run the following:  

```git clone https://github.com/OpenDataServices/flatten-tool.git```  
```cd flatten-tool```  
```python3 -m venv .ve```  
```source .ve/bin/activate```  
```pip install -r requirements_dev.txt```  

## Converting from OCDS JSON to OCDS CSV/Spreadsheet

For this tutorial, we’ll use a sample [open contracting json file](https://github.com/open-contracting/sample-data/blob/master/fictional-example/1.0/ocds-213czf-000-00001-04-award.json)

Our quest will be to flatten out this json format and identify:
- To whom the award was given?
- For which item (good/work/service)?
- And for how much ?

We will start our quest by using pandas & the json library in jupyter notebook. After which we will explore the flatten tool. This approach will help us identify the limitations of the json library and why the flatten tool is much efficient. 

### 1. Using Pandas & JSON library

First, import relevant libraries 

```import pandas as pd```  
```import json```  
```from pandas.io.json import json_normalize```


Let's read-in our json file 

```with open('./data/sample-ocds-award-data.json') as data:  
      ocds_award = json.load(data)
```


Print out the main keys of the json file to know which key has the data we need. With ocds data, details are contained in 'releases'  

```ocds_award.keys()```


Flatten out "releases” and print out the head. This gives us some data but looks like what we need is hidden in the awards column as a list of dictionary.  

```all_releases = json_normalize(ocds_award['releases'])```  
```pd.DataFrame(all_releases).head()```  


Again, flatten out awards column to a new dataframe. Yet still, we are not able to find answers to our questions.

```award_releases = json_normalize(ocds_award, 'releases', ['awards'], errors='ignore', record_prefix='awards/')```  
```pd.DataFrame(award_releases).head()```  


Let’s see what awards/awards contains. 

```award_details = pd.DataFrame(award_releases['awards/awards'])```  
```award_details```  


And unpack it  

``` 
def unpack(award_details):
  award_details_unpacked = []
  for i in award_details:
    if type(i) != list:
      award_details_unpacked.append(i)
    else:
      award_details_unpacked = award_details_unpacked + unpack(i)
  return award_details_unpacked
  
 award_details_unpacked = {}
 for k, v in award_details.items():
    award_details_unpacked[k] = unpack(v)
```  


Alas! There lies the answers we seek. But how do we flatten this into a dataframe?  

```
award_details_unpacked
```  


See how cumbersome this process is getting? For a simple json file, going through this process might probably not be a big deal. But with a huge file as deeply nested as this, we’ll have to think twice.



## 2. Using Flatten

In your flatten tool environment, run:  

```flatten-tool -h```  


This prints out how to use the tool plus all the arguments it takes. In our case since we want to convert ocds json to ocds csv, let’s see all the help we can get on flatten:  

```flatten-tool flatten –help```  


From the output we are able to view all the arguments of flatten. Don;t worry if everything doesn’t make sense. As we determine what we need, we will understand which arguments can help us achieve our aim. 

To convert our json file to csv, we run the command below:  

```flatten-tool flatten ocds-award.json -f=csv --root-id=ocid --main-sheet-name releases --root-list-path=releases -o=ocds-award.csv```  


Command Breakdown:  
```flatten```: specifies that we want to flatten a file  

```ocds-award.json```: name of the file to be flattened  

```-f=csv```: format of the output file we want  

```--root-id=ocid```: the root id of our file  

```--main-sheet-name releases```: name of the key with the data we need  

```-o=ocds-award```: name of output folder  


In your flatten tool folder, you’ll find the folder “ocds-award” with multiple csv files. We have these number of csv files because, the flatten tool creates a new csv file for every key that has a list as a value in the json file.

We can then go ahead to merge the multiple files into a dataframe in Jupyter Notebook.

First, import the libraries we need  

```import pandas as pd```  
```import glob```  


Concatenate all the csv files into a dataframe and print out the head  

```ocds_award_csv = [pd.read_csv(filename) for filename in glob.glob('./data/flattened_json(csv)/*.csv')]```  
```ocds_award = pd.concat(ocds_award_csv, axis=1, sort='False')```  

```ocds_award.head()```  


List all the column heads to get a sense of what data we’re working with  

```list(ocds_award)```  


Now we have an idea as to which columns contain the answers we seek.

To find details of supplier information i.e. to whom the award was given, we run the following code 

```ocds_award_supplier = [col for col in ocds_award if col.startswith('awards/0/suppliers/')]```  
```ocds_award[ocds_award_supplier]```  


For items awarded;  

```ocds_award_items = [col for col in ocds_award if col.startswith('awards/0/items/')]```  
```ocds_award[ocds_award_items]```  


Lastly, the value of the award  

```ocds_award_value = [col for col in ocds_award if col.startswith('awards/0/value/')]```  
```ocds_award[ocds_award_value]```  


Easy peasy!

## Converting From OCDS CSV/Spreadsheet to OCDS JSON
Now, let’s convert back our files from csv to json. Using the flatten tool, we can pass the argument “unflatten.”  Run this code in your flatten environment on your terminal:  

```flatten-tool unflatten ocds-award-csv --root-id=ocid --input-format csv --output-name ocds_award.json –root-list-path=releases```  


By now, the arguments in the command should be familiar to you.

Wait, don’t get too excited yet. You’ll realize that, this new json file does not contain the metadata that the original ocds json file includes. To rectify this you need to create a base json file that will be included in unflattening our csv files.   


Find the Flatten tool cool? Learn more here: https://flatten-tool.readthedocs.io/en/latest/
