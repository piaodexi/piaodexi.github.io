---
title: Swift 다이나믹링크(Firebase) 사용방법
author: dexi
date: 2023-01-13 00:00:01 +0800
categories: [Blog, Swift]
tags: [Swift]
math: true
mermaid: true
futrue: true
published: true
---

# 딥 링크



  



- 앱의 특정 기능 또는 특정 화면에 도달 할 수 있는 링크 정보(URL)을 의미



  



# 다이나믹 링크



  



- 다이나믹 링크는 구글 파이어베이스에서 제공하는 서비스이며, 다이나믹 링크 또한 딥 링크이다.



- 기존 딥 링크의 경우 AOS, iOS에 따라 각각 따로 구현해야했음. 그렇지만 다이나믹 링크는 플랫폼 관계 없이 링크를 만들면 앱이 있다면 해당 콘텐츠로 이동하고, 없으면 스토어 이동하여 설치 후 해당 콘텐츠로 이동할 수 있게 하는 기술


# Xcode 셋팅


1. FireBase 프로젝트 -> 내 프로젝트 -> 프로젝트 설정 -> 내 앱 (Apple 앱) SDK 안내보기



2. GoogleService-Info.plist 다운로드후 Xcode내의 프로젝트에 불어넣기



3. SDK 안내보기의 가이드대로 Swift Package Manager를 사용해 Firebase 종속 항목을 설치하고 관리하거나 Podfile이 있는 경우 "pod 'FirebaseDynamicLinks'" 추가 후 pod update



4. 초기화 코드 추가 -> didFinishLaunchingWithOptions(AppDelegate)에서 "FirebaseApp.configure()" 코드 추가



5. Xcode에서 내프로젝트의 TARGETS에서 Capability -> Associated Domains 추가(Xcode 14.2 기준으로 두번 추가해줘야 release, debug 두개가 추가되고 하나로 합쳐짐.)



6. applink 기입 (YOUR_URL_PREFIX는 Firebase에 나와있는 prefix 중에서 `https://`를 제외한 도메인 값)


7. AppDegate, SceneDelegate에 Dynamic Link 수신 코드 작성


## AppDelegate (firebase 문서: https://firebase.google.com/docs/dynamic-links/ios/receive)



  (문서내용중



7. Next, in the application:continueUserActivity:restorationHandler: method, handle links received as Universal Links when the app is already installed:



8. Finally, in application:openURL:options: handle links received through your app's custom URL scheme. This method is called when your app is opened for the first time after installation. If the Dynamic Link isn't found on your app's first launch, this method will be called with the DynamicLink's url set to nil, indicating that the SDK failed to find a matching pending Dynamic Link.



8번에서 application:openURL:options: 이메소드는 앱을 설치한 후 처음 열 때 호출된다고 했지만 나의 경우엔 호출되지 않았고 application:continueUserActivity:restorationHandler: 여기서 호출되었음.



(Xcode 14.2, Swifte5 버전 기준)



)



  



```



func application(_ application: UIApplication, continue userActivity: NSUserActivity, restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {



// Dynamic Link 처리



//어플이 처음 실행될때



if let incomingURL = userActivity.webpageURL {



debugPrint("incomingURL : \(incomingURL)")



_ = DynamicLinks.dynamicLinks().handleUniversalLink(incomingURL) { dynamicLinks, error in



guard let realLinks = dynamicLinks?.url?.absoluteString, error == nil else {



debugPrint("### DynamicLinks error")



return



}



//어플이 처음 실행될때 userDefaults에 저장 (didFinishLaunchingWithOptions에서 유저디폴츠 초기화 필요)



LocalStorage.shared.setIsDynamicLinks(isDynamicLinks: realLinks)



}



} else {



//..



}



//background -> foreground 로 갈경우



let handled = DynamicLinks.dynamicLinks().handleUniversalLink(userActivity.webpageURL!) { dynamiclink, error in



guard let realLinks = dynamiclink?.url?.absoluteString



else {return}



debugPrint("### AppDelegate DynamicLink is app background -> foreground: \(realLinks)")



}



return handled



}



```



  



