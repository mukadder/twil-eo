# Using Twilio, AWS, and Electric Objects to create an MMS-powered family photo frame: setting up Twilio and AWS

The following are instructions for building the MMS-powered family photo frame described in [this post](https://jed.github.io/twil-eo/intro.html).

All in all this setup should take you about ten minutes. Setup is divided into three parts:

1. [create a JSON file](#login) with your Electric Objects login information,
2. set up an [S3 bucket](#s3), [IAM role](#iam), [Lambda function](#lambda), and [API Gateway endpoint](#apigateway) on AWS,
3. set up a [Twilio phone number](#twilio), and [point it at your API Gateway endpoint](#webhook).

<a name="login"></a>
### Step 1: Create a JSON file with your Electric Objects login information

Open your favorite text editor, and paste in the following JSON, replacing `where@jed.is` with the email address of your Electric Objects account and `timtam` with the password of your Electric Objects account.
```
{"email": "where@jed.is", "password": "timtam"}
```

Save this file as `eo-config.json`, anywhere on your computer where you can find it again.

<a name="s3"></a>
### Step 2a: Create an AWS S3 bucket.

This is where you'll store your EO1 account settings, the images coming in from Twilio, and the composited collage to be sent to the Electric Objects display.

1. From the top nav of the [AWS Console](https://console.aws.amazon.com), choose **Services**, then **All AWS Services**, then **S3**.
2. Click the **Create bucket** button.
3. For **Bucket Name**, specify the name of your project. Here, mine is `jed-family-frame`.
4. For **Region**, choose **US Standard**.
5. Click the **Create** button.
6. Click the **Upload** button.
7. Click the **Add Files** button.
8. Choose the `eo-config.json` file created above.
9. Click the **Start Upload** button.
10. Your bucket is ready.

<a name="iam"></a>
### Step 2b: Create an AWS IAM role.

This gives your Lambda function the permissions it needs to read from and write to the S3 bucket.

1. From the top nav of the [AWS Console](https://console.aws.amazon.com), choose **Services**, then **All AWS Services**, then **IAM**.
2. In the left sidebar, click the **Roles** button.
3. Click the **Create New Role** button.
4. For **Role Name**, specify the same name of your project as you did for the S3 bucket.
5. Click the **Select** button for **AWS Lambda**, under **AWS Service Roles**.
6. Select the checkboxes next to **AmazonS3FullAccess** and **CloudWatchLogsFullAccess**.
7. Click the **Next Step** button.
8. Click the **Create Role** button.
9. Your role is ready.

<a name="lambda"></a>
### Step 2c: Create an AWS Lambda function.

1. From the top nav of the [AWS Console](https://console.aws.amazon.com), choose **Services**, then **All AWS Services**, then **Lambda**.
2. Click the **Skip** button.
3. Click the **Next** button.
4. For **Name**, specify the same name of your project as you did for the S3 bucket.
5. For **Runtime**, choose **Node.js 4.3**.
6. For **Code entry type**, choose **Upload a .ZIP file**.
7. Click the **Upload** button and choose the `lambda.zip` file in your project directory.
8. For **Handler**, specify `index.handler`.
9. For **Role**, choose **Choose an existing role**.
10. For **Existing role**, choose the name of the role you created.
11. For **Memory (MB)**, specify `1024`.
12. For **Timeout**, specify `1` min.
13. For **VPC**, choose **No VPC**.
14. Click the **Next** button.
15. Click the **Create function** button.
16. Your lambda is ready.

<a name="apigateway"></a>
### Step 2d: Create an API Gateway endpoint

1. From the top nav of the [AWS Console](https://console.aws.amazon.com), choose **Services**, then **All AWS Services**, then **API Gateway**.
2. Click the **Get Started** button.
3. Click the **OK** button.
4. Select the **New API** radio button.
5. For **API name**, specify the same name of your project as you did for the S3 bucket.
6. Click the **Create API** button.
7. Click the **Actions...** button, and then choose **Create Method**.
8. From the select box, choose **POST** and then click the check button.
9. For **Integration type**, choose the **Lambda Function** radio button.
 10. For **Lambda Region**, choose the region for your function, which should be **us-east-1**.
 11. For **Lambda Function**, specify the first character name of your function (the same name as your project), and then choose the matching function name.
12. Click the **Save** button.
13. Click **OK**.
14. Click **Integration Request**.
  1. Under **Body Mapping Templates**, click the **Add mapping template** button,
  2. For **Content-Type**, specify `application/x-www-form-urlencoded`.
  3. Click the check button.
  4. Click the **Yes, secure this integration** button.
  4. For **application/x-www-form-urlencoded**, specify `$input.json('$')`.
  5. Click the **Save** button.
  6. Click **← Method Execution** button to go back.
15. Click **Integration Response**.
  1. Under **Body Mapping Templates**, click the **-** button next to **application/json**.
  2. Click the **Delete** button.
  3. click the **Add mapping template** button.
  4. For **Content-Type**, specify `application/xml`.
  5. Click the check button.
  6. For **application/xml**, specify `<?xml version="1.0" encoding="UTF-8"?><Response></Response>`.
  7. Click the **Save** button.
  8. Click **← Method Execution** button to go back.
16. Click **Method Response**.
  1. Under **Response Models for 200**, click the **Add response model** button,
  2. For **Content-Type**, specify `application/xml`.
  3. For **Model**, choose `Empty`.
  4. Click the check button.
  5. Click the **X** button next to **application/json** to delete that content type.
17. Click the **Actions...** button and choose **Deploy API**.
18. For **Deployment stage**, choose **[New Stage]**.
19. For **Stage name**, specify `prod`.
20. Click the **Deploy** button.
21. Take note of the URL given at the top in **Invoke URL**. This is the URL you'll use for your Twilio webhook.
22. Your endpoint is ready.

<a name="twilio"></a>
### Step 3a: Buy a Twilio number

1. Open the [Twilio Console](https://www.twilio.com/console/phone-numbers/incoming) for phone numbers.
2. Click the **+** button.
3. For **COUNTRY**, choose **United States**.
4. For **CAPABILITIES**, select **MMS**.
5. Click the **Search** button.
6. Choose the number you want, and click the **Buy** button.
7. Click the **Buy This Number** button.
8. Click the **Setup number** button.

<a name="webhook"></a>
### Step 3b: Point your Twilio number at API Gateway

1. Under **Messaging**, for **A MESSAGE COMES IN**,
  1. Choose **Webhook**.
  2. Specify the invoke URL from your API Gateway endpoint. It should look like `https://**********.execute-api.us-east-1.amazonaws.com/prod`.
  3. Choose **HTTP POST**
2. Click the **Save** button.
3. Your phone number is ready.