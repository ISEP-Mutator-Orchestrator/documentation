# Stryker Agent client interaction

This document describes how one can interact with the [Stryker Agent](https://github.com/ISEP-Mutator-Orchestrator/mutator-orchestrator). In this document the available REST-endpoints are described, along with their intended request and response formats.

## Technical details

For all requests to the orchestrator, except when uploading a file, it is recommended to use JSON, and therefore the `Content-Type: application/json; charset=utf-8` header should be set. By default, the orchestrator will then also reply with JSON objects, whenever data is returned.

## Starting a mutation run

Requesting a mutation run consists of several distinct steps. Firstly, the client should check whether a runner is available for the requested framework. It is also possible to skip this step, however in that case the run will be pending indefinitely (until there are runners for the specific framework available). Then, the client should upload the information regarding the mutant run, such as the mutants, to the orchestrator. This will also give the runner the UUID that the orchestrator has assigned to the mutation run. Afterwards, the client can use this UUID to upload the file that should be available to runners to the orchestrator. Only when this file is uploaded will the orchestrator start dividing the client's provided mutants over available runners.

### Checking runner availability

#### `POST /client/check`

The client should specify the name of the desired framework and its desired version. The endpoint will respond with a JSON object containing the key `"result"` with value either `"AVAILABLE"` or `"UNAVAILABLE"`.

##### Example request body

```json
{
  "frameworkInformation": {
    "name": "StrykerJS",
    "version": "6.4.0"
  }
}
```

##### Example response

###### Status code

`200`

###### Body

```json
{
  "result": "AVAILABLE"
}
```

### Uploading mutants

#### `POST /client/mutations`

The request body for this endpoint should be formatted according to the [mutation request schema](https://TODO). This is an OpenAPI schema that specifies what data needs to be posted to the orchestrator and in what format.

##### Example request body

```json
{
  "files": {
    "src/main.ts": {
      "language": "typescript",
      "mutants": [
        {
          "id": "0",
          "mutatorName": "BlockStatement",
          "replacement": "{}",
          "status": "NoCoverage",
          "static": false,
          "killedBy": [],
          "coveredBy": [],
          "location": {
            "end": {
              "column": 2,
              "line": 9
            },
            "start": {
              "column": 28,
              "line": 5
            }
          }
        },
        ...
      ],
      "source": "import { ValidationPipe } from '@nestjs/common';\n ..."
    },
    ...
  },
  "schemaVersion": "1.0",
  "testFiles": {
    "src/modules/client/client.controller.spec.ts": {
      "tests": [
        {
          "id": "0",
          "name": "ClientController should be defined",
          "location": {
            "start": {
              "column": 4,
              "line": 43
            }
          }
        },
        ...
      ],
      "source": "import { Test, TestingModule } from '@nestjs/testing';\n ..."
    },
    ...
  },
  "frameworkInformation": {
    "name": "StrykerJS",
    "version": "6.4.0"
  }
}

```

##### Example response

###### Status code

`200`

###### Body

```json
{
  "uuid": "eb0fc635-ab92-49e0-853a-1a04168df66a",
  "mutants": 148,
  "tests": 16,
  "files": 16,
  "testFiles": 8
}
```

### Uploading file

The client can upload one file that will be made available to runners that pick up the client's project. After uploading this file, the orchestrator will start dividing mutants of the client's project over available runners. The orchestrator will not return any content for this request.

#### `POST /client/file/{uuid}`

Requests to this endpoint should not use JSON, but FormData. Therefore, the `Content-Type` should be set to `multipart/form-data`.

##### Example request body

```
--8c697e738076fcc7cc95612add33a664
Content-Disposition: form-data; name="file"; filename="test.txt"

Lorem ipsum
--8c697e738076fcc7cc95612add33a664--
```

##### Example response

###### Status code

`201`

###### Body

None

## Getting run logs

While a mutation run is running, a client can poll for updates from the orchestrator. These updates can for example be used to display progress to the user.

#### `GET /client/log/{uuid}`

The UUID used in the path parameter should be the UUID that was assigned to the mutation run by the orchestrator while submitting the mutation information.

An optional `lastMessageId` query parameter can be used to let the orchestrator only return log messages that were created after the specified message. Using this the request URL would for example become `https://example.com/client/log/eb0fc635-ab92-49e0-853a-1a04168df66a?lastMessageId=1675435115029-0`. The `lastMessageId` returned by the orchestrator is the id of the last message that was included in the response.

##### Example response

###### Status code

`200`

###### Body

```json
{
  "messages": ["0% (0 / 148)"],
  "lastMessageId": "1675435115029-0"
}
```

## Getting run results

When the run is completed, a client can retrieve the results of the run.

#### `GET /client/result/{uuid}`

The UUID used in the path parameter should be the UUID that was assigned to the mutation run by the orchestrator while submitting the mutation information.

##### Example response

###### Status code

`200`

###### Body

```json
{
  "id": "0",
  "mutatorName": "BlockStatement",
  "replacement": "{}",
  "status": "Killed",
  "static": false,
  "killedBy": [],
  "coveredBy": [],
  "location": {
    "end": {
      "column": 2,
      "line": 9
    },
    "start": {
      "column": 28,
      "line": 5
    }
  }
}
```