## SceneDelegate



  



```



func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {



// Use this method to optionally configure and attach the UIWindow `window` to the provided UIWindowScene `scene`.



// If using a storyboard, the `window` property will automatically be initialized and attached to the scene.



// This delegate does not imply the connecting scene or session are new (see `application:configurationForConnectingSceneSession` instead).



debugPrint("### willConnectTo")



//다이나믹웹링크 초기화



LocalStorage.shared.setIsDynamicLinks(isDynamicLinks: "")



guard let _ = (scene as? UIWindowScene) else {



return



}



if let userActivity = connectionOptions.userActivities.first {



self.scene(scene, continue: userActivity)



}



}



func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {



// Dynamic Link 처리



if let incomingURL = userActivity.webpageURL {



debugPrint("incomingURL : \(incomingURL)")



_ = DynamicLinks.dynamicLinks().handleUniversalLink(incomingURL) { dynamicLinks, error in



debugPrint("dynamicLinks : \(String(describing: dynamicLinks))")



  



guard let realLinks = dynamicLinks?.url?.absoluteString, error == nil else {



debugPrint("### DynamicLinks error")



return



}



debugPrint("### userActivity realLinks \(realLinks)")



//어플이 처음 실행될때 userDefaults에 저장 (위의 willConnectTo에서 유저디폴츠 초기화 필요)



LocalStorage.shared.setIsDynamicLinks(isDynamicLinks: realLinks);



  



//background -> foreground 로 갈경우 notificationCenter를 쓰거나 직접 프로퍼티에 접근하여 wiilset을 사용하거나 함.



//willset이나 notificationCenter나 초기실행시에는 쓸 수가 없었음.



viewController?.dynamicLinks = realLinks



}



}



}



}



```



## issue



- 다이나믹링크에 한글이 포함될 경우 appdelegate나 scenedelegate의 incomingUrl까지는 들어오나 dynamicLinks에서 nil이 떨어짐.   
  -> incomingUrl을 캐치해서 url끝에 ?d=1을 붙이고 워크플로어에서 디버깅해봐야함. icommingurl자체는 영어로 와도 ?d=1을 붙이고 플로우에서 "App withdeep link"클릭했을때 한글이 찍히면 안됨.      
  -> 인코딩을 해서 처리해볼 수는 있지만 백단에 영문으로 요청해서 처리하는걸로 감. 필요하다면 한글체크하여 인코딩 처리함.



- 다이나믹 링크를 타고 들어온 url에 html entity나 대체url이 포함되어 올경우 되지 않음   
  -> 대체  url의 경우 String의 removingPercentEncoding 메소드를 사용하여 처리, html entity의 경우 String을 확장하여 HTML코드를 표현문자로 변경하는걸로 처리(https://ios-development.tistory.com/485 참조)



- AppDelegate를 타는 구버전의 iphone의 경우 파이어베이스의 다이나믹 링크 수신받는 문서의 내용과 다름.   
  -> 디버깅하여 들어오는쪽에다가 작성..하여 해결



- AppDelegate를 타는 구버전의 iphone의 경우 폰에서 다이나믹 링크를 탔을때 어플이 분명 빌드되어 설치되었음에도 불구하고 appId가 null로 찍혀 앱스토어로 이동   
  -> Associated Domains에서 디버그모드만 추가되었는데 한번더 추가하여 release모드도 추가시킴으로 해결 


- 초기에는 notificationCenter로 처음 실행되었을때, 백그라운드에서 포어그라운드로 갔을때 모두 처리하려 하였으나.. 뷰컨트롤러의 노티피케이션이 생성전에 SceneDelegate나 Appdelegate에서 호출이 되어 사용불가   
  -> userDefaults 사용으로 해결 

