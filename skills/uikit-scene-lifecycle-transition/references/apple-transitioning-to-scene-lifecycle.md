<!--
{
  "documentType" : "article",
  "framework" : "UIKit",
  "identifier" : "/documentation/UIKit/transitioning-to-the-uikit-scene-based-life-cycle",
  "metadataVersion" : "0.1.0",
  "role" : "article",
  "title" : "Transitioning to the UIKit scene-based life cycle"
}
-->
# Transitioning to the UIKit scene-based life cycle
Adopt the scene-based life cycle to replace the app delegate life cycle in UIKit.
## Overview
The UIKit scene-based life cycle separates the app process life cycle from the UI life cycle, allowing apps to manage multiple instances of UI independently. For example, a document-based app, such as a text editor, can display each open document in its own scene, letting people work on multiple documents side by side. Your app process launches once and [`UIApplicationDelegate`](/documentation/UIKit/UIApplicationDelegate) handles it, but each piece of visible UI — each *scene* — has its own independent life cycle that [`UISceneDelegate`](/documentation/UIKit/UISceneDelegate) and [`UIWindowSceneDelegate`](/documentation/UIKit/UIWindowSceneDelegate) coordinate. [`UISceneDelegate`](/documentation/UIKit/UISceneDelegate) handles basic life-cycle events that apply to all scenes, while [`UIWindowSceneDelegate`](/documentation/UIKit/UIWindowSceneDelegate) takes care of UI-specific events like window controls and geometry changes. A scene can go to the background while another stays active, and the system can create, destroy, and arrange scenes without affecting others.
UIKit represents each scene as a [`UIWindowScene`](/documentation/UIKit/UIWindowScene) object. For your app’s code, this means life-cycle events no longer occur globally — they occur per scene. Tasks like saving state or updating your UI happen at the scene level rather than the app level.
> Important:
> Adopting the scene-based life cycle is required. Beginning in iOS 27, iPadOS 27, Mac Catalyst 27, tvOS 27, and visionOS 27, apps built with the latest SDK must adopt the scene-based life cycle or they fail to launch.
For more information about configuring scene support, see [Specifying the scenes your app supports](/documentation/UIKit/specifying-the-scenes-your-app-supports).
Starting in iOS 18.4, iPadOS 18.4, Mac Catalyst 18.4, tvOS 18.4, and visionOS 2.4, UIKit logs this message for apps that haven’t migrated:
```other
This process does not adopt UIScene lifecycle. 
This will become an assert in a future version.
```
In iOS 26, iPadOS 26, Mac Catalyst 26, tvOS 26, and visionOS 26, the message changes to:
```other
UIScene lifecycle will soon be required.
Failure to adopt will result in an assert in the future.
```
## Determine if your app needs to migrate
Migrate to the scene-based life cycle if your app meets either of the following conditions:
- The <doc://com.apple.documentation/documentation/BundleResources/Information-Property-List/UIApplicationSceneManifest> key is missing from your information property list, or it has no specified configurations.
- Your app delegate doesn’t implement [`application(_:configurationForConnecting:options:)`](/documentation/UIKit/UIApplicationDelegate/application(_:configurationForConnecting:options:)).
## Adopt the scene-based life cycle
To configure your app’s scenes, add a <doc://com.apple.documentation/documentation/BundleResources/Information-Property-List/UIApplicationSceneManifest> key with a scene configuration to your information property list. If your app requires dynamic scene configurations — such as customizing scenes based on user activities, or handling different scene roles — implement [`application(_:configurationForConnecting:options:)`](/documentation/UIKit/UIApplicationDelegate/application(_:configurationForConnecting:options:)) in your app delegate instead.
### Configure the information property list for scene support
Add a <doc://com.apple.documentation/documentation/BundleResources/Information-Property-List/UIApplicationSceneManifest> key with a scene configuration to your information property list:
1. Open your Xcode project.
2. Select your app target.
3. Go to the General settings for your app target.
4. Select “Scene manifest” in the Deployment Info section.
5. Add a <doc://com.apple.documentation/documentation/BundleResources/Information-Property-List/UIApplicationSceneManifest> key to the information property list.
For example:
```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <key>UIApplicationSupportsMultipleScenes</key>
    <false/>
    <key>UISceneConfigurations</key>
    <dict>
        <key>UIWindowSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneConfigurationName</key>
                <string>Default Configuration</string>
                <key>UISceneDelegateClassName</key>
                <string>$(PRODUCT_MODULE_NAME).SceneDelegate</string>
                <key>UISceneStoryboardFile</key>
                <string>Main</string>
            </dict>
        </array>
    </dict>
</dict>
```
> Note: Supporting multiple scenes is optional, and may require restructuring your app’s data model to be scene-specific. Consider whether your app’s user experience benefits from multiple scenes before enabling it. To support multiple scenes, set <doc://com.apple.documentation/documentation/BundleResources/Information-Property-List/UIApplicationSceneManifest/UIApplicationSupportsMultipleScenes> to `true` and give each ``doc://com.apple.uikit/documentation/UIKit/UISceneConfiguration`` a unique configuration name.
### Provide scene configurations from your app delegate
Implement [`application(_:configurationForConnecting:options:)`](/documentation/UIKit/UIApplicationDelegate/application(_:configurationForConnecting:options:)) in your app delegate if your app doesn’t include scene-configuration data in its information property list, or if it requires dynamic scene configuration — such as loading different scenes based on session-specific data:
```swift
@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    func application(
        _ application: UIApplication,
        configurationForConnecting connectingSceneSession: UISceneSession,
        options: UIScene.ConnectionOptions
    ) -> UISceneConfiguration {
        // Each `UISceneConfiguration` must have a unique configuration name
        // that corresponds to an entry in the information property list scene manifest.
        let configurationName: String
        switch options.userActivities.first?.activityType {
        case UserActivity.GalleryOpenInspectorActivityType:
            // Create a photo inspector window scene.
            configurationName = "Inspector Configuration"
        default:
            // Create a default gallery window scene.
            configurationName = "Default Configuration"
        }
        return UISceneConfiguration(
            name: configurationName,
            sessionRole: connectingSceneSession.role
        )
    }
}
```
In this example, the app uses the <doc://com.apple.documentation/documentation/Foundation/NSUserActivity/activityType> property to determine which scene to create. For more information about configuring your app for different scene types, see [Specifying the scenes your app supports](/documentation/UIKit/specifying-the-scenes-your-app-supports). For information about creating multiple windows programmatically, see [Supporting multiple windows on iPad](/documentation/UIKit/supporting-multiple-windows-on-ipad).
### Configure your window scene
UIKit creates a [`UIWindowScene`](/documentation/UIKit/UIWindowScene) object for each scene instance. When configuring scene support, specify `UIWindowScene` objects rather than [`UIScene`](/documentation/UIKit/UIScene) objects. If your app adopts scenes for CarPlay, use <doc://com.apple.documentation/documentation/CarPlay/CPTemplateApplicationScene> instead. To learn how to add a CarPlay scene, see <doc://com.apple.documentation/documentation/CarPlay/displaying-content-in-carplay>.
If you load your root view controller from the storyboard, include the storyboard name in the <doc://com.apple.documentation/documentation/BundleResources/Information-Property-List/UIApplicationSceneManifest/UISceneConfigurations> key in your information property list scene manifest. The system automatically configures your window scene and root view controller.
If you load your root view controller programmatically, implement [`scene(_:willConnectTo:options:)`](/documentation/UIKit/UISceneDelegate/scene(_:willConnectTo:options:)) to create a [`UIWindow`](/documentation/UIKit/UIWindow) and associate it with the scene:
```swift
import UIKit
class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    var window: UIWindow?
    func scene(
        _ scene: UIScene,
        willConnectTo session: UISceneSession,
        options connectionOptions: UIScene.ConnectionOptions
    ) {
        guard let windowScene = scene as? UIWindowScene else { return }
        window = UIWindow(windowScene: windowScene)
        window?.rootViewController = YourRootViewController()
        window?.makeKeyAndVisible()
    }
}
```
`SceneDelegate` is a [`UIResponder`](/documentation/UIKit/UIResponder) subclass that conforms to [`UIWindowSceneDelegate`](/documentation/UIKit/UIWindowSceneDelegate). For more information about preparing your app at launch time, see [Responding to the launch of your app](/documentation/UIKit/responding-to-the-launch-of-your-app).
## Migrate app life-cycle logic
Move your app’s existing life-cycle methods from [`UIApplicationDelegate`](/documentation/UIKit/UIApplicationDelegate) to [`UISceneDelegate`](/documentation/UIKit/UISceneDelegate):
|UIApplicationDelegate                                                                                 |UISceneDelegate                                                                           |
|------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
|``doc://com.apple.uikit/documentation/UIKit/UIApplicationDelegate/applicationDidBecomeActive(_:)``    |``doc://com.apple.uikit/documentation/UIKit/UISceneDelegate/sceneDidBecomeActive(_:)``    |
|``doc://com.apple.uikit/documentation/UIKit/UIApplicationDelegate/applicationWillResignActive(_:)``   |``doc://com.apple.uikit/documentation/UIKit/UISceneDelegate/sceneWillResignActive(_:)``   |
|``doc://com.apple.uikit/documentation/UIKit/UIApplicationDelegate/applicationDidEnterBackground(_:)`` |``doc://com.apple.uikit/documentation/UIKit/UISceneDelegate/sceneDidEnterBackground(_:)`` |
|``doc://com.apple.uikit/documentation/UIKit/UIApplicationDelegate/applicationWillEnterForeground(_:)``|``doc://com.apple.uikit/documentation/UIKit/UISceneDelegate/sceneWillEnterForeground(_:)``|
> Important:
> Once your app adopts the scene-based life cycle, UIKit no longer calls these ``doc://com.apple.uikit/documentation/UIKit/UIApplicationDelegate`` methods, even if you keep them implemented. Each scene delegate method reflects only that scene’s own transitions. If you need to track your app’s aggregate state across all of its scenes (``doc://com.apple.uikit/documentation/UIKit/UIApplication/applicationState``), such as whether any scene is in the foreground, use the corresponding ``doc://com.apple.uikit/documentation/UIKit/UIApplication`` notification instead of relying on a single scene delegate callback. For example, use ``doc://com.apple.uikit/documentation/UIKit/UIApplication/willEnterForegroundNotification``.
After migrating, test your app in Full Screen Apps, Windowed Apps, and Stage Manager on iPad. To learn how to respond to state transitions, see [Managing your app’s life cycle](/documentation/UIKit/managing-your-app-s-life-cycle).
## Support noninteractive external display scenes
If your app doesn’t present custom content on an external display, you don’t need to configure anything for this scene role. When an external display connects, the system mirrors your app’s primary display, or, on compatible iPad models with extended display enabled, presents your app’s interactive windows on the external display.
In iOS 26 and earlier, the system connected a [`windowExternalDisplayNonInteractive`](/documentation/UIKit/UISceneSession/Role-swift.struct/windowExternalDisplayNonInteractive) scene automatically, and your app opted out by declining to provide content for it. Beginning in iOS 27, your app receives this scene only after it registers a *scene accessory*. A scene accessory declares supplementary content that the system presents on your app’s behalf when associated functionality becomes available, such as an external display connected by cable or AirPlay. Your app declares what content to provide, and the system decides when and where to present it. Because the content appears only when a display is available, design your app to remain fully functional without the external display.
If your app previously provided content for this scene by checking the connecting scene’s role in [`application(_:configurationForConnecting:options:)`](/documentation/UIKit/UIApplicationDelegate/application(_:configurationForConnecting:options:)) or your scene delegate, remove that role-specific logic and register a scene accessory instead. You can reuse your existing scene delegate as the accessory’s delegate class, or create a new one dedicated to the external-display scene.
Register the accessory on a view controller in your app’s main interface. Choose the view controller whose content the external display supplements:
```swift
class PlayerViewController: UIViewController {
    var displayRegistration: UISceneAccessoryRegistration?
    override func viewDidLoad() {
        super.viewDidLoad()
        // Describe the scene to present, including the delegate that attaches its window.
        let configuration = UISceneConfiguration()
        configuration.delegateClass = ExternalDisplaySceneDelegate.self
        // Register the accessory so the system can present this content on an external display.
        let accessory = UISceneAccessory.externalNonInteractive(sceneConfiguration: configuration)
        displayRegistration = registerSceneAccessory(accessory)
    }
}
```
While your app presents the view controller and an external display is available, the system connects the scene and calls [`scene(_:willConnectTo:options:)`](/documentation/UIKit/UISceneDelegate/scene(_:willConnectTo:options:)) on your delegate. Content for this scene spans the full screen.
For more information about presenting content on connected displays, including how to control when your content appears, respond to display availability, and stop presenting content, see [Presenting content on a connected display](/documentation/UIKit/presenting-content-on-a-connected-display).
---
Copyright &copy; 2026 Apple Inc. All rights reserved. | [Terms of Use](https://www.apple.com/legal/internet-services/terms/site.html) | [Privacy Policy](https://www.apple.com/privacy/privacy-policy)
