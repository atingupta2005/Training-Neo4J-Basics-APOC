# Introduction to Python with Neo4J

## Install Python
- [Download Python](https://www.python.org/ftp/python/3.9.5/python-3.9.5-amd64.exe)

## Create Python Virtual Environment

```
mkdir c:\my-neo4j-project
cd c:\my-neo4j-project
wget https://bootstrap.pypa.io/get-pip.py  -outfile "get-pip.py"
python get-pip.py
pip install virtualenv
virtualenv venv
cmd
.\venv\Scripts\activate.bat
```


### Install Libraries
```
pip install -r requirements.txt
```

### Run Jupyter Notebook
```
git clone https://github.com/atingupta2005/Training-Neo4J-Basics-APOC
cd Training-Neo4J-Basics-APOC
jupyter notebook --ip 0.0.0.0
```
