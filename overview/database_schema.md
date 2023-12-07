# <ins>Database Schema</ins>

NOTE: this is a WIP, check back soon!

TODO: Write the schema here using some sort of a useful visual!

## 1. Overview of Psyanim Database Schema

Important Notes to cover: 

- psyanim-2's database is a NoSQL document database that can exist locally as flat files, or in any cloud-based NoSQL database service (e.g. firestore)

    - by default, we only support firestore, but could add support for other NoSQL database services

- 5 top-level collections in database:

    - `trial-metadata`
    - `animation-clips`
    - `state-logs`
    - `session-logs`
    - `jspsych-experiment-data`

## 2. `trial-metadata` collection

```json
{
    "data": {
        "sceneKey": "Wander Test",
        "agentMetadata": [
            {
                "animationClipId": "4b2bc253-c434-4b8c-8aab-00f92eec36b9",
                "name": "agent",
                "initialPosition": {
                    "x": 741.4686846915806,
                    "y": 519.5448651348128
                },
                "matterOptions": {
                    "name": "agent",
                    "isSleeping": false,
                    "isSensor": false,
                    "collisionFilter": {
                        "category": 1,
                        "mask": 13
                    }
                },
                "shapeParams": {
                    "shapeType": "PSYANIM_SHAPE_TRIANGLE",
                    "altitude": 32,
                    "color": 16761035,
                    "textureKey": "wanderTestTexture",
                    "base": 16
                }
            }
        ],
        "sceneParameters": [
            {
                "parameterType": "scene",
                "parameterName": "canvasSize",
                "parameterValue": {
                    "width": 800,
                    "height": 600
                }
            },
            {
                "parameterType": "scene",
                "parameterName": "backgroundColor",
                "parameterValue": "#ffffff"
            },
            {
                "parameterType": "scene",
                "parameterName": "wrapScreenBoundary",
                "parameterValue": true
            }
        ],
        "trialNumber": 0,
        "sessionID": "bea6e389-5ae5-47f4-9235-4b221d8ff7f3",
        "userID": "Jason",
        "experimentName": "defaultExperimentName"
    },
    "time": {
        "_seconds": 1701914763,
        "_nanoseconds": 233000000
    }
}
```

## 3. `animation-clips` collection

```json
{
    "data": [
        {
            "t": 0,
            "rotation": 0,
            "x": 741.4686846915806,
            "y": 519.5448651348128
        },
        {
            "t": 28,
            "rotation": -0.3394931361149034,
            "x": 742.9471888602887,
            "y": 519.0227067075969
        },
        {
            "t": 44.29999999701977,
            "rotation": -0.3346741390403061,
            "x": 744.3460539036649,
            "y": 518.5410051854006
        }
    ],
    "lastTimeModified": {
        "_seconds": 1701914763,
        "_nanoseconds": 115000000
    }
}
```

## 4. `session-logs` collection

```json
{
    "data": [
        {
            "msg": [
                "load scene called with key: Wander Test"
            ],
            "type": "log",
            "clientTime": 1701914760896
        },
        {
            "msg": [
                "New wander scene created, logging state for agent: ",
                "agent"
            ],
            "type": "log",
            "clientTime": 1701914760930
        },
        {
            "msg": [
                "_angle = ",
                304.4683669002278
            ],
            "type": "log",
            "clientTime": 1701914761924
        },
        {
            "msg": [
                "_angle = ",
                363.9969997706003
            ],
            "type": "log",
            "clientTime": 1701914762944
        },
        {
            "msg": [
                "_angle = ",
                532.0378947698973
            ],
            "type": "log",
            "clientTime": 1701914763946
        },
        {
            "msg": [
                "load scene called with key: Wander Test_2"
            ],
            "type": "log",
            "clientTime": 1701914771227
        },
        {
            "msg": [
                "New wander scene created, logging state for agent: ",
                "agent_1"
            ],
            "type": "log",
            "clientTime": 1701914771256
        },
        {
            "msg": [
                "_angle = ",
                445.7914669163267
            ],
            "type": "log",
            "clientTime": 1701914772219
        },
        {
            "msg": [
                "_angle = ",
                552.2095983980269
            ],
            "type": "log",
            "clientTime": 1701914773220
        }
    ],
    "time": {
        "_seconds": 1701914780,
        "_nanoseconds": 159000000
    }
}
```

## 5. `state-logs` collection

Example JSON coming soon...

## 6. `jspsych-experiment-data` collection

```json
{
    "data": {
        "jsPsychData": "[{\"rt\":424,\"stimulus\":\"<p style='text-align:center'>Welcome to the experiment.  Press any key to begin.</p>\",\"response\":\"enter\",\"trial_type\":\"html-keyboard-response\",\"trial_index\":0,\"time_elapsed\":425,\"internal_node_id\":\"0.0-0.0\"},{\"trial_type\":\"psyanim-jsPsych-plugin\",\"trial_index\":1,\"time_elapsed\":3698,\"internal_node_id\":\"0.0-1.0\"},{\"trial_type\":\"psyanim-jsPsych-plugin\",\"trial_index\":2,\"time_elapsed\":7188,\"internal_node_id\":\"0.0-2.0\"},{\"trial_type\":\"psyanim-jsPsych-plugin\",\"trial_index\":3,\"time_elapsed\":10759,\"internal_node_id\":\"0.0-3.0\"},{\"trial_type\":\"psyanim-jsPsych-plugin\",\"trial_index\":4,\"time_elapsed\":15287,\"internal_node_id\":\"0.0-4.0\"},{\"rt\":863,\"stimulus\":\"<p style='text-align:center'>Congrats - you have completed your first experiment!  Press any key to end this trial.</p>\",\"response\":\"enter\",\"trial_type\":\"html-keyboard-response\",\"trial_index\":5,\"time_elapsed\":16151,\"internal_node_id\":\"0.0-5.0\"}]",
        "sessionID": "bea6e389-5ae5-47f4-9235-4b221d8ff7f3",
        "userID": "Jason",
        "experimentName": "defaultExperimentName"
    },
    "time": {
        "_seconds": 1701914776,
        "_nanoseconds": 880000000
    }
}
```