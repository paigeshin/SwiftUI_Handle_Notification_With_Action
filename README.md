# SwiftUI_Handle_Notification_With_Action

- notification을 받고 action을 선택시 다른 화면으로 transition하게 하는 코드 예제

```swift
//
//  RichNotificationApp.swift
//  RichNotification
//
//  Created by paige shin on 2021/03/07.
//

import SwiftUI

@main
struct RichNotificationApp: App {
    
    @UIApplicationDelegateAdaptor private var appDelegate:AppDelegate
    
    var body: some Scene {
        WindowGroup {
            ContentView()
            
        }
    }
}

class AppDelegate: NSObject, UIApplicationDelegate {
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {
        return true
    }
    
    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data)          {
        print(deviceToken)
    }
    func application(_ application: UIApplication, didFailToRegisterForRemoteNotificationsWithError error: Error) {
        print(error.localizedDescription)
    }
}

class NotificationReceiver: NSObject, ObservableObject, UNUserNotificationCenterDelegate {
    
    override init() {
        super.init()
        UNUserNotificationCenter.current().delegate = self
    }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        
        if response.actionIdentifier == "open" {
            
            NotificationCenter.default.post(name: NSNotification.Name("Detail"), object: nil)
            
        }
        
    }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) { }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter, openSettingsFor notification: UNNotification?) { }
    
}

struct ContentView: View {
    
    @ObservedObject var notificationReceiver = NotificationReceiver()
    @State var show = false
    
    var body: some View {
        
        NavigationView {
            ZStack {
                
                NavigationLink(
                    destination: DetailView(show: self.$show),
                    isActive: self.$show,
                    label: {
                        Text("")
                    })
                
                Button(action: {
                    
                    send()
                    
                }, label: {
                    Text("Send Notification")
                })
                .navigationBarTitle("Home")
                
            }
            .onAppear {
                NotificationCenter.default.addObserver(forName: NSNotification.Name("Detail"), object: nil, queue: .main) { (_) in
                    self.show = true
                }
            }
        }
        
    }
    
    func send() {
        
        UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) { (_, _) in
            
        }
        
        //create basic content
        let content = UNMutableNotificationContent()
        content.title = "Message"
        content.body = "New Tutorial From Paige"
        
        //add action with categories
        let open = UNNotificationAction(identifier: "open", title: "Open", options: .foreground)
        let cancel = UNNotificationAction(identifier: "cancel", title: "Cancel", options: .destructive)
        let categories = UNNotificationCategory(identifier: "action", actions: [open, cancel], intentIdentifiers: [])
        UNUserNotificationCenter.current().setNotificationCategories([categories])
        content.categoryIdentifier = "action"
        
        //trigger
        let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
        
        //create request with `content` and `trigger`
        let request = UNNotificationRequest(identifier: "request", content: content, trigger: trigger)
        
        //add pending request
        UNUserNotificationCenter.current().add(request, withCompletionHandler: nil)
    }
    
}

struct DetailView: View {
    
    @Binding var show: Bool
    
    var body: some View {
        
        Text("Detail")
            .navigationBarTitle("Detail View")
            .navigationBarBackButtonHidden(true)
            .navigationBarItems(leading:
                Button(action: {
                    self.show = false
                }, label: {
                    Image(systemName: "arrow.left")
                        .font(.title)
                })
            )
        
    }
    
}
```