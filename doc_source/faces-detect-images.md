# Detecting Faces in an Image<a name="faces-detect-images"></a>

Amazon Rekognition Image provides the [DetectFaces](API_DetectFaces.md) operation that looks for key facial features such as eyes, nose, and mouth to detect faces in an input image\. Amazon Rekognition Image detects the 100 largest faces in an image\.

You can provide the input image as an image byte array \(base64\-encoded image bytes\), or specify an Amazon S3 object\. In this procedure, you upload an image \(JPEG or PNG\) to your S3 bucket and specify the object key name\.

**To detect faces in an image**

1. If you haven't already:

   1. Create or update an IAM user with `AmazonRekognitionFullAccess` and `AmazonS3ReadOnlyAccess` permissions\. For more information, see [Step 1: Set Up an AWS Account and Create an IAM User](setting-up.md#setting-up-iam)\.

   1. Install and configure the AWS CLI and the AWS SDKs\. For more information, see [Step 2: Set Up the AWS CLI and AWS SDKs](setup-awscli-sdk.md)\.

1. Upload an image \(that contains one or more faces\) to your S3 bucket\. 

   For instructions, see [Uploading Objects into Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/UploadingObjectsintoAmazonS3.html) in the *Amazon Simple Storage Service Console User Guide*\.

1. Use the following examples to call `DetectFaces`\.

------
#### [ Java ]

   This example displays the estimated age range for detected faces, and lists the JSON for all detected facial attributes\. Change the value of `photo` to the image file name\. Change the value of `bucket` to the Amazon S3 bucket where the image is stored\.

   ```
   //Copyright 2018 Amazon.com, Inc. or its affiliates. All Rights Reserved.
   //PDX-License-Identifier: MIT-0 (For details, see https://github.com/awsdocs/amazon-rekognition-developer-guide/blob/master/LICENSE-SAMPLECODE.)
   
   package aws.example.rekognition.image;
   
   import com.amazonaws.services.rekognition.AmazonRekognition;
   import com.amazonaws.services.rekognition.AmazonRekognitionClientBuilder;
   import com.amazonaws.services.rekognition.model.AmazonRekognitionException;
   import com.amazonaws.services.rekognition.model.Image;
   import com.amazonaws.services.rekognition.model.S3Object;
   import com.amazonaws.services.rekognition.model.AgeRange;
   import com.amazonaws.services.rekognition.model.Attribute;
   import com.amazonaws.services.rekognition.model.DetectFacesRequest;
   import com.amazonaws.services.rekognition.model.DetectFacesResult;
   import com.amazonaws.services.rekognition.model.FaceDetail;
   import com.fasterxml.jackson.databind.ObjectMapper;
   import java.util.List;
   
   
   public class DetectFaces {
      
      
      public static void main(String[] args) throws Exception {
   
         String photo = "input.jpg";
         String bucket = "bucket";
   
         AmazonRekognition rekognitionClient = AmazonRekognitionClientBuilder.defaultClient();
   
   
         DetectFacesRequest request = new DetectFacesRequest()
            .withImage(new Image()
               .withS3Object(new S3Object()
                  .withName(photo)
                  .withBucket(bucket)))
            .withAttributes(Attribute.ALL);
         // Replace Attribute.ALL with Attribute.DEFAULT to get default values.
   
         try {
            DetectFacesResult result = rekognitionClient.detectFaces(request);
            List < FaceDetail > faceDetails = result.getFaceDetails();
   
            for (FaceDetail face: faceDetails) {
               if (request.getAttributes().contains("ALL")) {
                  AgeRange ageRange = face.getAgeRange();
                  System.out.println("The detected face is estimated to be between "
                     + ageRange.getLow().toString() + " and " + ageRange.getHigh().toString()
                     + " years old.");
                  System.out.println("Here's the complete set of attributes:");
               } else { // non-default attributes have null values.
                  System.out.println("Here's the default set of attributes:");
               }
   
               ObjectMapper objectMapper = new ObjectMapper();
               System.out.println(objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(face));
            }
   
         } catch (AmazonRekognitionException e) {
            e.printStackTrace();
         }
   
      }
   
   }
   ```

------
#### [ AWS CLI ]

   This example displays the JSON output from the `detect-faces` AWS CLI operation\. Replace `file` with the name of an image file\. Replace `bucket` with the name of the Amazon S3 bucket that contains the image file\.

   ```
   aws rekognition detect-faces \
   --image '{"S3Object":{"Bucket":"bucket","Name":"file"}}' \
   --attributes "ALL"
   ```

------
#### [ Python ]

   This example displays the estimated age range for detected faces, and lists the JSON for all detected facial attributes\. Change the value of `photo` to the image file name\. Change the value of `bucket` to the Amazon S3 bucket where the image is stored\.

   ```
   #Copyright 2018 Amazon.com, Inc. or its affiliates. All Rights Reserved.
   #PDX-License-Identifier: MIT-0 (For details, see https://github.com/awsdocs/amazon-rekognition-developer-guide/blob/master/LICENSE-SAMPLECODE.)
   
   import boto3
   import json
   
   if __name__ == "__main__":
       photo='input.jpg'
       bucket='bucket'
       client=boto3.client('rekognition')
   
       response = client.detect_faces(Image={'S3Object':{'Bucket':bucket,'Name':photo}},Attributes=['ALL'])
   
       print('Detected faces for ' + photo)    
       for faceDetail in response['FaceDetails']:
           print('The detected face is between ' + str(faceDetail['AgeRange']['Low']) 
                 + ' and ' + str(faceDetail['AgeRange']['High']) + ' years old')
           print('Here are the other attributes:')
           print(json.dumps(faceDetail, indent=4, sort_keys=True))
   ```

------
#### [ \.NET ]

   This example displays the estimated age range for detected faces, and lists the JSON for all detected facial attributes\. Change the value of `photo` to the image file name\. Change the value of `bucket` to the Amazon S3 bucket where the image is stored\.

   ```
   //Copyright 2018 Amazon.com, Inc. or its affiliates. All Rights Reserved.
   //PDX-License-Identifier: MIT-0 (For details, see https://github.com/awsdocs/amazon-rekognition-developer-guide/blob/master/LICENSE-SAMPLECODE.)
   
   using System;
   using System.Collections.Generic;
   using Amazon.Rekognition;
   using Amazon.Rekognition.Model;
   
   public class DetectFaces
   {
       public static void Example()
       {
           String photo = "input.jpg";
           String bucket = "bucket";
   
           AmazonRekognitionClient rekognitionClient = new AmazonRekognitionClient();
   
           DetectFacesRequest detectFacesRequest = new DetectFacesRequest()
           {
               Image = new Image()
               {
                   S3Object = new S3Object()
                   {
                       Name = photo,
                       Bucket = bucket
                   },
               },
               // Attributes can be "ALL" or "DEFAULT". 
               // "DEFAULT": BoundingBox, Confidence, Landmarks, Pose, and Quality.
               // "ALL": See https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/Rekognition/TFaceDetail.html
               Attributes = new List<String>() { "ALL" }
           };
   
           try
           {
               DetectFacesResponse detectFacesResponse = rekognitionClient.DetectFaces(detectFacesRequest);
               bool hasAll = detectFacesRequest.Attributes.Contains("ALL");
               foreach(FaceDetail face in detectFacesResponse.FaceDetails)
               {
                   Console.WriteLine("BoundingBox: top={0} left={1} width={2} height={3}", face.BoundingBox.Left,
                       face.BoundingBox.Top, face.BoundingBox.Width, face.BoundingBox.Height);
                   Console.WriteLine("Confidence: {0}\nLandmarks: {1}\nPose: pitch={2} roll={3} yaw={4}\nQuality: {5}",
                       face.Confidence, face.Landmarks.Count, face.Pose.Pitch,
                       face.Pose.Roll, face.Pose.Yaw, face.Quality);
                   if (hasAll)
                       Console.WriteLine("The detected face is estimated to be between " +
                           face.AgeRange.Low + " and " + face.AgeRange.High + " years old.");
               }
           }
           catch (Exception e)
           {
               Console.WriteLine(e.Message);
           }
       }
   }
   ```

------

## DetectFaces Operation Request<a name="detectfaces-request"></a>

The input to `DetectFaces` is an image\. In this example, the image is loaded from an Amazon S3 bucket\. The `Attributes` parameter specifies that all facial attributes should be returned\. For more information, see [Working with Images](images.md)\.

```
{
    "Image": {
        "S3Object": {
            "Bucket": "bucket",
            "Name": "input.jpg"
        }
    },
    "Attributes": [
        "ALL"
    ]
}
```

## DetectFaces Operation Response<a name="detectfaces-response"></a>

 `DetectFaces` returns the following information for each detected face:
+ **Bounding box** – The coordinates of the bounding box that surrounds the face\.
+ **Confidence** – The level of confidence that the bounding box contains a face\. 
+ **Facial landmarks** – An array of facial landmarks\. For each landmark \(such as the left eye, right eye, and mouth\), the response provides the x and y coordinates\.
+ **Facial attributes** – A set of facial attributes, including gender, or whether the face has a beard\. For each such attribute, the response provides a value\. The value can be of different types, such as a Boolean type \(whether a person is wearing sunglasses\) or a string \(whether the person is male or female\)\. In addition, for most attributes, the response also provides a confidence in the detected value for the attribute\. 
+ **Quality** – Describes the brightness and the sharpness of the face\. For information about ensuring the best possible face detection, see [Recommendations for Facial Recognition Input Images](recommendations-facial-input-images.md)\.
+ **Pose** – Describes the rotation of the face inside the image\.
+ **Emotions** – A set of emotions with confidence in the analysis\.

The following is an example response of a `DetectFaces` API call\. 

```
{
    "FaceDetails": [
        {
            "AgeRange": {
                "High": 43,
                "Low": 26
            },
            "Beard": {
                "Confidence": 97.48941802978516,
                "Value": true
            },
            "BoundingBox": {
                "Height": 0.6968063116073608,
                "Left": 0.26937249302864075,
                "Top": 0.11424895375967026,
                "Width": 0.42325547337532043
            },
            "Confidence": 99.99995422363281,
            "Emotions": [
                {
                    "Confidence": 0.042965151369571686,
                    "Type": "DISGUSTED"
                },
                {
                    "Confidence": 0.002022328320890665,
                    "Type": "HAPPY"
                },
                {
                    "Confidence": 0.4482877850532532,
                    "Type": "SURPRISED"
                },
                {
                    "Confidence": 0.007082826923578978,
                    "Type": "ANGRY"
                },
                {
                    "Confidence": 0,
                    "Type": "CONFUSED"
                },
                {
                    "Confidence": 99.47616577148438,
                    "Type": "CALM"
                },
                {
                    "Confidence": 0.017732391133904457,
                    "Type": "SAD"
                }
            ],
            "Eyeglasses": {
                "Confidence": 99.42405700683594,
                "Value": false
            },
            "EyesOpen": {
                "Confidence": 99.99604797363281,
                "Value": true
            },
            "Gender": {
                "Confidence": 99.722412109375,
                "Value": "Male"
            },
            "Landmarks": [
                {
                    "Type": "eyeLeft",
                    "X": 0.38549351692199707,
                    "Y": 0.3959200084209442
                },
                {
                    "Type": "eyeRight",
                    "X": 0.5773905515670776,
                    "Y": 0.394561767578125
                },
                {
                    "Type": "mouthLeft",
                    "X": 0.40410104393959045,
                    "Y": 0.6479480862617493
                },
                {
                    "Type": "mouthRight",
                    "X": 0.5623446702957153,
                    "Y": 0.647117555141449
                },
                {
                    "Type": "nose",
                    "X": 0.47763553261756897,
                    "Y": 0.5337067246437073
                },
                {
                    "Type": "leftEyeBrowLeft",
                    "X": 0.3114689588546753,
                    "Y": 0.3376390337944031
                },
                {
                    "Type": "leftEyeBrowRight",
                    "X": 0.4224424660205841,
                    "Y": 0.3232649564743042
                },
                {
                    "Type": "leftEyeBrowUp",
                    "X": 0.36654090881347656,
                    "Y": 0.3104579746723175
                },
                {
                    "Type": "rightEyeBrowLeft",
                    "X": 0.5353175401687622,
                    "Y": 0.3223199248313904
                },
                {
                    "Type": "rightEyeBrowRight",
                    "X": 0.6546239852905273,
                    "Y": 0.3348073363304138
                },
                {
                    "Type": "rightEyeBrowUp",
                    "X": 0.5936762094497681,
                    "Y": 0.3080498278141022
                },
                {
                    "Type": "leftEyeLeft",
                    "X": 0.3524211347103119,
                    "Y": 0.3936865031719208
                },
                {
                    "Type": "leftEyeRight",
                    "X": 0.4229775369167328,
                    "Y": 0.3973258435726166
                },
                {
                    "Type": "leftEyeUp",
                    "X": 0.38467878103256226,
                    "Y": 0.3836822807788849
                },
                {
                    "Type": "leftEyeDown",
                    "X": 0.38629674911499023,
                    "Y": 0.40618783235549927
                },
                {
                    "Type": "rightEyeLeft",
                    "X": 0.5374732613563538,
                    "Y": 0.39637991786003113
                },
                {
                    "Type": "rightEyeRight",
                    "X": 0.609208345413208,
                    "Y": 0.391626238822937
                },
                {
                    "Type": "rightEyeUp",
                    "X": 0.5750962495803833,
                    "Y": 0.3821527063846588
                },
                {
                    "Type": "rightEyeDown",
                    "X": 0.5740782618522644,
                    "Y": 0.40471214056015015
                },
                {
                    "Type": "noseLeft",
                    "X": 0.4441811740398407,
                    "Y": 0.5608476400375366
                },
                {
                    "Type": "noseRight",
                    "X": 0.5155643820762634,
                    "Y": 0.5569332242012024
                },
                {
                    "Type": "mouthUp",
                    "X": 0.47968366742134094,
                    "Y": 0.6176465749740601
                },
                {
                    "Type": "mouthDown",
                    "X": 0.4807897210121155,
                    "Y": 0.690782368183136
                },
                {
                    "Type": "leftPupil",
                    "X": 0.38549351692199707,
                    "Y": 0.3959200084209442
                },
                {
                    "Type": "rightPupil",
                    "X": 0.5773905515670776,
                    "Y": 0.394561767578125
                },
                {
                    "Type": "upperJawlineLeft",
                    "X": 0.27245330810546875,
                    "Y": 0.3902156949043274
                },
                {
                    "Type": "midJawlineLeft",
                    "X": 0.31561678647994995,
                    "Y": 0.6596118807792664
                },
                {
                    "Type": "chinBottom",
                    "X": 0.48385748267173767,
                    "Y": 0.8160444498062134
                },
                {
                    "Type": "midJawlineRight",
                    "X": 0.6625112891197205,
                    "Y": 0.656606137752533
                },
                {
                    "Type": "upperJawlineRight",
                    "X": 0.7042999863624573,
                    "Y": 0.3863988518714905
                }
            ],
            "MouthOpen": {
                "Confidence": 99.83820343017578,
                "Value": false
            },
            "Mustache": {
                "Confidence": 72.20288848876953,
                "Value": false
            },
            "Pose": {
                "Pitch": -4.970901966094971,
                "Roll": -1.4911699295043945,
                "Yaw": -10.983647346496582
            },
            "Quality": {
                "Brightness": 73.81391906738281,
                "Sharpness": 86.86019134521484
            },
            "Smile": {
                "Confidence": 99.93638610839844,
                "Value": false
            },
            "Sunglasses": {
                "Confidence": 99.81478881835938,
                "Value": false
            }
        }
    ]
}
```

Note the following:
+ The `Pose` data describes the rotation of the face detected\. You can use the combination of the `BoundingBox` and `Pose` data to draw the bounding box around faces that your application displays\.
+ The `Quality` describes the brightness and the sharpness of the face\. You might find this useful to compare faces across images and find the best face\.
+ The preceding response shows all facial `landmarks` the service can detect, all facial attributes and emotions\. To get all of these in the response, you must specify the `attributes` parameter with value `ALL`\. By default, the `DetectFaces` API returns only the following five facial attributes: `BoundingBox`, `Confidence`, `Pose`, `Quality` and `landmarks`\. The default landmarks returned are: `eyeLeft`, `eyeRight`, `nose`, `mouthLeft`, and `mouthRight`\. 
+ The following illustration shows the relative location of the facial landmarks\(`Landmarks`\) on the face that are returned by the `DetectFaces` API operation\.  
![\[Image NOT FOUND\]](http://docs.aws.amazon.com/rekognition/latest/dg/images/landmarkface.png)