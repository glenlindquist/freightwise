# FreightWise Weather App

[View app](https://glenlindquist.github.io/freightwise/)

Simple app to grab weather data for Brentwood, TN from OpenWeather API.

Displays current temp, feels-like temp, daily high and low, along with icon to indication weather conditions.

## External Libraries

For quick styling/prototyping I pulled in Bootstrap. For quick alerts I pulled in Alertify.js library.

## SMS Feature

To enable SMS, I set up an AWS API gateway endpoint that would trigger an AWS Lambda function that I wrote in Ruby. This function used the Ruby SDK for AWS Pinpoint, which allows you to send one-off SMS messages.

### AWS Lambda function

Here's the Lambda function in question:

    require 'json'
    require 'aws-sdk-pinpoint'
    
    def lambda_handler(event:, context:)
        begin
            phone = event["queryStringParameters"]["phone"].gsub("-","")
            city = event["queryStringParameters"]["city"]
            state = event["queryStringParameters"]["state"]
            current_temp = event["queryStringParameters"]["currentTemp"].to_f
            high_temp = event["queryStringParameters"]["highTemp"].to_f 
            low_temp = event["queryStringParameters"]["lowTemp"].to_f
            feels_like_temp = event["queryStringParameters"]["feelsLike"].to_f
            condition = event["queryStringParameters"]["condition"]
            
            phone = "+1" + phone
            return { statusCode: 400, body: JSON.generate(message: "Phone number incorrect length") } unless phone.length == 12
            
            message = "Weather for #{city}, #{state}\n" + 
                "#{condition}\n" +
                "Current temp: #{current_temp.round}°F\n" +
                "Feels like: #{feels_like_temp.round}°F\n" +
                "Low: #{low_temp.round}°F\n" +
                "High: #{high_temp.round}°F\n\n" +
                "Text STOP to stop receiving these messages"
            
            pinpoint = Aws::Pinpoint::Client.new()
            sent_message = pinpoint.send_messages(
                application_id: [REPLACE W/ PINPOINT APPLICATION ID],
                message_request: {
                    addresses: {
                        phone => {channel_type: 'SMS'}
                    },
                    message_configuration: {
                        sms_message: {
                            body: message,
                            message_type: 'PROMOTIONAL'
                        }
                    }
                }
            )
            
            if sent_message[:message_response][:result][phone][:delivery_status] == "SUCCESSFUL"
                { statusCode: 200, body: JSON.generate(sent_message) }
            else
                { statusCode: 400, body: JSON.generate(message: "Issue sending SMS.") }
            end
        rescue
            { statusCode: 400, body: JSON.generate(message: "Issue sending SMS.") }
        end
    end