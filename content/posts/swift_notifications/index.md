---
title: "SwiftUI: Switching to a page when a user clicks on notification"
date: 2022-02-12T06:32:07-04:00
hero: /images/heros/hero-office.png
images: "images/"
menu:
  sidebar:
    name: "SwiftUI: Switching to a page when a user clicks on notification"
    identifier: fixutre-helpers
    weight: -273
tags: [swiftui, swift, note2self]
---

### Quick Notes to Self on how to have the user go to a specific page after clicking on a Notification in iOS.


<img src="images/notifications.gif" width="600">


The two main articles that helped are [https://stackoverflow.com/questions/62504400/programmatically-change-to-another-tab-in-swiftui](https://stackoverflow.com/questions/62504400/programmatically-change-to-another-tab-in-swiftui)

[https://stackoverflow.com/questions/62309251/how-can-i-open-a-specific-view-in-swiftui-when-a-user-opens-a-notification](https://stackoverflow.com/questions/62309251/how-can-i-open-a-specific-view-in-swiftui-when-a-user-opens-a-notification)

One helped me to see how in a more “modern” SwiftUI way get the state of the Notification from the UIDelegate up to the SwiftUI view. The second one was key since I am using TabView and needed to change the view when this was updated.


This will assume you have Firebase notifications already working. The pattern I used there can be seen in [https://designcode.io/swiftui-advanced-handbook-push-notifications-part-1](https://designcode.io/swiftui-advanced-handbook-push-notifications-part-1) some really great material here as well as [https://seanallen.teachable.com](https://seanallen.teachable.com) 

## Setup up `NotificationManager` 
I followed the first post but I was a bit more specific since I just, for now, have one page.

For example when that person setups up the `pageToNavigateTo` a number I just set it to 

```swift
    notificationManager?.pageToNavigationTo = "myRecalls"
```

This means I had to set the class as well

```swift
class NotificationManager: ObservableObject {
    static let shared = NotificationManager()
    @Published var pageToNavigationTo : String?
}
```

From here, I followed along with everything else till they get to the page with the Navigation since I am using tabs. That is where the second article comes in.  I set my tags on the Tabs and then set the default.

```swift
    @State private var tabSelection = "topics"
    
    var body: some View {
        ZStack {
            VStack {
                if(authState.signedIn) {
                    TabView(selection: $tabSelection) {
                        TopicsParentsView()
                            .tabItem {
                                Label("Topics", systemImage: "house")
                            }
                            .tag("topics")
                        MyRecallsView() .tabItem {
                            Label("My Recalls", systemImage: "person.crop.rectangle")
                        }
                        .tag("myRecalls")
                        RecallsView() .tabItem {
                            Label("Recalls", systemImage: "list.dash")
                        }
                        .tag("recalls")
                        AccountView()
                            .tabItem {
                                Label("My Account", systemImage: "person.crop.circle")
                            }
                            .tag("account")
```

That was it. After a rebuild my notifications brought me right to the page. For me the articles, when I actually followed all the details, worked perfectly. Below are the full examples.

```swift
@main
struct TotalRecallsIoApp: App {
    @UIApplicationDelegateAdaptor(AppDelegate.self) var appDelegate
    
    
    let notificationManager = NotificationManager()
    
    
    func setUpdNotificationManager() {
         appDelegate.notificationManager = notificationManager
     }
     
    
    var body: some Scene {
        let authState = AuthState()
        WindowGroup {
            HomeView()
                .environmentObject(authState)
                .environmentObject(notificationManager)
                .onAppear {
                    setUpdNotificationManager()
                }
        }
    }
}

class NotificationManager: ObservableObject {
    static let shared = NotificationManager()
    @Published var pageToNavigationTo : String?
}
```

Then the AppDelegate

```swift
@available(iOS 10, *)
extension AppDelegate : UNUserNotificationCenterDelegate {

    
  // Receive displayed notifications for iOS 10 devices.
  func userNotificationCenter(_ center: UNUserNotificationCenter,
                              willPresent notification: UNNotification,
    withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    let userInfo = notification.request.content.userInfo

    if let messageID = userInfo[gcmMessageIDKey] {
        print("Message ID: \(messageID)")
    }

    print(userInfo)

    // Change this to your preferred presentation option
    completionHandler([[.banner, .badge, .sound]])
  }

    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {

    }

    func application(_ application: UIApplication, didFailToRegisterForRemoteNotificationsWithError error: Error) {

    }

  func userNotificationCenter(_ center: UNUserNotificationCenter,
                              didReceive response: UNNotificationResponse,
                              withCompletionHandler completionHandler: @escaping () -> Void) {
    let userInfo = response.notification.request.content.userInfo

    if let messageID = userInfo[gcmMessageIDKey] {
      print("Message ID from userNotificationCenter didReceive: \(messageID)")
    }

    print(userInfo)
      
    print("DEBUG: User clicks on notifications")
      
    
    notificationManager?.pageToNavigationTo = "myRecalls"
            
    print("DEBUG: Done setting page \(notificationManager?.pageToNavigationTo)")
    completionHandler()
  }
}
```

Finally the page with TabView

```swift
import SwiftUI
import Firebase


struct HomeView: View {
    
    @EnvironmentObject var authState: AuthState
    @EnvironmentObject private var notificationManager: NotificationManager
    
    @State var email = ""
    @State var password = ""
    @State var signedIn = false
    @State private var tabSelection = "topics"
    
    var body: some View {
        ZStack {
            VStack {
                if(authState.signedIn) {
                    TabView(selection: $tabSelection) {
                        TopicsParentsView()
                            .tabItem {
                                Label("Topics", systemImage: "house")
                            }
                            .tag("topics")
                        MyRecallsView() .tabItem {
                            Label("My Recalls", systemImage: "person.crop.rectangle")
                        }
                        .tag("myRecalls")
                        RecallsView() .tabItem {
                            Label("Recalls", systemImage: "list.dash")
                        }
                        .tag("recalls")
                        AccountView()
                            .tabItem {
                                Label("My Account", systemImage: "person.crop.circle")
                            }
                            .tag("account")
                    }
                } else {
                    LoginView()
                }
            }
            .onAppear {
                authState.signedIn = authState.isSignedIn
            }
            .onReceive(notificationManager.$pageToNavigationTo) {
                guard let notificationSelection = $0 else  { return }
                self.tabSelection = notificationSelection
            }
        }
    }
    
    
}

struct HomeView_Previews: PreviewProvider {
    
    @EnvironmentObject var firebaseManager: FirebaseManager
    static var previews: some View {
        let authState = AuthState()
        HomeView().environmentObject(authState)
    }
}
```