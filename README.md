# iOS WatchConnectivity Tutorial
This tutorial covers the basics of connecting an iOS and Watch app together, as well as sending data between the two.
Language: Swift 3

### Introduction
You want to get up and running fast with building your first Apple Watch app and are probably frustrated beyond belief that there are very few resources that explain how the little lego piece actually come together.  Well, look no further.  This tutorial will explain everything to build a basic app in no time.  After this, you should be able to venture off to build virtually any app you'd like, depending on your familiarity with iOS, WatchKit, and Object-Oriented Programming.

### What You Will Learn
* Overview of how WatchKit works with iOS
* How to CONNECT your Apple Watch app to your iPhone
* How to SEND data to your app from your iPhone
* How to SEND data to your phone from your Watch app.

### What You Will Not Learn
* How to build the interface layout for the WatchKit app
* How to build the storyboard layout for the iOS app

*I will give you some references at the end to point you in the right direction to figure that stuff out.*

### Overview of WatchKit
WatchKit is specifically designed to help you manage your UI in your Watch apps.  Think of it like the Watch version of UIKit. Your app's logic lives in the WatchKit extension. Every interface element is a subclass of the WKInterfaceObject class. Your WatchKit UI is managed by an InterfaceController, a subclass of the WKInterfaceController class. When you add Interfaces (WatchKit's version of Views), you also need to add some logic to that interface using a subclass of the WKInterfaceController class.  If you don't know how you would go about doing this, I would recommend learning the basics of iOS development by taking some courses on *Lynda.com*, *TeamTreehouse.com*, *Udemy.com*, or find an awesome technical book on iOS development from *SafariBooksOnline.com*.

As of Swift 3, every Watch app is classified as a companion to an iOS app.  That means, *good luck selling your app in the App Store without an accompanying iOS application.*  Your watch app is nested inside of the iOS app in Xcode as a WatchKit extension.  You will have three groups/folders/directories in which you will have to make some edits, either in program logic, or in design.

* Group 1: Your App
* Group 2: Your App WatchKit App
* Group 3: Your App WatchKit Extension


