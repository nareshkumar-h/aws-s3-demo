# AWS SDK S3 API


##### Task 1: Add AWS SDK Dependency

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
</dependency>
<dependency>
	<groupId>commons-io</groupId>
	<artifactId>commons-io</artifactId>
  <version>2.6</version>
</dependency>
```

```xml
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>software.amazon.awssdk</groupId>
				<artifactId>bom</artifactId>
				<version>2.17.195</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
```

##### Task 2: Add AWS Credentials in Spring Boot "application.properties"

```
AWS_ACCESS_KEY_ID=AKIA4CFB5WA643ASL35M
AWS_SECRET_ACCESS_KEY=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
S3_BUCKET_NAME=ecommerce-kv-b1-2022
```

##### Task 3: Create AWSConfig

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import software.amazon.awssdk.auth.credentials.AwsBasicCredentials;
import software.amazon.awssdk.auth.credentials.AwsCredentials;
import software.amazon.awssdk.auth.credentials.StaticCredentialsProvider;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.presigner.S3Presigner;

@Configuration
public class AWSConfig {

	@Value("${AWS_ACCESS_KEY_ID}")
	private String awsId;

	@Value("${AWS_SECRET_ACCESS_KEY}")
	private String awsKey;

	@Bean
	public S3Client s3client() {
		AwsCredentials credentials = AwsBasicCredentials.create(awsId, awsKey);
		S3Client s3client = S3Client.builder().credentialsProvider(StaticCredentialsProvider.create(credentials))
				.region(Region.AP_SOUTH_1).build();
		return s3client;
	}

	@Bean
	public S3Presigner s3Presigner() {
		AwsCredentials credentials = AwsBasicCredentials.create(awsId, awsKey);
		S3Presigner s3Presigner = S3Presigner.builder()
				.credentialsProvider(StaticCredentialsProvider.create(credentials)).region(Region.AP_SOUTH_1).build();
		return s3Presigner;
	}
}
```

##### Task 4: Upload File API

```java
@Service
public class AwsS3Service {

	private Logger logger = LoggerFactory.getLogger(AwsS3Service.class);

	@Autowired
	private S3Client s3;
	

}
```

* **Add a method to upload the file to s3 bucket**

```java

	public String uploadFileContentAsFile(String keyName, MultipartFile file) {

		String url = null;
		try {
			PutObjectRequest request = PutObjectRequest.builder().bucket(bucketName).key(keyName).build();

			RequestBody body = RequestBody.fromBytes(file.getBytes());
//			RequestBody body = RequestBody.fromInputStream(file.getInputStream(), file.getSize());
			PutObjectResponse response = s3.putObject(request, body);
			logger.debug("Upload File content as bytes:" + response);

			GetUrlRequest request2 = GetUrlRequest.builder().bucket(bucketName).key(keyName).build();
			url = s3.utilities().getUrl(request2).toExternalForm();

		} catch (Exception e) {
			e.printStackTrace();
		}
		return url;

	}
```

##### Task 5: Develop REST API

```
@RestController
@RequestMapping("api/s3")
public class S3Controller {
	
	@Autowired
	AwsS3Service s3Service;
  
  
}
```

##### Task 6: Add a method to upload api

```java
@PostMapping("upload")
	public String uploadFile(@RequestParam("fileName") String fileName, @RequestBody MultipartFile file ) {		
		String fileNameUrl = s3Service.uploadFileContentAsFile(fileName, file);
		return fileNameUrl;
	}
```
