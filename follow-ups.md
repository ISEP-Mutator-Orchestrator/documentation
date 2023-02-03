# Follow ups

In this document the next steps for the Mutation Testing Orchestrator will be laid out. They are categorized into the respective folder structure.

## Client

### Cancellation

Implement the cancellation of a mutation testing run.

### Framework matching

The framework matching is slightly different between the Client Service and the Orchestration service. This should be generalized and include more of the options. At the moment only the framework name and version are considered, one might need more specification for the framework selection such as other dependencies or OS specification.

`client.service.ts`

```javascript
async checkrunners(){}
```

### Duplicate mutants in result

Result aggregation does not account for duplicate runs of results and will return all duplicates.

`client.service.ts`

```javascript
async getResult(){}
```

## Orchestration

### Distribution of runners

The Orchestration Service distributes the runners on a first come first serve basis. This might not be the best option for some cases, say Round Robin or other distribution.

`orchestration.service.ts`

```javascript
async matchRunner(){}
async matchProject(){}
```

## Runners

### Handle unregistered runners

Runners which have been unregistered or do not have a registration anymore can still request and receive Heartbeats and Mutants. They should be instructed to re-register themselves. Also rename function to `heartBeat` from `heartBeast`.

`runner.controller.ts`

```javascript
async heartBeast(){}
```

## Admin

There is a very small application under `src/modules/admin` which provides endpoints for the queue size and running projects

### Testing

There are no tests yet for this module

### Queue size checks

These checks might be very resource intensive as they loop through each active project and check the queues. Caching or some form of prediction might be needed for large installations.

`admin.service.ts`

```javascript
async getQueueSizeByFramework(){}
```



### Queue size format

The queue size returned offer some very simple information which might need extensions for implementation with e.g. Kubernetes.

`admin.controller.ts`

```javascript
async getQueueSize(){}
```

## Generic

### Security

There is no login or other kind of security. Adding a security layer to the NestJS application should fix this. A dashboard for registration and management of users is also required then.

### Cleanup

The projects which have been run are not properly cleaned up yet. This includes:

- Redis queues and keys
  - Keys are created in `client.service.ts` and `orchestration.service.ts`
- Physical files on storage solution
  - See the `core/storage` folder

### Traceback reporting

Currently only mutant results are being reported back to the client. Any logs or tracebacks while running the mutant are ignored.

### Dashboard

Create a dashboard where information about the Orchestrator is displayed and management functionality is implemented.

