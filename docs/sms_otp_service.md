# SMS OTP Service

This module provides a fully decoupled OSGI service to generate and validate SMS OTPs, outside
an authentication flow.

## Deployment instructions
1. Install Java 11 (or Java 17).
2. Build the repository using `mvn clean install`.
3. Copy the `org.wso2.carbon.extension.identity.smsotp.common-<VERSION>.jar` to the 
`<IS_HOME>/repository/components/dropins` directory.
4. Add the below configurations to the `<IS_HOME>/repository/conf/deployment.toml` file.
```properties
[[event_handler]]
name= "smsOtp"
properties.enabled=true
properties.tokenLength=6
properties.triggerNotification=true
properties.alphanumericToken=true
# OTP validation failure reason will be sent in the response.
properties.showValidationFailureReason=false
properties.tokenValidityPeriod=120
# Same valid OTP will be resent, if issued within the interval.
# Set '0' to always send a new OTP.
# Should be less than the 'tokenValidityPeriod' value.
properties.tokenRenewalInterval=60
# Throttle OTP generation requests from the same user Id.
# Set '0' for no throttling.
properties.resendThrottleInterval=30
# Set the maximum validation attempts allowed until the generated sms-otp expires.
properties.maxValidationAttemptsAllowed=5
```
4. If notifications are managed by the Identity Server, configure the **SMS template** by appending below at the end of
   the `<IS_HOME>/repository/conf/sms/sms-templates-admin-config.xml` file.
```xml
    <configuration type="sendOTP" display="sendOTP" locale="en_US">
        <body>Your One Time Password is : {{confirmation-code}}</body>
    </configuration>
```
5. If notifications are managed by the Identity Server, configure the **event publisher** by creating 
   `SMSPublisher.xml` file in the `<IS_HOME>/deployment/server/eventpublishers/` directory.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<eventPublisher name="SMSPublisher" processing="enable"
                statistics="disable" trace="disable" xmlns="http://wso2.org/carbon/eventpublisher">
    <from streamName="id_gov_sms_notify_stream" version="1.0.0"/>
    <mapping customMapping="enable" type="json">
        <inline>{"text": {{body}}, "to": [{{mobile}}]}</inline>
    </mapping>
    <to eventAdapterType="http">
        <property name="http.client.method">httpPost</property>
        <property name="http.url">https://webhook.site/678cf852-39a3-416a-8ff9-4331905d1b95</property>
    </to>
</eventPublisher>
```
6. Restart the server.

**NOTE::** To include a **unique identification** in the **SMS template**, use the `correlation-id` variable in the 
following syntax,

`{{correlation-id}}`

Following is a sample which includes the `correlation-id` in the SMS Template located available 
`<IS_HOME>/repository/conf/sms/sms-templates-admin-config.xml` file,
```xml
    <configuration type="sendOTP" display="sendOTP" locale="en_US">
        <body>Your One Time Password is : {{confirmation-code}}. 
        Reference Id: {{correlation-id}}</body>
    </configuration>
```