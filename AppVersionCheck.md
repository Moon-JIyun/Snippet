## App version checker

```swift
import Foundation
import Alamofire

class VersionCheck {

    public static let shared = VersionCheck()

    var newVersionAvailable: Bool?
    var appStoreVersion: String?

    func checkAppStore(callback: ((_ versionAvailable: Bool, _ version: String?)->Void)? = nil) {
        
        let ourBundleId = Bundle.main.infoDictionary!["CFBundleIdentifier"] as! String
        print("ourBundleId: ", ourBundleId)
        AF.request("https://itunes.apple.com/kr/lookup?bundleId=\(ourBundleId)").responseJSON { response in
            var shouldUpdateApp: Bool = false
            var versionStr: String?
            if let json = response.value as? NSDictionary,
               let results = json["results"] as? NSArray,
               let entry = results.firstObject as? NSDictionary,
               let appVersionOnMarket = entry["version"] as? String,
               let ourVersion =  Bundle.main.infoDictionary?["CFBundleShortVersionString"] as? String
            {
                
                // 오름차순이면 앱 업데이트를 해야한다.
                let compareResult = ourVersion.versionCompare(appVersionOnMarket)
                print(#file, #function, #line, "compareResult: \(self.getCompareResult(compareResult))")
                
                shouldUpdateApp = self.checkShouldUpdateApp(compareResult)
                    
                print("[앱 강제 업데이트 테스트] ourVersion: \(ourVersion)")
                print("[앱 강제 업데이트 테스트] appVersionOnMarket: \(appVersionOnMarket)")
                
                print("[앱 강제 업데이트 테스트] 앱 업데이트를 해야하는지 여부 shouldUpdateApp: \(shouldUpdateApp)")
                
                AppConfig.shared.updateAppVersionData(to: (versionOnTheMarket: appVersionOnMarket, localVersion: ourVersion))
                versionStr = appVersionOnMarket
            }

            self.appStoreVersion = versionStr
            self.newVersionAvailable = shouldUpdateApp
            callback?(shouldUpdateApp, versionStr)
        }
    }
    
    fileprivate func getCompareResult(_ result: ComparisonResult) -> String {
        print(#file, #function, #line, "")
        switch result {
        case .orderedSame: return "일치한다."
        case .orderedDescending: return "내림차순이다."
        case .orderedAscending: return "오름차순이다."
        }
    }
    
    fileprivate func checkShouldUpdateApp(_ result: ComparisonResult) -> Bool {
        print(#file, #function, #line, "")
        return result == .orderedAscending
    }
    
}
```

### example
```swift
  // 앱 버전 체크하기
  VersionCheck.shared.checkAppStore{ shouldUpdateApp, version in
      print("IS NEW VERSION AVAILABLE: shouldUpdateApp : \(shouldUpdateApp), APP STORE VERSION: \(String(describing: version))")
      if shouldUpdateApp { // 앱 업데이트 있음
          self.alertInfo = JsAlert("앱 업데이트가 있습니다.", "확인")
          return
      } else { // 앱 업데이트 없음
          DispatchQueue.main.asyncAfter(deadline: .now() + 2.1) { 
             /// 원하는 로직 처리
          }
      }
  }
```
