# Runner Interaction

This document lists a set of APIs that are used for runner-orchestrator interaction.

## Register runner
Description: This API is used to manually register the runner to the orchestrator. A UUID token is needed to register the runner. Currently, no auth function is provided so this token can be any UUID.
#### URL
POST /api/runner/registration
#### Request
Request body
```json
{
  "token": "tooooooken",
  "framework": {
    "name": "Stryker",
    "version": "<string>",
    "branding": {
      "homepageUrl": "<string>",
      "imageUrl": "<string>"
    },
    "dependencies": {
      "name": "node",
      "version": "17.0.1"
    },
    "system": {
      "ci": true,
      "os": {
        "description": "Windows 10 Pro",
        "platform": "linux",
        "version": "10.0.19041"
      },
      "cpu": {
        "baseClock": 18,
        "logicalCores": 2154,
        "model": "Intel(R) Core(TM) i9-9880H CPU @ 2.30GHz"
      },
      "ram": {
        "total": 66
      }
    }
  }
}
```

####  Response
Status: 201 CREATED

Description: The runner is registered successfully.

Response body:

```json
{
  "runner_id": "uuid string"
}
```


## Deregister runner
Description: The API is used to manually de-register the runner. The runner ID (UUID format) should be provided. The runner ID will be added to a blacklist. After de-registration, any following requests from this runner will be refused.
#### URL
DELETE /api/runner/registration
#### Request
Request body
```json
{
  "id": "uuid of runner"
}
```
#### Response
Status: 200 OK
Response body: (empty)

## Heartbeat
Description: The runner will regularly send a heartbeat request including the latest runner status. All the commands from the orchestrator to the runner should be included in this heartbeat response. This mechanism is designed in case the runner is in an internal network or behind a proxy or firewall.
#### URL
POST /api/runner/heartbeat
#### Request

Description: Currently, the supporting framework and dependencies are included in the heartbeat request. The information provided in the request will be saved in orchestrator 

Request body:
```json
{
  "id": "uuid of runner",
  "status": "busy | ready",
  "framework": {
    "name": "StrykerJS",
    "version": "1.0.0",
    "branding": {
      "homepageUrl": "Optional",
      "imageUrl": "Optional"
    },
    "dependencies": [
      {"dependency1": "1.0.0"},
      {"dependency2": "1.2.0"}
    ]
  }
}
```
#### Response
Description: If the orchestrator is ok and the runner is ready, then the orchestrator may give a mutant to the runner. If there is no job for this runner, the orchestrator will only return an ok.

Status: 200 OK

```json
{
  "heartbeat": {
    "status": "ok",
    "mutant": {
      "uuid": "uuid of the project",
      "messageId": "uuid of the message",
      "filename": "task.ts",
      "projectFiles": "url of the project",
      "mutant": {
        "id": "id of the mutant",
        "mutatorName": "name of the mutation operator",
        "status": "Killed | Survived | NoCoverage | CompileError | RuntimeError | Timeout | Ignored",
        "location": {
          "start": {
            "column": 1,
            "line": 2
          },
          "end": {
            "column": 3,
            "line": 4
          }
        }
      }
    }
  }
}
```

Description: If the runner is on the blacklist (e.g. has been deregistered from orchestrator)

Status: 403 FORBIDDEN

```json
{
    "statusCode": 403,
    "message": "Forbidden resource",
    "error": "Forbidden"
}
```

## Upload result

Description: This API is used for the runner to upload the execution result of a mutant.

#### URL
complete/{runnerUUID}/{projectUUID}
#### Request

Description: After receiving a mutant from the heartbeat request, the runner will try to execute the mutant. After execution, the mutant id, and execution status and the position of the mutant will be given back to the orchestrator.

Path variables:
- runnerUUID: uuid of the runner
- projectUUID: uuid of the project

Request body:
```json
{
  "filename": "task.ts",
  "messageId": "id of the message",
  "result": {
    "id": "id of the mutant",
    "mutatorName": "name of the mutation operator",
    "status": "Killed | Survived | NoCoverage | CompileError | RuntimeError | Timeout | Ignored",
    "location": {
      "start": {
        "column": 1,
        "line": 2
      },
      "end": {
        "column": 3,
        "line": 4
      }
    }
  }
}
```

#### Response
Status: 200 OK
Response body

```json
{
  "status": "OK"
}
```