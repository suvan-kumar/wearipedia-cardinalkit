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
