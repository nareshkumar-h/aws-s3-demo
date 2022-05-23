#### AWSS3Config.java
```java
package in.todoapp.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import software.amazon.awssdk.auth.credentials.AwsBasicCredentials;
import software.amazon.awssdk.auth.credentials.AwsCredentials;
import software.amazon.awssdk.auth.credentials.StaticCredentialsProvider;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.services.iam.IamClient;
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

#### application.properties

```
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
S3_BUCKET_NAME=your-bucket-name
```

#### S3Controller

```java
package in.todoapp.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import in.todoapp.service.S3Service;

@RestController
@RequestMapping("api/s3")
public class S3Controller {
	
	@Autowired
	S3Service s3Service;
	
	
	@GetMapping("download/content")
	public Object getFileContentAsString(@RequestParam("fileName") String fileName) {
		
		return s3Service.getContentAsString(fileName);
	}
	
	@PostMapping("upload/content")
	public void uploadContent(@RequestParam("fileName") String fileName, @RequestBody String content) {
		
		s3Service.uploadFileContentAsString(fileName, content);
	}
	
	@PostMapping("upload/file")
	public String uploadFile(@RequestParam("fileName") String fileName, @RequestBody MultipartFile file ) {
		System.out.println(file);
		String fileNameUrl = s3Service.uploadFileContentAsFile(fileName, file);
		return fileNameUrl;
	}
	
	@GetMapping("download/file")
	public Object downloadFileContent(@RequestParam("fileName") String fileName) {
		
		return s3Service.downloadFileContentAsFile(fileName);
	}
	
	@GetMapping("files")
	public Object getFiles() {
		
		return s3Service.getFiles("");
	}
	
	@GetMapping("deletefile")
	public void deleteFileContent(@RequestParam("fileName") String fileName) {
		
		s3Service.deleteFile(fileName);
	}
	
	@GetMapping("presignurl")
	public Object getPresignUrl(@RequestParam("fileName") String fileName) {
		
		return s3Service.getPresignedUrl(fileName);
	}
	
	@GetMapping("download/pdf")
	public ResponseEntity<?> downloadAsFile(@RequestParam("fileName") String fileName) {
		
		byte[] bytes = s3Service.downloadFileContentAsFile(fileName);
		HttpHeaders httpHeaders = new HttpHeaders();
		httpHeaders.setContentType(MediaType.APPLICATION_PDF);
		httpHeaders.setContentLength(bytes.length);
		httpHeaders.setContentDispositionFormData("attachment", fileName);
		return new ResponseEntity<byte[]>(bytes, httpHeaders, HttpStatus.OK);
	}
}

```

##### S3Service.java
```java
package in.todoapp.service;

import java.net.URL;
import java.time.Duration;
import java.util.ArrayList;
import java.util.List;

import org.apache.commons.io.IOUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import in.todoapp.service.AwsS3Service;
import software.amazon.awssdk.core.ResponseInputStream;
import software.amazon.awssdk.core.sync.RequestBody;
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.services.s3.model.DeleteObjectRequest;
import software.amazon.awssdk.services.s3.model.DeleteObjectResponse;
import software.amazon.awssdk.services.s3.model.GetObjectRequest;
import software.amazon.awssdk.services.s3.model.GetObjectResponse;
import software.amazon.awssdk.services.s3.model.GetUrlRequest;
import software.amazon.awssdk.services.s3.model.ListObjectsRequest;
import software.amazon.awssdk.services.s3.model.ListObjectsResponse;
import software.amazon.awssdk.services.s3.model.PutObjectRequest;
import software.amazon.awssdk.services.s3.model.PutObjectResponse;
import software.amazon.awssdk.services.s3.model.S3Object;
import software.amazon.awssdk.services.s3.presigner.S3Presigner;
import software.amazon.awssdk.services.s3.presigner.model.GetObjectPresignRequest;
import software.amazon.awssdk.services.s3.presigner.model.PresignedGetObjectRequest;

@Service
public class S3Service {

	private Logger logger = LoggerFactory.getLogger(AwsS3Service.class);

	@Autowired
	private S3Client s3;
	
	@Autowired
	private S3Presigner s3Presigner;

	@Value("${S3_BUCKET_NAME}")
	private String bucketName;

	@Override
	public String getContentAsString(String keyName) {
		String content = null;
		try {
			GetObjectRequest getObjectRequest = GetObjectRequest.builder().bucket(bucketName).key(keyName).build();

			ResponseInputStream<GetObjectResponse> responseInputStream = s3.getObject(getObjectRequest);
			byte[] bytes = IOUtils.toByteArray(responseInputStream);

			content = new String(bytes);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return content;
	}

	@Override
	public String uploadFileContentAsString(String keyName, String content) {

		String url = null;
		try {
			PutObjectRequest request = PutObjectRequest.builder().bucket(bucketName).key(keyName).build();

			RequestBody body = RequestBody.fromString(content);
			PutObjectResponse response = s3.putObject(request, body);
			logger.debug("Upload File content as string:" + response);

			GetUrlRequest request2 = GetUrlRequest.builder().bucket(bucketName).key(keyName).build();
			url = s3.utilities().getUrl(request2).toExternalForm();

		} catch (Exception e) {
			e.printStackTrace();
		}
		return url;

	}

	@Override
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

	@Override
	public byte[] downloadFileContentAsFile(String key) {

		byte[] content = null;
		try {
			GetObjectRequest getObjectRequest = GetObjectRequest.builder().bucket(bucketName).key(key).build();

			ResponseInputStream<GetObjectResponse> responseInputStream = s3.getObject(getObjectRequest);
			content = responseInputStream.readAllBytes();

		} catch (Exception e) {
			e.printStackTrace();
		}
		return content;

	}

	@Override
	public void deleteFile(String keyName) {

		try {
			DeleteObjectRequest request = DeleteObjectRequest.builder().bucket(bucketName).key(keyName).build();

			DeleteObjectResponse response = s3.deleteObject(request);
			logger.debug("Delete File:" + response);

		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	@Override
	public List<String> getFiles(String keyName) {

		List<String> files = new ArrayList<>();
		try {
			ListObjectsRequest request = ListObjectsRequest.builder().bucket(bucketName).build();

			ListObjectsResponse response = s3.listObjects(request);
			logger.debug("List Files:" + response);
			List<S3Object> contents = response.contents();
			for (S3Object s3Object : contents) {
				System.out.println(s3Object);
				files.add(s3Object.key());
			}

		} catch (Exception e) {
			e.printStackTrace();
		}
		return files;
	}

	@Override
	public String getPresignedUrl(String keyName) {

		String fileUrl = null;
		try {
			GetObjectRequest getObjectRequest = GetObjectRequest.builder().bucket(bucketName).key(keyName).build();

			GetObjectPresignRequest getObjectPresignRequest = GetObjectPresignRequest.builder()
					.signatureDuration(Duration.ofMinutes(1)).getObjectRequest(getObjectRequest).build();

			// Generate the presigned request
			PresignedGetObjectRequest presignedGetObjectRequest = s3Presigner.presignGetObject(getObjectPresignRequest);

			URL url = presignedGetObjectRequest.url();
			fileUrl = url.toExternalForm();

		} catch (Exception e) {
			e.printStackTrace();
		}
		return fileUrl;
	}

}
```

##### Postman - API Testing
![image](https://user-images.githubusercontent.com/2763774/169839576-ad1acac7-bf09-4a9f-923f-5a45a8c0b9f4.png)
