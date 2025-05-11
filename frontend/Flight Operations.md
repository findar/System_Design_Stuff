# Problem

Design a front end to display flight operations of a hypothetical drone logistics company.  At a high level this should let me see my overall health, as well as provide the ability to drill down and find more information about entities.


## Requirements:
* Show health on a map 
* able to see information about drones 

## Non-Funct Requirements
* availability for things like maint records 
* consistency for things like flight status 
* fast - near-real time tracking is ideal
* up to date - missing/stale data should show warnings 

## Design thoughts out loud
* Main dash with overall health 
* geo map visualizing drones, clickable item with indicator matching status
*	click status to see more details
* followup page:
    * list of drones + health
		* click drone to see more details (maint/flight records)
* when fetching data, have an indicator (spinner, beachball, etc)
		

## Core entities(data)
* **drone** 
    * {droneId, flightId[], maintenanceId[],nestId, health}        
* **flight** 
    * {flightId, timestamp, flightTime, droneId, destinationCoord, departureCoord}
* **maintenance** 
    * {maintenanceId, droneId, description, type}
* **nest** 
    * {nestId, droneId[], health, coord}



## API Design
// All API's require auth, we likely pass auth token in the header 

`GET /nests/`
    -> returns nests[]

`GET /nests/{nestId}`
    -> returns nest

`GET /nests/{nestId}/drones `
    -> returns droneId[] at nestId

`GET /nests/{nestId}/flights `
    -> returns flights[] at nestId 

`GET /drones/{droneId}`
    -> returns drone 

`GET /flights/{flightId}`
    -> returns flight

`GET /drones/{droneId}/flights`
    -> returns flight[] of drone, unsorted

`GET /drones/{droneId}/maintenance`
    -> returns maintenance[] of drone, unsorted


## High Level Design

load a google map with auth overlay
    Map should start either with a global view or regional view, depending on feedback
Once Auth'd => `GET /nests/`

create elements on map at lat/lon 
    style based on STATE 
    if multiple in same area, use clustering to join
    always use the worst STATE to style - allows user to drill in to see why 

if click on element -> 
    zoom in on area 
    if we're using clustering, generate a zoom to show all grouped nodes 
    if not clustering, zoom in on the nest + radius of X where X is calculated from the flight distance 

with zoomed-to-nest-view 
    show active flights (dotted line between departure and destination, could indicate direction by animation)
        GET /nests/{nestId}/flights 
    
    show health of node 
    clicking on the nest can open a side bar 
        GET /nests/{nestId}/drones 
    side bar is a data table of drones 
    clicking an item in the data table can jump to a new view with drone details 
        GET /drones/{droneId}
        
## Deep Dive
* for keeping things up to date:
    Server side events the best approach - we don't change data just view it 
    would want to be getting any updates for 
    change in STATE would trigger a SSE for nests/drone 
    change in FLIGHTS would trigger a SSE if in zoomed in view 
