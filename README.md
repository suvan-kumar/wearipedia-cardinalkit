
# CardinalKit Version for Wearipedia Apple HealthKit Notebook


Changes made from [CardinalKit](https://github.com/CardinalKit/CardinalKit) specifically for Wearipedia

1. Recasted stepCount and activityindex in HealthKitDataSync
    - Line 326 on HealthKitDataSync was changed. Initially, stepCount and activityindex were both doubles that was being recasted to Strings. This was changed to `"stepCount":Int(stepCount)` as well as `"activityindex":Int(value)`. The following was added as well: as `[String: Any]`. The full line modified is here for reference:  
    - `let dictionary = ["date":date.shortStringFromDate(),"activityindex":Int(value),"stepCount":Int(stepCount)] as [String : Any]`

2. Modifications to `fileprivate func sendHealthKit()`
	- `sendHealthKit()` in CKNetworkManager was originally writing each measurement from HealthKit into one collection. This collection stored every possible measurement, requiring iteration through the entire collection to accumulate one category of measurements. Recategorizing each measurement into their own collection (ex: HKQuantityTypeIdentifierStepCount, HKQuantityTypeIdentifierHeartRate, etc.) reduces the number of reads required by a very large margin and cuts down on operating costs for Firebase. The same amount of writes is used for this modification. The full updated function is below for reference:  
```    
    fileprivate func sendHealthKit(_ file: URL, _ package: Package, _ onCompletion: @escaping (Bool) -> Void) {
        do {
            let data = try Data(contentsOf: file)
            guard let json = try JSONSerialization.jsonObject(with: data, options: []) as? [String: Any],
                  let authPath = CKStudyUser.shared.authCollection else {
                onCompletion(false)
                return
            }
            
            guard let body = json["body"] as? [String: Any],
                  let collectionName = (body["quantity_type"])
            else {
                let collectionName = Constants.Firebase.dataBucketHealthKit
                return
            }
            let identifier = Date().startOfDay.shortStringFromDate() + "-\(package.fileName)"
            let trimmedIdentifier = identifier.trimmingCharacters(in: .whitespaces)
            
            let db=firestoreDb()
            db.collection(authPath + "\(collectionName)").document(trimmedIdentifier).setData(json) { err in
                
                if let err = err {
                    onCompletion(false)
                    print("Error writing document: \(err)")
                } else {
                    onCompletion(true)
                    print("[sendHealthKit] \(trimmedIdentifier) - successfully written!")
                }
            }
            
        } catch {
            print("Error \(error.localizedDescription)")
            onCompletion(false)
            return
        }
    }
```

3. Modifications to `fileprivate fun joinData()`
	- `joinData()` in HealthKitDataSync was preventing calling a function to accumulate the total step count for the day. This does not provide value for finding the correlation between step count and other data points, such as heart rate. Modifying the switch case to the code down below solved the issue and wrote the interval measurements to the HKQuantityTypeIdentifierStepCount collection in Firebase. The cumulative total is still being written in Firebase and can be found in users -> userID -> metrics -> stepCount
```
    fileprivate func JoinData(data: [[String: Any]])->[[String: Any]]{
        
        let firstElement = data.first
        if let element = firstElement,
           let body = element["body"] as? [String: Any]
        {
            if let quantityType = body["quantity_type"] as? String{
                switch(quantityType){
                case "HKQuantityTypeIdentifierActiveEnergyBurned":
                    return joinDataEnergyBurned(data: data)
                default:
                    return data
                }

            }

        }
        
        return data
        
    }
```


The following is a work in progress...the repo will be updated once it is complete

1. Enabling an option for removing the limitation of 1000 measurements per category written to Firebase
2. Fixing email and participant information storage in Firebase. The removal and maintenance of user information as fields in Firebase has bugs and is prone to removing information at the wrong time. 
=======
![CardinalKit Logo](https://raw.githubusercontent.com/CardinalKit/.github/main/assets/ck-header-light.png#gh-light-mode-only)
![CardinalKit Logo](https://raw.githubusercontent.com/CardinalKit/.github/main/assets/ck-header-dark.png#gh-dark-mode-only)

<!-- ALL-CONTRIBUTORS-BADGE:START - Do not remove or modify this section -->
[![All Contributors](https://img.shields.io/badge/all_contributors-8-orange.svg?style=flat-square)](#contributors-)
<!-- ALL-CONTRIBUTORS-BADGE:END --> 

---

<img src="https://raw.githubusercontent.com/CardinalKit/.github/main/assets/CK_Map.jpg" alt="cardinalkit map">

Includes:
* Informed consent process using ResearchKit.
* Track day-to-day adherence with CareKit.
* Monitor health data with HealthKit.
* Collect and upload EHR data.
* CoreMotion data demo.
* Awesome SwiftUI templates.
* Zero-code [customizable configuration file.](https://cardinalkit.org/docs/ckconfig)
* GCP Firebase Integration.

CardinalKit runs on iOS 15.0 and up. [Xcode 14](https://developer.apple.com/xcode/) is required for development.

## Build your App with CardinalKit

This repository contains a fully functional example in the `CardinalKit-Example` directory that you can use as a starting point for building your own app. To get started, clone this repository and follow our simple [setup instructions](https://cardinalkit.org/cardinalkit-docs/1-cardinalkit-app/1-start.html).

Feel free to join our Slack community or attend one of our workshops or buildathons for help customizing your app! Learn more at https://cardinalkit.org.

## Contribute to CardinalKit

Head on over to https://cardinalkit.org/ to get onboarded to our open source community ⚡️ 

## Contributors ✨

Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tr>
    <td align="center"><a href="http://gutierrezsantiago.com"><img src="https://avatars2.githubusercontent.com/u/5482213?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Santiago Gutierrez</b></sub></a><br /><a href="https://github.com/CardinalKit/CardinalKit/commits?author=ssgutierrez42" title="Code">💻</a></td>
    <td align="center"><a href="http://varunshenoy.com"><img src="https://avatars3.githubusercontent.com/u/10859091?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Varun Shenoy</b></sub></a><br /><a href="https://github.com/CardinalKit/CardinalKit/commits?author=varunshenoy" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/mhittle"><img src="https://avatars1.githubusercontent.com/u/1742619?v=4?s=100" width="100px;" alt=""/><br /><sub><b>mhittle</b></sub></a><br /><a href="#ideas-mhittle" title="Ideas, Planning, & Feedback">🤔</a> <a href="#maintenance-mhittle" title="Maintenance">🚧</a> <a href="#projectManagement-mhittle" title="Project Management">📆</a></td>
    <td align="center"><a href="https://github.com/aamirrasheed"><img src="https://avatars3.githubusercontent.com/u/7892721?v=4?s=100" width="100px;" alt=""/><br /><sub><b>aamirrasheed</b></sub></a><br /><a href="#content-aamirrasheed" title="Content">🖋</a> <a href="#video-aamirrasheed" title="Videos">📹</a></td>
    <td align="center"><a href="http://apollozhu.github.io/en"><img src="https://avatars1.githubusercontent.com/u/10842684?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Zhiyu Zhu/朱智语</b></sub></a><br /><a href="https://github.com/CardinalKit/CardinalKit/commits?author=ApolloZhu" title="Code">💻</a></td>
    <td align="center"><a href="http://vishnu.io"><img src="https://avatars.githubusercontent.com/u/1212163?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Vishnu Ravi</b></sub></a><br /><a href="https://github.com/CardinalKit/CardinalKit/commits?author=vishnuravi" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/griffinac"><img src="https://avatars0.githubusercontent.com/u/14243141?s=460&u=203a5408c41deaae65c2416b3043e777bdf9de0e&v=4?s=100" width="100px;" alt=""/><br /><sub><b>Ashley Griffin</b></sub></a><br /><a href="#ideas-griffinac" title="Ideas, Planning, & Feedback">🤔</a> <a href="https://github.com/CardinalKit/CardinalKit/commits?author=griffinac" title="Code">💻</a></td>
  </tr>
  <tr>
    <td align="center"><a href="http://ase.in.tum.de/schmiedmayer"><img src="https://avatars.githubusercontent.com/u/28656495?v=4?s=100" width="100px;" alt=""/><br /><sub><b>Paul Schmiedmayer</b></sub></a><br /><a href="https://github.com/CardinalKit/CardinalKit/commits?author=PSchmiedmayer" title="Code">💻</a></td>
  </tr>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind welcome!

## License

CardinalKit is available under the MIT license. See the LICENSE file for more info.


![Stanford Byers Center for Biodesign Logo](https://raw.githubusercontent.com/CardinalKit/.github/main/assets/ck-footer-light.png#gh-light-mode-only)
![Stanford Byers Center for Biodesign Logo](https://raw.githubusercontent.com/CardinalKit/.github/main/assets/ck-footer-dark.png#gh-dark-mode-only)
