# aws-s3-demo 


#### Task 1: Create a S3 Bucket 

* ecommerce-kv-b1-2022

| Step 1  | Step 2  | 
|---|---|
| ![image](https://user-images.githubusercontent.com/2763774/169633938-5a3c7ce6-d187-4299-bbcd-b9858400c4f5.png)  | ![image](https://user-images.githubusercontent.com/2763774/169633958-23ae0b8e-4dce-48c0-982b-519d8a586cca.png)  |

#### Task 2: View the bucket contents
| Step 1   | Step 2  | 
|---|---|
| ![image](https://user-images.githubusercontent.com/2763774/169634014-3fbf0dd0-f129-45ad-8dc6-241bb866b5bc.png)  | ![image](https://user-images.githubusercontent.com/2763774/169634037-d82f30fd-0681-4b0e-8cc6-5715a92bc69c.png)  |


#### Task 3: Upload a file

| Step 1  | Step 2  | 
|---|---|
|![image](https://user-images.githubusercontent.com/2763774/169634047-e14fc215-6fd7-4ea2-93f4-8adc945ecae2.png) | ![image](https://user-images.githubusercontent.com/2763774/169634070-af657b69-ee03-4edd-9f85-6edc9eb15945.png) |
| ![image](https://user-images.githubusercontent.com/2763774/169634083-e8ca0bad-e01b-4b34-9c6d-0208220fb40a.png) | ![image](https://user-images.githubusercontent.com/2763774/169634094-7e69d7b0-31a1-4a1c-aac5-567b89e1e521.png) |


#### Task 4: Access the Uploaded file

* Test URL - https://ecommerce-kv-b1-2022.s3.ap-south-1.amazonaws.com/m.jpeg
* 
| Step 1  |  |
|---|---|
| ![image](https://user-images.githubusercontent.com/2763774/169634109-9185fd2d-6394-4ae5-9399-a2b7ac54b825.png) | |

#### Task 5: Add Permission Policy

|   |   | 
|---|---|
| ![image](https://user-images.githubusercontent.com/2763774/169634128-0ea2482e-433f-4866-8567-919b9764b9c0.png) |![image](https://user-images.githubusercontent.com/2763774/169634241-d3f4012e-8a3a-4cd4-9759-60e82fd6a8eb.png) |

* **Add Policy**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::ecommerce-kv-b1-2022/*"
        }
    ]
}
```

* **Test URL** - https://ecommerce-kv-b1-2022.s3.ap-south-1.amazonaws.com/m.jpeg
![image](https://user-images.githubusercontent.com/2763774/169634254-45e96b32-f675-4a0d-a064-28142927ea04.png)


#### Task 6: Perform File Upload and Download using AWS SDK API
* Refer the separate article - https://github.com/nareshkumar-h/aws-s3-demo/blob/main/api.md
