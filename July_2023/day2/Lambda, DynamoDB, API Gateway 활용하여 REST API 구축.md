# Lambda, DynamoDB, API Gateway 활용하여 REST API 구축

# DynamoDB 구축

---

![](<./image/스크린샷 2023-08-01 오전 12.11.34.png>)

# IAM setup

---

![](<./image/스크린샷 2023-08-01 오전 12.12.27.png>)

![](<./image/스크린샷 2023-08-01 오전 12.12.44.png>)

# Lambda 구성

![](<./image/스크린샷 2023-08-01 오전 12.14.38.png>)

![](<./image/스크린샷 2023-08-01 오전 12.17.01.png>)

# Lambda Function Code

```go
package main

import (
	"context"
	"fmt"
	"github.com/aws/aws-lambda-go/lambda"
	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/dynamodb"
	"github.com/aws/aws-sdk-go/service/dynamodb/dynamodbattribute"
	"log"
	"regexp"
	"time"
)

type CreateUserRequest struct {
	Email string `json:"email"`
}

type User struct {
	Email string `json:"email"`
	Date  string `json:"date"`
}

func main() {
	lambda.Start(HandleRequest)
}

func HandleRequest(ctx context.Context, request CreateUserRequest) (string, error) {
	if !isEmailValid(request.Email) {
		return fmt.Sprintln("Failed to add user '" + request.Email + " as it has invalid email"), nil
	}

	sess := session.Must(session.NewSessionWithOptions(session.Options{
		SharedConfigState: session.SharedConfigEnable,
	}))

	svc := dynamodb.New(sess)

	user := User{
		Email: request.Email,
		Date:  time.DateTime,
	}

	av, err := dynamodbattribute.MarshalMap(user)
	if err != nil {
		log.Fatalf("Got error marshalling new movie item: %s", err)
	}

	tableName := "rest-dyanmodb"

	input := &dynamodb.PutItemInput{
		Item:      av,
		TableName: aws.String(tableName),
	}

	_, err = svc.PutItem(input)
	if err != nil {
		log.Fatalf("Got error calling PutItem: %s", err)
	}

	return fmt.Sprintln("Successfully added '" + user.Email + " to table " + tableName), nil
}

func isEmailValid(e string) bool {
	emailRegex := regexp.MustCompile("^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$")
	return emailRegex.MatchString(e)
}
```

이후 `GOOS=linux GOARCH=amd64 CGO_ENABLED=0 go build -o main main.go`, `zip main.zip main`한 뒤, 해당 파일을 업로드한다. 이때 함수를 main으로 사용하도록 한다.

# 테스트

## 올바르지 않은 값

![](<./image/스크린샷 2023-08-01 오전 12.30.56.png>)

![](<./image/스크린샷 2023-08-01 오전 12.30.43.png>)

## 올바른 값

![](<./image/스크린샷 2023-08-01 오전 12.36.11.png>)

![](<./image/스크린샷 2023-08-01 오전 12.32.40.png>)

요청이 처리되었다.

![](<./image/스크린샷 2023-08-01 오전 12.33.21.png>)

dynamoDB에도 잘 들어간 것을 확인할 수 있다.

# API Gateway 배포

apigateway에 배포하고 요청을 넣었으나, Internal Server error가 표시되어 cloudwatch 로그들을 확인해보니 lambda 함수에서 apigateway Response로 반환하지 않아서 apigateway가 internal server error를 띄웠다. 때문에 handler 코드를 고치고 다시 업로드하였다.

## 바뀐 코드

```go
package main

import (
	"fmt"
	"github.com/aws/aws-lambda-go/events"
	"github.com/aws/aws-lambda-go/lambda"
	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/dynamodb"
	"github.com/aws/aws-sdk-go/service/dynamodb/dynamodbattribute"
	"regexp"
	"time"
)

type User struct {
	Email string `json:"email"`
	Date  string `json:"date"`
}

func main() {
	lambda.Start(HandleRequest)
}

func HandleRequest(request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
	ApiResponse := events.APIGatewayProxyResponse{
		StatusCode: 200,
		Body:       "",
	}

	email := request.QueryStringParameters["email"]
	if !isEmailValid(email) {
		ApiResponse = events.APIGatewayProxyResponse{
			Body:       "invalid email",
			StatusCode: 400,
		}
		return ApiResponse, fmt.Errorf(ApiResponse.Body)
	}

	sess := session.Must(session.NewSessionWithOptions(session.Options{
		SharedConfigState: session.SharedConfigEnable,
	}))

	svc := dynamodb.New(sess)

	user := User{
		Email: email,
		Date:  time.Now().String(),
	}

	av, err := dynamodbattribute.MarshalMap(user)
	if err != nil {
		ApiResponse = events.APIGatewayProxyResponse{
			Body:       fmt.Sprintln("Got error marshalling : %s", err),
			StatusCode: 400,
		}
		return ApiResponse, fmt.Errorf(ApiResponse.Body)
	}

	tableName := "rest-dyanmodb"

	input := &dynamodb.PutItemInput{
		Item:      av,
		TableName: aws.String(tableName),
	}

	_, err = svc.PutItem(input)
	if err != nil {
		ApiResponse = events.APIGatewayProxyResponse{
			Body:       fmt.Sprintln("Got error calling PutItem: %s", err),
			StatusCode: 500,
		}
		return ApiResponse, fmt.Errorf(ApiResponse.Body)
	}

	ApiResponse = events.APIGatewayProxyResponse{
		Body:       "Successfully added '" + user.Email + " to table " + tableName,
		StatusCode: 201,
	}

	return ApiResponse, nil
}

func isEmailValid(e string) bool {
	emailRegex := regexp.MustCompile("^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$")
	return emailRegex.MatchString(e)
}
```

# 결과

![](<./image/스크린샷 2023-08-01 오전 1.42.26.png>)

![](<./image/스크린샷 2023-08-01 오전 1.44.52.png>)

postman과 dynamoDB에서 성공적으로 응답을 확인할 수 있었다.