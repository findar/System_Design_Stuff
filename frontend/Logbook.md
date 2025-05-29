## Problem
Design a digital logbook for a logistics company.  This logbook would record when maintenance has occurred.  It would also provide insight into upcoming work to be done as well. 

## Requirements:

* CRU(D?) of maintenance records for a specific on a vehicle
* Tracking of mechanic who performed work/signed off
* Display alerts about upcoming/past due work 


### Non-Functional Requirements:

* High availability - should be able to always see some data
* Eventual Consistency is ok - Devices may not have connectivity but should eventually sync
* Fast - sub 1s delay to fetching data
* Notifications are timely - Should not be surprised by regular maintenance 

## Entities:
**Notification**

  `{ vehicleId, type, title, description, severity }`

**Vehicle**

  `{ vehicleId, vehicleMetaDataId, ownerId }`

**VehicleMetaData**

    `{vehicleId, description, ownerId, make, model, year, serialNumber, ...etc}`

**Maintenance Record**

  `{ vehicleId, timeStamp, description, mechanicId[], type(100hr, annual, etc), imageMetadata[] }`

**Mechanic**

  `{ mechanicId, name, license#, desc, ScannedDocumentId[], ProfilePicture }`

**ScannedDocument**

  `{ ScannedDocumentId, type, ImageMetadata }`

**ProfilePicture**

  `{ mechanicId, ImageMetadata }`

**ImageMetadata**

  `{ metadataId, description, s3Link }`

## API 

`GET notifications/{vehicleId}` -> returns notification[]

`GET logs/maint/{vehicleId}` -> Returns Maintenance Record[]

`POST logs/maint/` -> create new Maintenance Record
{ 
  ...fields
}

`GET /mechanics/{mechanicId}` -> returns Mechanic[] if no Id provided - good for dropdown // in SaaS tenantKey would be in the header

`PATCH logs/maint/{vehicleId}` -> update existing Maintenance Record   // we should keep diffs on backend

## High Level Design


Example landing page for maintenance details on a vehicle as well as what adding/reading a maintence log might look like

![image](https://github.com/user-attachments/assets/50d9b125-d1f2-4596-af87-811ca6e39a26)

LogLandingPage State:
`{ MaintenanceRecord[], Notification[], VehicleID }`

RecordViewState:
`{ VehicleId, MaintenanceRecord?, Mechanic? }`

Example alerts page - reached by clicking alert notification

![image](https://github.com/user-attachments/assets/717c39ab-0e96-4d08-9da8-b82c75b8eb18)

NotificationsLandingPage State:

` Notification[], VehicleId }`

## Digging Deep

**Q: Should logging into the app associate mechanic and pre-populate field any time the order is updated?**

A: Yes.  This would ensure that each mechanic can only sign off for their own work.  

**Q: How do we handle losing connectivity?**

A: Multiple scenarios to deal with.  Loss before/after vehicles stored?

  After: Queue records locally, reupload after regaining connectivity

  Before: Allow "offline" mode where we manually create a record and all associated data.  Will require a reconcilliation step on reconnection

**Q: How do we determine notifications?**

A: Each notification type can assign conditions for severity.  Ex: a PropellerSwap type might be 50 hours of flight, an AnnualInspection might be 1 year from previous inspection.  Our notification service then runs through each notification, checks conditions against the vehicle, and generates a list of notifications.  This is stored in cache on the service side and sent to the user.  This would allow easy unit testing of each.

## Notes ##

For notifications, we would implement the rules with the [Specification Pattern](https://en.wikipedia.org/wiki/Specification_pattern).  This would allow clear descriptions and testability.  

We could then run the notification service on a cron job daily for computation and update the `Notifications` datastore as well as push a notification to client.