Both the iOS and WatchKit applications will need to communicate via the WatchConnectivity framework using ```WCSession.default()```.  To respond to any messages being sent from either device, you will need to implement the ```WCSessionDelegate```.  You will pass messages back and forth using dictionaries ```[String:Any]``` (*well, that's the preferred option*) and will need to handle each case independently in the callback method, whereas the response will be handled in a closure from the method call where the request was made. In other words...

##### iPhone App
> 1. import WatchConnectivity and conform to WCSessionDelegate.
> 2. use the static session delegate via ```WCSession.default()```.
> 3. Send data to Watch app using:
> ```swift
>  WCSession.default().sendMessage(message, replyHandler:_ errorHandler:_)
> ```
> 4. The message object should be a dictionary of type [String:Any]
> 5. If you are looking for data to be returned from the Watch app, provide the logic in a closure defined in the replyHandler; otherwise, pass in nil.
> 6. To respond to messages sent from the Watch app, you will use the WCSessionDelegate callback method
> ```swift
> func session(_ session: WCSession, didReceiveMessage message: [String : Any], replyHandler: @escaping ([String : Any]) -> Swift.Void){...}
> ```


##### WatchKit App
> 1. import WatchConnectivity and conform to WCSessionDelegate.
> 2. use the static session delegate via ```WCSession.default()```.
> 3. Send data to the iPhone app using:
> ```swift
>  WCSession.default().sendMessage(message, replyHandler:_ errorHandler:_)
> ```
> 4. The message object should be a dictionary of type [String:Any]
> 5. If you are looking for data to be returned from the Watch app, provide the logic in a closure defined in the replyHandler; otherwise, pass in nil.
> 6. To respond to messages sent from the iPhone app, you will use the WCSessionDelegate callback method
> ```swift
> func session(_ session: WCSession, didReceiveMessage message: [String : Any], replyHandler: @escaping ([String : Any]) -> Swift.Void){...}
> ```
> 7. These methods will not be required in your Watch app to properly conform to the WCSessionDelegate:
> ```swift
>  func sessionDidBecomeInactive(_ session: WCSession)
>  func sessionDidDeactivate(_ session: WCSession)
>  func sessionWatchStateDidChange(_ session: WCSession)
> ```

And finally, it is generally best practice to store any common images, the images and assets that will continually be used for the Watch app to be placed in the Watch's xcassets folder.  Okay, now that you are confused, let's get to the details!

### To The Details
#### Connecting the Apps

##### Configuring the iOS App
You need to import the WatchConnectivity framework into your *view controller* file in the iOS application.  This gives you access to the class objects that are designed to communicate with the Watch companion application.  You will need to import this on the Watch app side as well.  The only real difference with the Watch app is that you will not have a *view controller* file, but an *interface controller* file.

```swift
import WatchConnectivity
```

Next, you will need to make sure that your application can support a session for WatchConnectivity.  If it can, then you must set the view controller as its delegate and activate the default session.  You will get an error here.  You need to conform to WCSessionDelegate and implement a few methods before the IDE begins to calm down.

```swift
// MARK: - View Life Cycle Callbacks
override func viewDidLoad() {
    super.viewDidLoad()

    automaticallyAdjustsScrollViewInsets = false
    if WCSession.isSupported() {
        WCSession.default().delegate = self
        WCSession.default().activate()
    }
    else {
        print("\nViewController: connectionManager is nil\n")
    }
}
```

To conform to WCSessionDelegate, let's add an extension to the bottom of the view controller.  Some people hate this approach.  I have my way, but for tutorial purposes, I will follow the *RayWenderlich* approach.  I am a huge fan of just getting code directly from COMMAND + CLICKING on the delegates and grabbing ALL OF THE METHODS out of the specification and begin to manipulate it and understand how things work.  In this extension, I am providing you will all of the methods.  They are already marked so that you will see when each of them fire in the console.  If you feel frisky, delete some of the ones that are marked OPTIONAL to make your code file look spiffy.

```swift
extension ViewController : WCSessionDelegate {

    func session(_ session: WCSession, activationDidCompleteWith activationState: WCSessionActivationState, error: Error?) {
        print("0. ViewController: ", "activationDidCompleteWith activationState")
    }


    /** ------------------------- iOS App State For Watch ------------------------ */

    func sessionDidBecomeInactive(_ session: WCSession) {
        print("1. ViewController: ", "sessionDidBecomeInactive")
    }


    func sessionDidDeactivate(_ session: WCSession) {
        print("2. ViewController: ", "sessionDidDeactivate")
    }


    func sessionWatchStateDidChange(_ session: WCSession) {
        print("3. ViewController: ", "sessionDidDeactivate")
    }


    /** ------------------------- Interactive Messaging ------------------------- */

    func sessionReachabilityDidChange(_ session: WCSession) {
        print("4. ViewController: ", "sessionReachabilityDidChange")
    }


    func session(_ session: WCSession, didReceiveMessage message: [String : Any]) {
        print("5. ViewController: ", "didReceiveMessage")
    }



    func session(_ session: WCSession, didReceiveMessage message: [String : Any], replyHandler: @escaping ([String : Any]) -> Swift.Void) {
        print("6. ViewController: ", "didReceiveMessage")
        // This is where you handle any requests coming from your Watch App
    }

    func session(_ session: WCSession, didReceiveMessageData messageData: Data) {
        print("7. ViewController: ", "didReceiveMessageData")
    }

    func session(_ session: WCSession, didReceiveMessageData messageData: Data, replyHandler: @escaping (Data) -> Swift.Void) {
        print("8. ViewController: ", "didReceiveMessageData")
    }

    /** -------------------------- Background Transfers ------------------------- */

    func session(_ session: WCSession, didReceiveApplicationContext applicationContext: [String : Any]) {
        print("9. ViewController: ", "didReceiveApplicationContext")
    }

    func session(_ session: WCSession, didFinish userInfoTransfer: WCSessionUserInfoTransfer, error: Error?) {
        print("10. ViewController: ", "didFinish userInfoTransfer")
    }

    func session(_ session: WCSession, didReceiveUserInfo userInfo: [String : Any] = [:]) {
        print("11. ViewController: ", "didReceiveUserInfo")
    }

    func session(_ session: WCSession, didFinish fileTransfer: WCSessionFileTransfer, error: Error?) {
        print("12. ViewController: ", "didFinish fileTransfer")
    }

    func session(_ session: WCSession, didReceive file: WCSessionFile) {
        print("13. ViewController: ", "didReceive file")
    }
}
```

To send data to your Watch App, once the Watch App has requested it, you will handle this in the method...

```swift
func session(_ session: WCSession, didReceiveMessage message: [String : Any], replyHandler: @escaping ([String : Any]) -> Swift.Void) {
    print("6. ViewController: ", "didReceiveMessage")
    // build out your response message using a Dictionary
    let returnMessage: [String : Any] = [
       "key1" : value1,
       "key2" : value2,
       "key3" : value3
    ]
    // return your data in this manner
    replyHandler(returnMessage)
    // WARNING
    // You must call the replyHandler before the method ends, otherwise, your app will crash.
}
```

This is only one half of the transaction!  You must configure your Watch app to connect to the iPhone app and handle any returned messages!


##### Configuring the Watch App

It might be a surprise to you, but you need to import WatchConnectivity again.

```swift
import WatchConnectivity
```

Next, you need to verify that the session is even possible.

```swift
// MARK: - View Life Cycle Callbacks
override func awake(withContext context: Any?) {
    super.awake(withContext: context)

    // Configure interface objects here.
    if WCSession.isSupported() {
        WCSession.default().delegate = self
        WCSession.default().activate()
        print("InterfaceController: Session Activated")

        // Request Data from iPhone App
        let requestMessage = ["message":"get-data"]

        WCSession.default().sendMessage(requestMessage, replyHandler: { (replyMessage) in
            print("Got a reply from the phone: \(replyMessage)")

            // handle reply message here

        }, errorHandler: { (error) in
            print("Got an error sending to the phone: \(error)")
        })
    }
    else {
        print("\nViewController: connectionManager is nil\n")
    }
}
```

But none of this will work, unless you implement methods required for the WCSessionDelegate.

```swift
extension InterfaceController : WCSessionDelegate {

    func session(_ session: WCSession, activationDidCompleteWith activationState: WCSessionActivationState, error: Error?) {
        print("1. InterfaceController: ", "activationDidCompleteWith activationState") // first
    }

    /** ------------------------- Interactive Messaging ------------------------- */

    func sessionReachabilityDidChange(_ session: WCSession) {
        print("2. InterfaceController: ", "sessionReachabilityDidChange") // second
    }

    func session(_ session: WCSession, didReceiveMessage message: [String : Any]) {
        print("3. InterfaceController: ", "didReceiveMessage")
    }

    func session(_ session: WCSession, didReceiveMessage message: [String : Any], replyHandler: @escaping ([String : Any]) -> Swift.Void) { // third
        print("4. InterfaceController: ", "didReceiveMessage")
        //print("Message Contents: ", message["message"]!)
    }

    func session(_ session: WCSession, didReceiveMessageData messageData: Data) {
        print("5. InterfaceController: ", "didReceiveMessageData")
    }

    func session(_ session: WCSession, didReceiveMessageData messageData: Data, replyHandler: @escaping (Data) -> Swift.Void) {
        print("6. InterfaceController: ", "didReceiveMessageData")
    }

    /** -------------------------- Background Transfers ------------------------- */

    func session(_ session: WCSession, didReceiveApplicationContext applicationContext: [String : Any]) {
        print("7. InterfaceController: ", "didReceiveApplicationContext")
    }

    func session(_ session: WCSession, didFinish userInfoTransfer: WCSessionUserInfoTransfer, error: Error?) {
        print("8. InterfaceController: ", "didFinish userInfoTransfer")
    }

    func session(_ session: WCSession, didReceiveUserInfo userInfo: [String : Any] = [:]) {
        print("9. InterfaceController: ", "didReceiveUserInfo")
    }

    func session(_ session: WCSession, didFinish fileTransfer: WCSessionFileTransfer, error: Error?) {
        print("10. InterfaceController: ", "didFinish fileTransfer")
    }

    func session(_ session: WCSession, didReceive file: WCSessionFile) {
        print("11. InterfaceController: ", "didReceive file")
    }    
}
```

### Sending Data
Sending data to your Watch app from your iPhone or from your iPhone to your Watch App after triggering some event is very simple, though the code can look a little complex at first.  
```swift
let message = ["key":"value-to-send"]

WCSession.default().sendMessage(message, replyHandler: { (replyMessage) in
    print("Got a reply from the phone: \(replyMessage)")

    if let returnedValues = replyMessage["returned-value"] as? NSArray {
        for val in returnedValues {
            // do something here with the data
            // Dispatch to Main Thread if affecting UI
        }
    }
}, errorHandler: { (error) in
    print("Got an error sending to the phone: \(error)")
})
```
### Conclusion
This has been a down and dirty on WatchConnectivity.  There is a whole lot more to cover at another Bat Time and another Bat Channel, but this should get you started on the right foot.  If you are bored with this tutorial, then continue in your development journey by tackling some of the more advanced topics such as:
* Working with Multiple Interface Controllers
* Creating Glances / Complications
* Notifications

I truly hope this tutorial has helped you in your journey as an iOS developer who is gaining a specialty in Apple Watch app development.
