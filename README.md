# go-cloud-lambda-info
A simple AWS serverless lambda function in Golang


### How to deploy (long version)

##### first let's configure your connection to aws (login)
	aws configure

##### lets' build your go code
	GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build lambdaFunction.go 

##### zip the build
	zip lambdaFunction.zip lambdaFunction

##### create the IAM role for your lambda function (if needed)
 	aws iam create-role --role-name lambda-basic-execution --assume-role-policy-document file://lambda-trust-policy.json
	aws iam attach-role-policy --role-name lambda-basic-execution --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
##### check that role was created
	aws iam get-role --role-name lambda-basic-execution

##### extract the arn with jq
	export MyARN=$(aws iam get-role --role-name lambda-basic-execution |jq '.Role.Arn')

##### create your lambda function
	aws lambda create-function --function-name go-cloud-lambda-info_lambdaFunction --zip-file fileb://lambdaFunction.zip --handler  lambdaFunction --runtime go1.x  --role  $MyARN

##### call your lambda function from shell
using a file for the payload : 

	aws lambda invoke --function-name go-cloud-lambda-info_lambdaFunction --invocation-type "RequestResponse" --cli-binary-format raw-in-base64-out --payload file://payload.json response.txt
	{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
	}

or  just typing the payload in the command line:

	aws lambda invoke --function-name go-cloud-lambda-info_lambdaFunction --invocation-type "RequestResponse" --cli-binary-format raw-in-base64-out --payload '{"name": "lambda is working !"}' response.txt
will output :

	{
    "StatusCode": 200,
    "ExecutedVersion": "$LATEST"
	}


now the file response.txt contains the result of your call :

	cat response.txt
    Hello lambda is working !!
	



### how to update your lambda function

	aws lambda update-function-code --function-name go-cloud-lambda-info_lambdaFunction --zip-file fileb://lambdaFunction.zip



### more information

+  [Creating a role for a service (AWS CLI)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html#roles-creatingrole-service-console)
+  [how to deploy a Golang package in AWS Lambda](https://medium.com/@daniel.woods/deploying-a-golang-package-to-aws-lambda-in-5-minutes-cd11685f576)
+  [Aws documentation for building Lambda functions in Go](https://docs.aws.amazon.com/lambda/latest/dg/lambda-golang.html)

+ [Deploy Go Lambda functions with .zip file archives](https://docs.aws.amazon.com/lambda/latest/dg/golang-package.html)