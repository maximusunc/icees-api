[![Build Status](https://travis-ci.com/NCATS-Tangerine/icees-api.svg?branch=master)](https://travis-ci.com/NCATS-Tangerine/icees-api)

## How to run

### Run docker compose

#### Run tests
```
test/test.sh
```

#### schema

edit `config/features.yml`

#### database

for each table create dir with table name under `db/data` and put csv files under that dir

for example, put `patient.csv` under `db/data/patient`

put `visit.csv` under `db/data/visit`

to generate random samples

run
```
python samples.py patient 2010 1000 db/data/patient/patient.csv
```

```
python samples.py visit 2010 1000 db/data/patient/visit.csv
```

#### start services
edit `.env`

run
```
docker-compose up --build
```

### run docker

The following steps can be run using the `redepoly.sh`

#### Build Container

```
docker build . -t icees-api:0.3.0
```

#### Run Container in Standalone Mode (optional)

```
docker run -e ICEES_DBUSER=<dbuser> -e ICEES_DBPASS=<dbpass> -e ICEES_HOST=<host> -e ICEES_PORT=<port> -e ICEES_DATABASE=<database> --rm -v log:/log -p 8080:8080 icees-api:0.3.0
```

```
docker run -e ICEES_DBUSER=<dbuser> -e ICEES_DBPASS=<dbpass> -e ICEES_HOST=<host> -e ICEES_PORT=<port> -e ICEES_DATABASE=<database> --rm -v log:/log --net host icees-api:0.2.0
```

#### Setting up `systemd` (optional)

run docker containers
```
docker run -d -e ICEES_DBUSER=<dbuser> -e ICEES_DBPASS=<dbpass> -e ICEES_HOST=<host> -e ICEES_PORT=<port> -e ICEES_DATABASE=<database> --name icees-api_server -v log:/log -p 8080:8080 icees-api:0.2.0
```

```
docker run -d -e ICEES_DBUSER=<dbuser> -e ICEES_DBPASS=<dbpass> -e ICEES_HOST=<host> -e ICEES_PORT=<port> -e ICEES_DATABASE=<database> --name icees-api_server -v log:/log --net host icees-api:0.3.0
```

```
docker stop icees-api_server
```

copy `<repo>/icees-api-container.service` to `/etc/systemd/system/icees-api-container.service`

start service

```
systemctl start icees-api-container
```

### Run manually
#### Setup environment
set env variables

`ICEES_DBUSER` 

`ICEES_DBPASS`

`ICEES_HOST` 

`ICEES_PORT` 

`ICEES_DATABASE`

`ICEES_DB_POOL_SIZE`

`ICEES_DB_MAX_OVERFLOW`

`ICEES_API_LOG_PATH`

`DB_PATH`

`DATA_PATH`

run
```
pip install -r requirements.txt
```
#### Set up Database

##### Create User

```createuser -P <dbuser>```

enter `<dbpass>` for new user

##### Create Database

```createdb <database>```

##### Create Permissions

```grant all privileges on database <database> to <dbuser>```

##### popluating database

```
python dbutils.py create
```

```
python dbutils.py insert <patient data input> patient
python dbutils.py insert <visit data input> visit
```

#### Run Flask 
run
```
python app.py
```



## REST API

### create cohort
method
```
POST
```

route
```
/(patient|visit)/(2010|2011|2012|2013|2014|2015|2016)/cohort
```
schema
```
{"<feature name>":{"operator":<operator>,"value":<value>},...,"<feature name>":{"operator":<operator>,"value":<value>}}
```

`feature name`: see Kara's spreadsheet

`operator ::= <|>|<=|>=|=|<>`

### get cohort definition
method
```
GET
```

route
```
/(patient|visit)/(2010|2011|2012|2013|2014|2015|2016)/cohort/<cohort id>
```

### get cohort features
method
```
GET
```

route
```
/(patient|visit)/(2010|2011|2012|2013|2014|2015|2016)/cohort/<cohort id>/features
```

### get cohort dictionary
method
```
GET
```

route
```
/(patient|visit)/(2010|2011|2012|2013|2014|2015|2016)/cohort/dictionary
```

### feature association between two features
method
```
POST
```

route
```
/(patient|visit)/(2010|2011|2012|2013|2014|2015|2016)/cohort/<cohort id>/feature_association
```
schema
```
{"feature_a":{"<feature name>":{"operator":<operator>,"value":<value>}},"feauture_b":{"<feature name>":{"operator":<operator>,"value":<value>}}}
```

### feature association between two features using combined bins
method
```
POST
```

route
```
/(patient|visit)/(2010|2011|2012|2013|2014|2015|2016)/cohort/<cohort id>/feature_association2
```
schema
```
{"feature_a":{"<feature name>":[{"operator":<operator>,"value":<value>}]},"feature_b":{"<feature name>":[{"operator":<operator>,"value":<value>}]},"check_coverage_is_full":<boolean>}
```
example
```
{
    "feature_a":{
        "AgeStudyStart":[
            {
                "operator":"=",
                "value":"0-2"
            }, {
                "operator":"between",
                "value_a":"3-17",
                "value_b":"18-34"
            }, {
                "operator":"in", 
                "values":["35-50","51-69"]
            },{
                "operator":"=",
                "value":"70+"
            }
        ]
    },
    "feature_b":{
        "ObesityBMI":[
            {
                "operator":"=",
                "value":0
            }, {
                "operator":"<>", 
                "value":0
            }
        ]
    }
}
```

### associations of one feature to all features
method
```
POST
```

route
```
/(patient|visit)/(2010|2011|2012|2013|2014|2015|2016)/cohort/<cohort id>/associations_to_all_features
```
schema
```
{"feature":{"<feature name>":{"operator":<operator>,"value":<value>}},"maximum_p_value":<maximum p value>}
```
### associations of one feature to all features using combined bins
method
```
POST
```
route
```
/(patient|visit)/(2010|2011|2012|2013|2014|2015|2016)/cohort/<cohort id>/associations_to_all_features2
```
schema
```
{"feature":{"<feature name>":[{"operator":<operator>,"value":<value>}]},"maximum_p_value":<maximum p value>, "check_coverage_is_full":<boolean>}
```
example
```
{
    "feature":{
        "AgeStudyStart":[
            {
                "operator":"=",
                "value":"0-2"
            }, {
                "operator":"between",
                "value_a":"3-17",
                "value_b":"18-34"
            }, {
                "operator":"in", 
                "values":["35-50","51-69"]
            },{
                "operator":"=",
                "value":"70+"
            }
        ]
    },
    "maximum_p_value": 0.1
}
```
### knowledge graph
method
```
POST
```

route
```
/knowledge_graph
```

input parameters:
 * `query_options`
   * `table` : ICEES table
   * `year` : ICEES year
   * `cohort_features`: features for defining the cohort
   * `feature`: a feature and operator and value for spliting the cohort to two subcohorts
   * `maximum_p_value`: ICEES maximum p value. The p value is calculated for each ICEES feature in `table`, using 2 * n contingency table where the rows are subcohorts and the columns are individual values of that feature. Any feature with p value greater than maximum p value is filtered out.
   * `regex`: filter target node name by regex.   
example
```
{
        "query_options": {
            "table": "patient", 
            "year": 2010, 
            "cohort_features": {
                "AgeStudyStart": {
                    "operator": "=",
                    "value": "0-2"
                }
            }, 
            "feature": {
                "EstResidentialDensity": {
                    "operator": "<",
                    "value": 1
                }
            }, 
            "maximum_p_value":1
        }, 
        "machine_question": {
            "nodes": [
                {
                    "id": "n00",
                    "type": "population_of_individual_organisms"
                },
                {
                    "id": "n01",
                    "type": "chemical_substance"
                }   
            ], 
            "edges": [
                {
                    "id": "e00",
                    "type": "association",
                    "source_id": "n00",
                    "target_id": "n01"
                } 
            ]
        }
}
```

## Examples

get cohort of all patients

```
curl -k -XPOST https://localhost:8080/patient/2010/cohort -H "Content-Type: application/json" -H "Accept: application/json" -d '{}'
```

get cohort of patients with `AgeStudyStart = 0-2`

```
curl -k -XPOST https://localhost:8080/patient/2010/cohort -H "Content-Type: application/json" -H "Accept: application/json" -d '{"AgeStudyStart":{"operator":"=","value":"0-2"}}'
```

Assuming we have cohort id `COHORT:10`

get definition of cohort

```
curl -k -XGET https://localhost:8080/patient/2010/cohort/COHORT:10 -H "Accept: application/json"
```

get features of cohort

```
curl -k -XGET https://localhost:8080/patient/2010/cohort/COHORT:10/features -H "Accept: application/json"
```

get cohort dictionary 

```
curl -k -XGET https://localhost:8080/patient/2010/cohort/COHORT:10/features -H "Accept: application/json"
```

get feature association


```
curl -k -XPOST https://localhost:8080/patient/2010/cohort/COHORT:10/feature_association -H "Content-Type: application/json" -d '{"feature_a":{"AgeStudyStart":{"operator":"=", "value":"0-2"}},"feature_b":{"ObesityBMI":{"operator":"=", "value":0}}}'
```

get association to all features


```
curl -k -XPOST https://localhost:8080/patient/2010/cohort/COHORT:10/associations_to_all_features -H "Content-Type: application/json" -d '{"feature":{"AgeStudyStart":{"operator":"=", "value":"0-2"}},"maximum_p_value":0.1}' -H "Accept: application/json"
```

knowledge graph

```
curl -X POST -k "http://localhost:5000/knowledge_graph" -H  "accept: application/json" -H  "Content-Type: application/json" -d '
{
        "query_options": {
            "table": "patient", 
            "year": 2010, 
            "cohort_features": {
                "AgeStudyStart": {
                    "operator": "=",
                    "value": "0-2"
                }
            }, 
            "feature": {
                "EstResidentialDensity": {
                    "operator": "<",
                    "value": 1
                }
            }, 
            "maximum_p_value":1
        }, 
        "machine_question": {
            "nodes": [
                {
                    "id": "n00",
                    "type": "population_of_individual_organisms"
                },
                {
                    "id": "n01",
                    "type": "chemical_substance"
                }   
            ], 
            "edges": [
                {
                    "id": "e00",
                    "type": "association",
                    "source_id": "n00",
                    "target_id": "n01"
                } 
            ]
        }
}
'
```

knowledge graph schema

```
curl -X GET -k "http://localhost:5000/knowledge_graph/schema" -H  "accept: application/json"
```

