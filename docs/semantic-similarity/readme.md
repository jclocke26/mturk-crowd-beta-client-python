# semantic-similarity API

* [Introduction](#introduction)
* [Create a Task](#create-a-task)
* [Get a Result](#get-a-result)
* [Delete Task when Task is complete](#delete-task-when-task-is-complete)
* [Get started with semantic-similarity](#get-started-with-semantic-similarity)

## Introduction
The `semantic-similarity` API enables you to compare two bodies of text and determine how close they are in meaning using Amazon Mechanical Turk (MTurk). This is useful in use cases such as evaluating how well machine-generated text compare to human-generated text.

The API takes as input two strings of text, max length 400 characters each, to be analyzed and returns a score between 0 and 1 indicating how similar the texts are to each other with 1 being the most similar. When you call this API, it uses your MTurk Requester account and AWS account to automatically create HITs for Workers on MTurk to complete, and automatically processes answers from Workers and returns the result.

### Example Input:

```Python
{
  "input": {
    "text1": "The sky is blue.", 
    "text2": "The sky was the color of blue."
   }
}
```

### Example Result:

```Python
{
  "result": {
    "similarityScore": 0.75
  }
}
```

Here’s how the input might be presented in a HIT to Workers on MTurk.

![Worker Preview of HIT](https://s3-us-west-2.amazonaws.com/mturk-sample-tasks/semantic-similarity-HIT-example.jpeg "Worker Preview of HIT")

The API implements an adjudication strategy based on Worker agreement and returns results only when it reaches confidence.

---
All of the single use APIs follow the same REST pattern as described in our [REST API reference]. They require a function name indicating which API you want to call, and a unique Task name used to represent your input and the corresponding result. 

`semantic-similarity-test` is a version of the `semantic-similarity` API meant for testing your integration. It has the same interface as the production version, but Tasks always complete quickly with mock results.  HITs are not created on MTurk, so you don’t need to pay Workers for rewards.

> Note: This will return a random mock result when you call `get_task` with `semantic-similarity-test`.

The rest of this article provides instructions on using the `semantic-similarity` API using the MTurk Python Client.
[Download Sample Python code] for `semantic-similarity-test`.


## Create a Task
This Python sample calls the `semantic-similarity` API using [Boto3] and the [MTurk Python client]. 

Tasks are created by calling `put_task`. For more details for how to use `put_task`, check out [Instructions for creating your first Task for a MTurk single use API].

The body of a `put_task` request looks like this:

```
{
  "input": {
    "text1": <first text you want to compare>
    "text2": <second text you want to compare>
  }
}
```

##### Input structure

| Name | Description | Type | Required | 
| ---- | ----------- | ---- | :------: |
|"text1" | first text you want to compare | String (between 1-400 characters) | Yes|
|"text2" | second text you want to compare | String (between 1-400 characters) | Yes|

#### Example request

```
text1 = 'The sky is blue.'
text2 = 'The sky was the color of blue.'

put_result = crowd_client.put_task('semantic-similarity', 'my-task-name',                         
                             {'text1': text1, 'text2': text2})

```

#### Example response

```
{
  "taskName": "my-task-name",
  "input": {
    "text1": "The sky is blue.", 
    "text2": "The sky was the color of blue."
  },
  "problemDetails": null,
  "state": "processing",
  "result": null
}
```



## Get a Result
This Python sample calls the `semantic-similarity` API using [Boto3] and the [MTurk Python client]. 

After creating a Task, users can call the `get_task` to poll its current state. 

> Remember it can take some time for a Worker to complete a Task, so you might have to wait before getting a result. 

For more details for how to use `get_task`, check out [Instructions for creating your first Task for a MTurk single use API].

#### Example request

```
client.get_task('semantic-similarity', 'my-task-name')
```

#### Response structure:

| Name | Description | Type | 
| ---- | ----------- | ---- |
|"taskName" | user-provided Task name  | String|
|"input" | input provided by user when Task was created  | JSON Object|
|"problemDetails" | if the "state" is "failed" - details about a failed Task | JSON Object or null|
|"state" | current Task state - one of "processing", "completed" or "failed" | String|
|"result" | if the "state" is "completed" - the results of the completed Task  | JSON Object or null|



#### Result structure:

| Name | Value | Type | 
| ---- | ----------- | ---- |
|"similarityScore" | a score between 0 and 1 indicating similarity of inputs with 1 being the most similar  | Integer |

#### Example response

Example response for a successful Task:

```
{
  "taskName": "my-task-name",
  "input": {
    "text1": "The sky is blue.", 
    "text2": "The sky was the color of blue."
  },
  "problemDetails": null,
  "state": "completed",
  "result": {
    "similarityScore": 0.75
  }
}
```

Example response for a failed Task:

```
{
  "taskName": "my-task-name",
  "input": {
    "text1": "The sky is blue.", 
    "text2": "The sky was the color of blue."
  },
  "problemDetails": {
    "code": "Expired",
    "message": "Not enough Workers provided answers for your task within the allotted time."
  },
  "state": "failed",
  "result": null
}
```
## Delete Task when Task is complete
This Python sample calls the `semantic-similarity` API using [Boto3] and the [MTurk Python client]. 

A user can optionally delete a finished Task (one whose state is either “completed” or “failed”). After you delete a Task, you can reuse the Task name for a future Task.

>Note: Deleting a Task that is still being processed (i.e., whose state is “processing”) is not allowed.

For more details for how to use `delete_task`, check out [Instructions for creating your first Task for a MTurk single use API].

#### Example Input

```
client.delete_task('semantic-similarity', 'my-task-name')
```

#### Example Output

A successful request for the `delete_task` operation returns with no errors and an empty body.

---

## Get started with semantic-similarity

1. [Set up your Amazon Mechanical Turk (MTurk) Requester account and AWS account]
1. [Set up permissions to call an MTurk single use API]
1. [Instructions for creating your first Task for a MTurk single use API]

MTurk has released several APIs for common use cases, [click here to see a list of all the available APIs].

If you have any questions or feedback, such as methods you wish our client supported, [please contact our product team], or submit a pull request [on GitHub] adding additional functionality to our client.

[REST API reference]: ../REST.md
[Set up your Amazon Mechanical Turk (MTurk) Requester account and AWS account]: ../step_0_setup_accounts.md
[Set up permissions to call an MTurk single use API]: ../step_1_setup_aws_user.md
[Install Python client and create your first Tasks with MTurk API]: ../step_2_first_task.md
[Instructions for creating your first Task for a MTurk single use API]: ../step_2_first_task.md
[click here to see a list of all the available APIs]: ../../readme.md#what-apis-are-available

[Boto3]: https://boto3.readthedocs.io/en/latest/guide/quickstart.html#installation
[MTurk Python client]: https://github.com/awslabs/mturk-crowd-beta-client-python


[Download Sample Python code]: https://gist.github.com/AmazonMTurk/6ec4e33a3b0b91821b1fba96f23de598#file-semantic-similarity-test-py

[please contact our product team]: mailto:mturk-requester-preview@amazon.com
[on GitHub]: https://github.com/awslabs/mturk-crowd-beta-client-python

