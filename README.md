# lightning-analysis
Lightning Non-performance Analysis on Reports

## Setup

### Linux environment

Python 3.5+ is required!

``` 
cd lightning-analysis
python3 -m pip install -r requirements.txt
jupyter notebook
```

### Windows environment

1. Go to [anconda products](https://www.anaconda.com/products/distribution) and download & install Anaconda Distribution.
2. During installation you must `Add Anaconda to my PATH environment variable`
4. Open Anaconda Navigator and launch Jupyter Notebook or via Anaconda prompt running `jupyter notebook`.


## Data

1. Download [Salesforce.zip](https://drive.google.com/drive/folders/1Ru1JEGT_cD0UiBXG7fNfHYnaa-ooJTES?usp=sharing) file from our shared point (Most recent version is recommended).
2. Unzip into `lightning-analysis/data/`

## Splunk Queries

Only Lightning events.

A join between LightningUriEventStream and ReportEventStream events on ReportId.

### Active reports

All possible fields according to [ReportEventStream](https://developer.salesforce.com/docs/atlas.en-us.platform_events.meta/platform_events/sforce_api_objects_reporteventstream.htm).

`ACTIVE_REPORTS_QUERY`:

```
index=pcf_lightning-cns cf_org_name=CNSCRMOrg cf_app_name=cns-platform-events-logger 
  "msg.channelId"="/event/LightningUriEventStream"
| eval ReportId=substr('msg.content.RecordId', 1, 15) 
| eval DurationInSecs=tonumber('msg.content.Duration')/1000
| eval DurationInMinutes=tonumber('msg.content.Duration')/60000  
| rename
  "msg.content.AppName" as AppName,
  "msg.content.ConnectionType" as ConnectionType,
  "msg.content.CreatedById" as CreatedById,
  "msg.content.CreatedDate" as CreatedDate,
  "msg.content.DeviceId" as DeviceId,
  "msg.content.DeviceModel" as DeviceModel,
  "msg.content.DevicePlatform" as DevicePlatform,
  "msg.content.DeviceSessionId" as DeviceSessionId,
  "msg.content.Duration" as Duration,
  "msg.content.EffectivePageTime" as EffectivePageTime,
  "msg.content.EventDate" as EventDate,
  "msg.content.EventIdentifier" as EventIdentifier,
  "msg.content.Operation" as OperationType,
  "msg.content.OsName" as OsName,
  "msg.content.OsVersion" as OsVersion,
  "msg.content.PageStartTime" as PageStartTime,
  "msg.content.PageUrl" as PageUrl,
  "msg.content.PreviousPageAppName" as PreviousPageAppName,
  "msg.content.PreviousPageEntityId" as PreviousPageEntityId,
  "msg.content.PreviousPageEntityType" as PreviousPageEntityType,
  "msg.content.PreviousPageUrl" as PreviousPageUrl,
  "msg.content.QueriedEntities" as LeftQueriedEntities,
  "msg.content.RecordId" as RecordId,
  "msg.content.SdkAppType" as SdkAppType,
  "msg.content.SdkAppVersion" as SdkAppVersion,
  "msg.content.SdkVersion" as SdkVersion,
  "msg.content.SessionLevel" as SessionLevel,
  "msg.content.SourceIp" as SourceIp,
  "msg.content.UserId" as UserId,
  "msg.content.Username" as Username,
  "msg.content.UserType" as UserType
| fields
  AppName,
  ConnectionType,
  CreatedById,
  CreatedDate,
  DeviceId,
  DeviceModel,
  DevicePlatform,
  DeviceSessionId,
  Duration,
  DurationInSecs,
  DurationInMinutes,
  EffectivePageTime,
  EventDate,
  EventIdentifier,
  OperationType,
  OsName,
  OsVersion,
  PageStartTime,
  PageUrl,
  PreviousPageAppName,
  PreviousPageEntityId,
  PreviousPageEntityType,
  PreviousPageUrl,
  LeftQueriedEntities,
  RecordId,
  ReportId,
  SdkAppType,
  SdkAppVersion,
  SdkVersion,
  SessionLevel,
  SourceIp,
  UserId,
  Username,
  UserType
| join 
  ReportId 
[ 
  search index=pcf_lightning-cns cf_org_name=CNSCRMOrg cf_app_name=cns-platform-events-logger 
    "msg.content.Operation"=ReportRunFromLightning
  | rename 
    "msg.content.ColumnHeaders" as ColumnHeaders,
    "msg.content.DashboardId" as DashboardId,
    "msg.content.DashboardName" as DashboardName,
    "msg.content.Description" as Description,
    "msg.content.DisplayedFieldEntities" as DisplayedFieldEntities,
    "msg.content.EvaluationTime" as EvaluationTime,
    "msg.content.ExportFileFormat" as ExportFileFormat,
    "msg.content.Format" as Format,
    "msg.content.GroupedColumnHeaders" as GroupedColumnHeaders,
    "msg.content.Name" as Name,
    "msg.content.NumberOfColumns" as NumberOfColumns,
    "msg.content.OwnerId" as OwnerId,
    "msg.content.QueriedEntities" as QueriedEntities,
    "msg.content.ReportId" as ReportId,
    "msg.content.RowsProcessed" as RowsProcessed,
    "msg.content.RowsReturned" as RowsReturned,
    "msg.content.Scope" as Scope,
    "msg.content.Sequence" as Sequence,
    "msg.content.SessionLevel" as SessionLevel,
    "msg.content.SourceIp" as SourceIp
  | fields 
    ColumnHeaders,
    DashboardId,
    DashboardName,
    Description,
    DisplayedFieldEntities,
    EvaluationTime,
    ExportFileFormat,
    Format,
    GroupedColumnHeaders,
    Name,
    NumberOfColumns,
    OwnerId,
    QueriedEntities,
    ReportId,
    RowsProcessed,
    RowsReturned,
    Scope,
    Sequence,
    SessionLevel,
    SourceIp
]
```

### Active and problematic reports regarding time

All possible fields according to [LightningUriEventStream](https://developer.salesforce.com/docs/atlas.en-us.platform_events.meta/platform_events/sforce_api_objects_reporteventstream.htm).

`TIME_PROBLEMATIC_REPORTS_QUERY`:

```
index=pcf_lightning-cns cf_org_name=CNSCRMOrg cf_app_name=cns-platform-events-logger 
  "msg.channelId"="/event/LightningUriEventStream"
  "msg.content.EffectivePageTime"=0 
  "msg.content.Duration">60000
| eval ReportId=substr('msg.content.RecordId', 1, 15) 
| eval DurationInSecs=tonumber('msg.content.Duration')/1000
| eval DurationInMinutes=tonumber('msg.content.Duration')/60000  
| rename
  "msg.content.AppName" as AppName,
  "msg.content.ConnectionType" as ConnectionType,
  "msg.content.CreatedById" as CreatedById,
  "msg.content.CreatedDate" as CreatedDate,
  "msg.content.DeviceId" as DeviceId,
  "msg.content.DeviceModel" as DeviceModel,
  "msg.content.DevicePlatform" as DevicePlatform,
  "msg.content.DeviceSessionId" as DeviceSessionId,
  "msg.content.Duration" as Duration,
  "msg.content.EffectivePageTime" as EffectivePageTime,
  "msg.content.EventDate" as EventDate,
  "msg.content.EventIdentifier" as EventIdentifier,
  "msg.content.Operation" as OperationType,
  "msg.content.OsName" as OsName,
  "msg.content.OsVersion" as OsVersion,
  "msg.content.PageStartTime" as PageStartTime,
  "msg.content.PageUrl" as PageUrl,
  "msg.content.PreviousPageAppName" as PreviousPageAppName,
  "msg.content.PreviousPageEntityId" as PreviousPageEntityId,
  "msg.content.PreviousPageEntityType" as PreviousPageEntityType,
  "msg.content.PreviousPageUrl" as PreviousPageUrl,
  "msg.content.QueriedEntities" as LeftQueriedEntities,
  "msg.content.RecordId" as RecordId,
  "msg.content.SdkAppType" as SdkAppType,
  "msg.content.SdkAppVersion" as SdkAppVersion,
  "msg.content.SdkVersion" as SdkVersion,
  "msg.content.SessionLevel" as SessionLevel,
  "msg.content.SourceIp" as SourceIp,
  "msg.content.UserId" as UserId,
  "msg.content.Username" as Username,
  "msg.content.UserType" as UserType
| fields
  AppName,
  ConnectionType,
  CreatedById,
  CreatedDate,
  DeviceId,
  DeviceModel,
  DevicePlatform,
  DeviceSessionId,
  Duration,
  DurationInSecs,
  DurationInMinutes,
  EffectivePageTime,
  EventDate,
  EventIdentifier,
  OperationType,
  OsName,
  OsVersion,
  PageStartTime,
  PageUrl,
  PreviousPageAppName,
  PreviousPageEntityId,
  PreviousPageEntityType,
  PreviousPageUrl,
  LeftQueriedEntities,
  RecordId,
  ReportId,
  SdkAppType,
  SdkAppVersion,
  SdkVersion,
  SessionLevel,
  SourceIp,
  UserId,
  Username,
  UserType
| join 
  ReportId 
[ 
  search index=pcf_lightning-cns cf_org_name=CNSCRMOrg cf_app_name=cns-platform-events-logger 
    "msg.content.Operation"=ReportRunFromLightning
  | rename 
    "msg.content.ColumnHeaders" as ColumnHeaders,
    "msg.content.DashboardId" as DashboardId,
    "msg.content.DashboardName" as DashboardName,
    "msg.content.Description" as Description,
    "msg.content.DisplayedFieldEntities" as DisplayedFieldEntities,
    "msg.content.EvaluationTime" as EvaluationTime,
    "msg.content.ExportFileFormat" as ExportFileFormat,
    "msg.content.Format" as Format,
    "msg.content.GroupedColumnHeaders" as GroupedColumnHeaders,
    "msg.content.Name" as Name,
    "msg.content.NumberOfColumns" as NumberOfColumns,
    "msg.content.OwnerId" as OwnerId,
    "msg.content.QueriedEntities" as QueriedEntities,
    "msg.content.ReportId" as ReportId,
    "msg.content.RowsProcessed" as RowsProcessed,
    "msg.content.RowsReturned" as RowsReturned,
    "msg.content.Scope" as Scope,
    "msg.content.Sequence" as Sequence,
    "msg.content.SessionLevel" as SessionLevel,
    "msg.content.SourceIp" as SourceIp
  | fields 
    ColumnHeaders,
    DashboardId,
    DashboardName,
    Description,
    DisplayedFieldEntities,
    EvaluationTime,
    ExportFileFormat,
    Format,
    GroupedColumnHeaders,
    Name,
    NumberOfColumns,
    OwnerId,
    QueriedEntities,
    ReportId,
    RowsProcessed,
    RowsReturned,
    Scope,
    Sequence,
    SessionLevel,
    SourceIp
]
```

## Research questions


### Non-performing dashboards


### Non-performing reports

| Data source | Tables | Columns |
|:-|:-:|:-:|
| Objects (workbench)| Report |  Id, Name, LastRunDate etc.|
| ELF (logs) | Report | REPORT_ID_DERIVED |

| | NB | Owner | Splunk | Status |
|-|-|-|-|-|
|1. | Active reports | Jeferson | :frog: | `DONE` |
|1.1. | What are the most frequently run reports? | Jeferson | :frog: | `DONE` |
|1.2. | How long has each report been running? | Jeferson || `DONE`|
|1.3. | How long does it take to a page load these reports?| Izaquiel || `DONE`|
|1.3.1. | What types of errors are associated with longer duration reports according to the EFFECTIVE_PAGE_TIME?| Izaquiel ||`STARTED`|

#### Performance
  
| Data source | Tables | Columns |
|:-|:-:|:-:|
| Objects (workbench)| Report |  Id, Name, LastRunDate etc.|
| ELF (logs) | Report, LightningPerformance | REPORT_ID_DERIVED, CLIENT_GEO, CONNECTION_TYPE,  UI_EVENT_SOURCE, etc. |
| Datasets | `active_reports` |  includes all Report Object and Report logs columns |
 
| | NB | Owner | Splunk | Status |
|-|-|-|-|-|
|1. | Problematic reports regarding performance | Maciel | | `DONE` |
|1.1. | In which browsers are these reports running? | Marcos | | `REVIEW` |
|1.2. | What region or locality do most of the executions come from? | Marcos |  | `TODO`|
|1.3. | What connection types are users using to access the reports? | Izaquiel |  |`TODO`|
|1.4. | Are these accesses being made through mobile devices? | Kleber |  |`REVIEW`|
|1.5. | What OS types are used to access the reports? | Izaquiel |  |`REVIEW`|
|1.6. | What are the trends for user actions? | Debora | |`TODO`|


#### PageView 

| Data source | Tables | Columns |
|:-|:-:|:-:|
| Objects (workbench)| Report |  Id, Name, LastRunDate etc.|
| ELF (logs) | LightningPageView | DURATION, EFFECTIVE_PAGE_TIME, PAGE_CONTEXT, PAGE_START_TIME, PAGE_URL, USER_TYPE etc. |
| Datasets | `active_reports` |  includes all Report Object and Report logs columns |

| | NB | Owner | Splunk | Status |
|-|-|-|-|-|
|1. | Problematic reports regarding page load time| Jeferson | |`DONE`|
|1.2. | How long the user spent on the page?| | |`TODO`|
|1.3. | How long does it take to a page load these reports?| |  |`REDO`|
|1.4. | Which components the reports are running?| ||`DONE`|
|1.5. | Which columns are being used by more than one report?| ||`DONE`|
|1.6. | What kind of users are running these reports?| Klebler | |`STARTED`|

#### Report

| Data source | Tables | Columns |
|:-|:-:|:-:|
| Objects (workbench)| Report |  Id, Name, LastRunDate etc.| 
| ELF (logs) | Report | REPORT_ID_DERIVED, DB_TOTAL_TIME, DB_CPU_TIME, CPU_TIME, RUNT_TIME etc. |
| Datasets | `active_reports` |  includes all Report Object and Report logs columns |

| | NB | Owner | Splunk | Status |
|-|-|-|-|-|
|1. | Problematic reports regarding user run reports | Jeferson | | `TODO`|
|1.1. | Where is the database taking the longest to load?| | | `TODO`|
|1.2. | Where is the application taking the longest to load?| | | `TODO`|
|1.3. | In what contexts are these reports being run?| | |`TODO`|
|1.4. | How long does it take to complete the requests?| | |`TODO`|

#### Errors

| Data source | Tables | Columns |
|:-|:-:|:-:|
| Objects (workbench)| Report |  Id, Name, LastRunDate etc.|
| ELF (logs) | Report, LightningError | PAGE_URL, PAGE_CONTEXT (component), USER_AGENT etc. |
| Datasets | `active_reports` |  includes all Report Object and Report logs columns |

| | NB | Owner | Splunk | Status |
|-|-|-|-|-|
| 1. | Problematic reports regarding errors| Gabriela | | `DONE`|
| 1.1. | What is the trend for the errors? | Kleber ||`TODO`|
| 1.1.1. | Are the errors coming from specific pages?| Gabrielly ||`STARTED` |
| 1.1.2. | What region do most of the errors come from?| Gabrielly ||`STARTED` |
| 1.2. | Which components are related to the most common errors? | Kleber | |`DONE`|

