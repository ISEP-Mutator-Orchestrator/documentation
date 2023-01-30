# Runner Interaction
## Register runner
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

Response body:
```json
{
  "runner_id": "uuid string"
}
```


## Deregister runner
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
#### URL
POST /api/runner/heartbeat
#### Request
Request body
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
If the runner is ok 
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

If the runner is on the blacklist (e.g. has been deregistered from orchestrator)  
Status: 403 FORBIDDEN
```json
{
    "statusCode": 403,
    "message": "Forbidden resource",
    "error": "Forbidden"
}
```

## Upload result
#### URL
complete/{runnerUUID}/{projectUUID}
#### Request
Path variables:
- runnerUUID: uuid of the runner
- projectUUID: uuid of the project

Request body
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