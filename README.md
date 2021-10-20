# IMGA Event Centre Android SDK

## Introduction

This sample app shows how to integrate the IMGA front row seat SDK into an Android application.

Supported IDEs:

* [Android Studio](#androidstudio)

Supported build systems:

* [Gradle](#gradle)

## Adding the library to the project

### Using Github Packages Maven repository

Add the repository to your build.gradle file
```
repositories {
     maven {
            url = uri("https://maven.pkg.github.com/IMGARENA/front-row-seat-android-package")
            credentials {
                username = "username"
                password = "password"
            }
        }
}
```

Add the package dependencies to your build.gradle file
```
dependencies {

...

implementation 'com.imgarena:eventcentre:1.0.0'
}
```
### Adding the library locally to the project

Copy the eventcentre.aar to the libs folder of your application:

```
flatDir {
   dirs 'libs'
}
```

Then include the event centre in the dependencies section of your application: 

```
dependencies {
   implementation(name:'eventcentre', ext:'aar')
}
```


## Usage

### Initialiasing de SDK

#### Configuration parameters

|Name         |Type             |Description                                       |
|:------------|:----------------|:-------------------------------------------------|
|operator     |string           | your unique operator Id name                     |
|sport        |string           | "golf" or "ufc"                                  |
|version      |string           | the version of the Event Centre                  |
|eventId      |string           | refers to the event you wish to display          |
|targetModule |string           | specifies the module that the app should display |
|language     |string           | ISO code for the language of the translations    |

#### Golf Modularisation

The Event centre supports being initialised in seven different modular configurations, Provide the required module to the library:

|Module name         |Description                                     |Notes                                                               |
|:-------------------|:-----------------------------------------------|:-------------------------------------------------------------------|
|full                |All Event Centre                                |-                                                                   |
|leaderboard         |Leaderboard Only (no global navigation)         |-                                                                   |
|leaderboarddetail   |eaderboard & Team/Player (no global navigation) |-                                                                   |
|course              |Course & Hole Only (no global navigation)       |-                                                                   |
|hole                |Hole Only (No navigation back to Course)        |holeNo must be passed in initialContext, see below for more details |
|groupdetail         |Groups List & Group Detail                      |-                                                                   |
|groups              |Group Only                                      |-                                                                   |


#### Sample initialisation for **Golf**:

```kotlin
var params: EventCentreParams = EventCentreParams(
    operator = "[OPERATOR-NAME]",
    sport = "golf",
    version = "5.x",
    eventId = "207",
    targetModule = "full",
    language = "en"
)

// Optionally initialise a context
params.initialContext = mapOf(
	"view" to "GroupDetail",
	"roundNo" to "1",
	"groupNo" to "19",
	"holeNo" to "12"
)

```



#### UFC Modularisation

The UFC Event Centre supports being initialised in 3 different modular configurations. This is done by providing the module name within the "targetModule" field:

|Module name         |Description                                        |
|:-------------------|:--------------------------------------------------|
|full                |Access to the full UFC Event Centre                |
|fight               |Single fight                                       |
|Fightcard           |Only displays the fightcard - allows no navigation |

#### Sample initialisation for **UFC**:

```kotlin
var params: EventCentreParams = EventCentreParams(
    operator = "[OPERATOR-NAME]",
    sport = "ufc",
    version = "5.x",
    eventId = "eventID",
    targetModule = "full",
    language = "en"
)

// Optionally initialise a context
params.initialContext = mapOf(
	"view" to "Fight",
	"fightId" to "fightID"
)

```

#### Creating the Event Center instance
```kotlin
var imga = EventCentre(container, params )
```

### Event Centre: Emitting Messages

The **emitMessage** method is available on the instance of the eventCentre, it is used to send messages to the Event Centre. Only supported message topics will be passed to the Event Centre.

```kotlin
fun emitMessage(topic: EventCentreMessageTopics, message: EventCentreMessage)
```
The **EventCentreMessageTopics** enumcontains the following values:

* HANDSHAKE_FAILED: Dedicated topic to signify the handshake between Event Centre and your site failed to complete. This topic is emitted by the integration library, it should only be subscribed to, not emitted.
* CONTEXT_UPDATE: Topic for covering general UI state updates, for example navigation changes or the user selecting a player in the UI.
* SELECTION_UPDATE: Dedicated topic for handling user selection updates. The selected field within the message data handles both selecting and deselecting updates.
* ERROR: Topic sent if the SDK had any problem to be initialized. This topic is emitted by the integration library, it should only be subscribed to, not emitted.

The **EventCentreMessage** type is an alias of a HashMap<String, Any> class. Keys must be Strings and the Values can be any type.

#### Updating the Context
There is a convinience method to update the context:

```kotlin
fun updateContext ( context: EventCentreContext) {
```

This method will update the context of the widget, and receives only the parameter context of type **EventCentreContext**. This type is a subclass of **EventCentreMessage**.

#### GOLF context params

|Name         |Type             |Description                                       |
|:------------|:----------------|:-------------------------------------------------|
|view         |string           |The name of the active view                       |
|tournamentId |number or string |Identifier for the tournament                     |
|roundNo      |number or string |The round the user is currently viewing data from |
|holeNo       |number or string |The hole the user is currently viewing data from  |
|playerNo     |number or string |The player's ID                                   |
|groupNo      |number or string |The group's ID                                    |
|courseId     |number or string |The course's ID                                   |


#### UFC context params

|Name         |Type             |Description                                            |
|:------------|:----------------|:------------------------------------------------------|
|view         |string           |The name of the active view                            |
|eventId      |number or string |Event ID as defined by the DDE                         |
|fightId      |number or string |Fight ID as defined by DDE. Available for view = fight |



### Event Centre: Receiving Messages
In order to subscribe to different messages emitted by the SDK, the **EventCentreListener** interface must be implemented.

```kotlin
class TestActivity : AppCompatActivity(), EventCentreListener {
```
#### Subscribing to status changes
The SDK will send every change in its status using the following Listener method:

```kotlin
override fun onStatusChanged(status: EventCentreStatus) {
    Log.d("eventcentretest", "Transition to status: ${status.name}")

    Toast.makeText(this, status.toString(), Toast.LENGTH_SHORT).show()
}
```
The **EventCentreStatus** enum contains the following values:

* INITIALISING: the SDK is initialising and not fully loaded.
* ACTIVE: the SDK is ready to interact.
* EMITTING_MESSAGE: the SDK is emitting a message.
* INACTIVE: the SDK is closed and can't receive any message.

#### Subscribing to new messages

To receive context or selection updates from the SDK, the host app must subscribe to the Listener using this method:

```kotlin
override fun onMessageReceived(topic: EventCentreMessageTopics, message: JSONObject) {
    Log.d("eventcentretest", "Message received: $message")
    Toast.makeText(this, message.toString(2), Toast.LENGTH_SHORT).show()
}
```
The **EventCentreMessageTopics** enumcontains the following values:

* HANDSHAKE_FAILED: Dedicated topic to signify the handshake between Event Centre and your site failed to complete. This topic is emitted by the integration library, it should only be subscribed to, not emitted.
* CONTEXT_UPDATE: Topic for covering general UI state updates, for example navigation changes or the user selecting a player in the UI.
* SELECTION_UPDATE: Dedicated topic for handling user selection updates. The selected field within the message data handles both selecting and deselecting updates.
* ERROR: Topic sent if the SDK had any problem to be initialized. This topic is emitted by the integration library, it should only be subscribed to, not emitted.

### Stop the sdk

```kotlin
// Instance the sdk
var imga = EventCentre(container, params )

// Close the instance
imga.close()
```

## Credits

IMGA SDK is owned and maintained by the [IMGA Development team](https://www.imgarena.com/).

## License
Private license
