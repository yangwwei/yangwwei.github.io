---
title:  YARN Internals
tags:
  - YARN
---

# Hadoop Yarn Code Helper

### Resource Manager

During startup, RM starts several services, such as

- an event `Dispatcher`, it dispatches events to registered even handlers based on event types. This includes a lot of handlers for all sorts of events. Including container, task, nodemanager, containermanager, application events. See more in `RM Events` section.
- an `AdminService`, this server talks to RM via ResourceManagerAdministrationProtocol. This service exposes some admin commandline utilities, such as refreshQueues.
- if HA is enabled, starts an `EmbeddedElector` for active master election
- a http web server
- a `RMApplicationHistoryWriter` which writes information of RMApp, RMAppAttempt, RMContainer to history server
- a `RMTimelineCollectorManager`, so system metrics publisher can bind to it

#### RM Events

| Event Listener  |  Duty | Events  |
|---|---|---|---|---|
| ContainerLauncher | handles container luanch events | REMOTE\_LUNCH, COMPLETE,  CLEANUP |
| TaskAttemptImpl | handles the lifecycle events of a task | ASSIGNED, KILL, COMPLETE |
| NodeManager | handles node manager lifecycle events | SHUTDOWN, RESYNC |
| ContainerManager | handles container events | FINISH\_APPS, FINISH\_CONTAINERS, UPDATE\_CONTAINERS, SIGNAL\_CONTAINERS |
| Application | handles application events | APPCONTAINER\_INIT,  APPCONTAINER\_FINISHED, CONTAINER\_INIT, APPLICATION\_FINISHED, APPLICATION\_INIT(INITED) |
| Container | handles container events | INIT, DIAGNOSISTIC\_UPDATE, EXIT, KILL, PUASE, REINIT, RESOURCE, RESOURCEFAILED, RESOURCELOCALIZED, RESUME, UPDATE/_TOKEN |
| ContainerLauncher | handles container launch events | LAUNCH\_CONTAINER, RELAUNCH\_CONTAINER, RECOVER\_CONTAINER, RECOVER\_PAUSED_CONTAINER, CLEANUP\_CONTAINER, CLEANUP\_CONTAINER\_FOR\_REINIT, SIGNAL\_CONTAINER, PAUSE\_CONTAINER, RESUME\_CONTAINER |
| LocalizedResource | handles resource localization events | FAILED\_LOCALIZATION, LOCALIZED, RECOVERED, RELEASE, REQUEST |
| LocalResourceTracker | tracks localization status | LOCALIZED, REQUEST, RELEASE, LOCALIZATION_FAILED, RECOVERED |
| ResourceLocalizationService | handles localization events | INIT\_APPLICATION\_RESOURCES, LOCALIZE\_CONTAINER\_RESOURCES, CONTAINER\_RESOURCES\_LOCALIZED, CACHE\_CLEANUP, CLEANUP\_CONTAINER\_RESOURCES, DESTROY\_APPLICATION\_RESOURCES |
| ... | ... | ... |


#### RM Services

- **ResourceTrackerService**: handles node managers' heartbeats, keep trakcing of node manager resources.
- **ClientRMService**: handles all the RPC interfaces to the RM from the client, exposes client level APIs.
- **ApplicationMasterService**: this service impelements `ApplicationMasterProtocol` which is the protocol between `ApplicationMaster` and `ResourceManager`. This handles AM registration, allocate resources to the end of the AM lifecycle.
- **AdminService**: this server talks to RM via ResourceManagerAdministrationProtocol. This service exposes some admin commandline utilities, such as refreshQueues.
- **RMSecretManagerService**: handles AM, NM secret tokens.
- **RMDelegatedNodeLabelsUpdater**: update nodes labels map for ResourceManager periodically.
