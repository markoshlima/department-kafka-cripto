# **Introduction**

The project is about testing an event drive archicture with encripted data, where all consumers could read from the same source, but not all could see the data. The ideia is working with encription keys that is managed in a federated way, is this model, we can easly by a governance control, delegate which applications could really read that information.

# **Event Drive Architecture Patterns**

Using the Apache Kafka as example, in a Event Drive Architecture simple pattern, we have the following model:

![alt text](https://raw.githubusercontent.com/markoshlima/department-kafka-cripto/main/docs/EDA-options-1.png)

The image above, is the simplest way to expose data by event drive architecture, where the owner does not know the consumers creating an uncoupled system. But what happen when there is totally personal and sensitive data that could not be shared in all applications?

We can use patterns to solve that problem:

The mailbox patter:

![alt text](https://raw.githubusercontent.com/markoshlima/department-kafka-cripto/main/docs/EDA-options-2.png)

This pattern create an especific topic for each one of consumers in their own domain context, it may even be a different messaging tecnology for each one, that is, like an email box, the appliation producer send a messaage for that interested one. Here we create not a totally desaclouped system as the way the owner producer data has to know every consumer.

The other pattern bellow is very closelly from mailbox, but here, the topics are controlled by te producer data, all the problems from mainbox is basically the same, but here, the owner data producer could control everything like: tecnology, CI/CD and the schema.

![alt text](https://raw.githubusercontent.com/markoshlima/department-kafka-cripto/main/docs/EDA-options-3.png)

All of these patterns solve the problem, but increase complexity for everyone. This project sugest using the same event drive archicteture, but instead of create lot of topics working with only one with distribuited encripty keys that could be controled by a governance and federated way.

# **Solution Architecture**

It was created three domain context:
 - Owner Data: The producer sensitive and personal data to AWS MKS (Managed Kafka) for departments inside company
 - Sales Domain: As the name implies, is the sales department that are interested in the events
 - Sales Domain: The finance consumer department

The services was deployed in AWS Elastic Container Service (ECS), and the server for encripty and decripty as the keys was used the AWS KMS.
To generate data, was created a Lambda function sending the messages to a SQS queue, where the owner data will consume and encripty the messages using the correctly key.
Every acces to the keys is controled by IAM roles/actions.
There is a partition strategy, to maintain order and each consumer get from the same topic.

![alt text](https://raw.githubusercontent.com/markoshlima/department-kafka-cripto/main/docs/Solution%20Archicture.png)


# Result

As the result, bellow it have two consumers from MSK topics reading from the same topic, and decripting the message using it own keys.

Finance Consumer

	---------------- ORIGINAL MESSAGE --------------
	{"department": "finance", "data": "AgV4KrWhRx1gYdlU+1T3qM551W1+3uZiW3tKHJPKyZdOUbwAXwABABVhd3MtY3J5cHRvLXB1YmxpYy1rZXkAREFyV2t5ditvcTNnSmdUUU04V1RNYi90NnBqNmxObEw3UVVRTEFzdXJ0eDBpcXE2NkpSTGNVTzhSSFFCUks3MnNadz09AAEAB2F3cy1rbXMAS2Fybjphd3M6a21zOnVzLWVhc3QtMToyOTc3MzgyNTg2MzM6a2V5LzAzZmJkZTA0LTVmY2ItNGQzNS04ZjMzLTNlYTFiMDQzNzVhZAC4AQIBAHjyt160wKfdtwKkN2WkpgNTvAv7GSeUuKcNOfSZxAa36QG2AyKyqNBEjWDieC3XVpjIAAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMWNwMZZ75P8d0GCeqAgEQgDu077wpYhcEZdAZ0fEgXp2fTVU81uUd3Yc8ezOEY0Md+X2G9HbioqsIeUJUl5teH2pfm4s3UOrfPPutaAIAABAAUs/xZMlvE477XqY4HWOsepCy6BAspAoNlPQ07NxzC1eHi+o+RD4Fca5GCqOp7epp/////wAAAAEAAAAAAAAAAAAAAAEAAAB6lsZxayYvujvaWGjMLaVkeIlJjgvmQebJV9vqvAy2CYU++I8xSKuNZltIC6dAw4YX8JQoc0WD3ZPDqmjch09Fm8Sx21cR4An3UZdWEn6YRiBubaC+CoXYv0dqBc5c4u1sBm1/igxlIBwdkW0zS3q+l0x9Y4GuLeo+HwllWQ/Q+rKT+6RWLzXsdPrTAGcwZQIxALwm3ZBHHZXYxeeoBKdmUcVPURj5m3wrq89gjzX3NqYl2UMlCNsU0Frsil1pDvwYpQIwexiiibqFYa7Aj7uOiOjtxOID2m92CpLH8sYFjlIsvkoJNRlj7YTQFQJBt+o49u/q"
	---------------- DECRIPTED MESSAGE --------------
	{"department": "finance", "data": "Lot of consumers, but this is a sensitive and personal information that only finance department can read (could be a json)"
	---------------- PARTITION --------------
	2

Sales Consumer

	---------------- ORIGINAL MESSAGE --------------
	{"department": "sales", "data": "AgV42eI09g91g9X0IDxLjJJQclVNMmPwlePN6nDvO2ygy7oAXwABABVhd3MtY3J5cHRvLXB1YmxpYy1rZXkAREF4di9oOFFzaENRZU9mZWNDTUpVdmp1N0Y2ZWZzOUE4WmIzbnNqVlQ2Q3RvYVpTc25EMVRCRmFsU2V4UjN6a1c3dz09AAEAB2F3cy1rbXMAS2Fybjphd3M6a21zOnVzLWVhc3QtMToyOTc3MzgyNTg2MzM6a2V5LzJlYzAwNzhmLTY0ZTEtNGViMi05ZjZkLTI2YzlkNTFhMDg0MgC4AQIBAHiFfnoEmQlznJ0UuzoebpJQO3RGqDOp3OR/g+y9r4XyZwHt0MHE2aYUJccFpMeT0TNTAAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMYkTQb06gnK3iWXFDAgEQgDuB+Kz0IThKCarlyKD+MNvbxYBYXZD50Q/CNH2un0Jf5S0U6TmkcWjcBP6n4/tnS9I2j3S7DspzcVJ8GAIAABAAgWeznSrpJ/A3ta0iC51wGmWKZe1hELPcBFeZas0rxonp1bkaxiG+Px7CZl9eFSjX/////wAAAAEAAAAAAAAAAAAAAAEAAAB2Iib5YC08XWnOD8chFU11yB3+hrGF1jeyWQ7rA9CJojP6M7snByqTFH1x0Cs9Byscs3BluK1Agz03T1qK34kQn4sxpnWKubMoiw/oyNIEyVmS367iss5LRaG44js9MrjhKN1l33HlcuIFh1NrJPDAaWEJ6HkUmkmW5W3eKQGnS+eiq+fE1pYAZzBlAjEA2H5Jqk0/xTurAC8cnTE5VQXDeHVkuTTutmFVAoxNWtpLT0iiAVyxivLhXru7vdo5AjAVJwlSAsszOIlHvMpuBeN0qrZQ9OvrffCD7z+U9Z7oCGCVLkeJfdhI4VK1xHt02I8="
	---------------- DECRIPTED MESSAGE --------------
	{"department": "sales", "data": "Lot of consumers, sensitive / personal information directly for sales department, just them can read (could be a json)"
	---------------- PARTITION --------------
	1

If any service try to read an message that is not for it, it will get the following error:

    Unable to decrypt any data keys
    The ciphertext refers to a customer master key that does not exist, does not exist in this region, or you are not allowed to access. (Service: AWSKMS; Status Code: 400; Error Code: AccessDeniedException)

# Limitations

AWS KMS only encripty 4KB of data, if the payload is greatter than that, probably it will has to encripty field by field.
Event using Apache Kafka or anyelse tecnology, it is not possible to have a strong contract between the services.
The performance could be affected, in a heavy workload this soluction could be a problem.

# Addition Information & Setup

To setup the environment:
  - Inside folder /IaaC there is the template used to create all infraestructure, it is possible to use with cloudformation.
  - Publish all container images inside ECR, and change all templates TaskDefinition.
  - Create the AWS MSK keys